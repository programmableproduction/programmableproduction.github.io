---
layout: post
title: CoreClr problem with ICU dependency on Linux
categories: Archlinux, CoreCRL
tags: Archlinux, CoreCRL, UCI
---
This post follows the [previous article]({{ site.url }}/2016/04/08/Building-CoreClr-On-ArchLinux/) about building .Net CoreClr on ArchLinux. The instruction that are in that post  show you how to install the dependency for ArchLinux, however they are still a problem about building mscorelib.dll on Archlinux. I will summarise the problem, how I investigated the root cause and some solution that I'm looking at to merge back to the CoreClr github repository later.

# The problem
The build process of the .Net CoreClr is mainly building some C++ native library (like JIT, PAL) which are doing the translation of IL code to machine code and all the platform abstraction layer. The last build step is building mscorelib.dll which is the base library of .Net. However this is failing with the below error 

```
Restoring BuildTools...
Raising file open limit - otherwise Mac build may fail
ulimit -n 2048
.NET CLI will be downloaded from https://dotnetcli.blob.core.windows.net/dotnet/beta/Binaries/1.0.0-beta-002173/dotnet-dev-ubuntu-x64.1.0.0-beta-002173.tar.gz
Locating /home/david/git/coreclr/Tools/1.0.25-prerelease-00308-04/project.json to see if we already downloaded .NET CLI tools...
/home/david/git/coreclr/Tools/1.0.25-prerelease-00308-04/project.json not found. Proceeding to download .NET CLI tools. 
curl -sSL --create-dirs -o /home/david/git/coreclr/Tools/dotnetcli/dotnet.tar https://dotnetcli.blob.core.windows.net/dotnet/beta/Binaries/1.0.0-beta-002173/dotnet-dev-ubuntu-x64.1.0.0-beta-002173.tar.gz
Unsupported Linux distribution 'arch' detected. Downloading ubuntu-x64 tools.
Failed to initialize CoreCLR, HRESULT: 0x80131500
Failed to initialize CoreCLR, HRESULT: 0x80131500
chmod: cannot access '/home/david/git/coreclr/Tools/corerun': No such file or directory
mkdir: cannot create directory '/home/david/git/coreclr/packages/Microsoft.DotNet.BuildTools/1.0.25-prerelease-00308-04/lib/portableTargets': File exists
Failed to initialize CoreCLR, HRESULT: 0x80131500
cp: cannot stat '/home/david/git/coreclr/Tools/corerun': No such file or directory
mv: cannot stat '/home/david/git/coreclr/Tools/Microsoft.CSharp.targets': No such file or directory
chmod: cannot access '/home/david/git/coreclr/Tools/corerun': No such file or directory
Failed to restore BuildTools.

```

The interesting line here is ```Failed to initialize CoreCLR, HRESULT: 0x80131500```. It look like something in the init-tools.sh is throwing this error. By putting some trace in the file I found that it's the line which is calling "dotnet" which throw the error.  Looking online or on the github issue of the project I couldn't find anything useful. 

# CoreClr Build process 
As I started to look around to understand what was happening there and looking at the build script. ```build.sh``` is calling ```init-tools.sh```. The script 'init-tools.sh' is downloading from a remote server the package of [.Net cli](https://github.com/dotnet/cli) (.Net core Command LIne) which is a new tool that allow to create, build and run .Net code.
The dotnetcli is used to download the dependency that mscorelib has from Nuget. The stepm that call ```dotnet restore``` is failing with the above error. The most basic command you can run is ```dotnet --version``` which doesn't run anything and should just return the current version of the tool. Unfortunately this simple version if failing with the same error. 
Looking at the script you can see that it's downloading a specific version of dotnetcli depending on the Linux distribution you are on, and it's the distribution cannot be found it will use the package for the ubuntu 14.04

# Investigation the error
The experience told me that the problem we have is related to missing dependency. On windows I would use a tool like [procmon](https://technet.microsoft.com/en-us/sysinternals/processmonitor.aspx) which allow to monitor all the input/output of the process. Luckily Linux has an equivalent tool named 'strace'. I found this great [tutorial](http://www.danielhall.me/tag/strace/) from Daniel Hall that explain the basic about the information you can find in the file.
The ```dotnet``` executable is downloaded into the child folder 'Tools/dotnetcli'. We just have to go to the pass and start strace

```bash
cd Tools/dotnetcli
strace -o dotnet.log ./dotnet --version
```
The command above saved the log to the file 'dotnet.log' which will we have to investigate now ;-)
Looking inside of the file, the first think we have to look at is for "(No such file or directory)" pattern which inform us about potential missing dependency. In our case one of the recurrent error is the following. 

``` 
open("/usr/lib/libicuuc.so.52", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)

```

Which show us that we are missing the library "icu". But wait.... We installed that on the previous post, so how come. It's time to check what I have in the folder "*/usr/lib*". 

```shell
[david@dc-archlinux-1 coreclr]$ ls -ali /usr/lib/libicu*
162387 lrwxrwxrwx 1 root root       18 Apr  2 15:18 /usr/lib/libicudata.so -> libicudata.so.57.1
162370 lrwxrwxrwx 1 root root       18 Apr  2 15:18 /usr/lib/libicudata.so.57 -> libicudata.so.57.1
162391 -rwxr-xr-x 1 root root 25674392 Apr  2 15:18 /usr/lib/libicudata.so.57.1
162389 lrwxrwxrwx 1 root root       18 Apr  2 15:18 /usr/lib/libicui18n.so -> libicui18n.so.57.1
162368 lrwxrwxrwx 1 root root       18 Apr  2 15:18 /usr/lib/libicui18n.so.57 -> libicui18n.so.57.1
162375 -rwxr-xr-x 1 root root  2591776 Apr  2 15:18 /usr/lib/libicui18n.so.57.1
162378 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicuio.so -> libicuio.so.57.1
162374 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicuio.so.57 -> libicuio.so.57.1
162385 -rwxr-xr-x 1 root root    56008 Apr  2 15:18 /usr/lib/libicuio.so.57.1
162390 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicule.so -> libicule.so.57.1
162379 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicule.so.57 -> libicule.so.57.1
162371 -rwxr-xr-x 1 root root   354544 Apr  2 15:18 /usr/lib/libicule.so.57.1
162381 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libiculx.so -> libiculx.so.57.1
162376 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libiculx.so.57 -> libiculx.so.57.1
162377 -rwxr-xr-x 1 root root    51640 Apr  2 15:18 /usr/lib/libiculx.so.57.1
162372 lrwxrwxrwx 1 root root       18 Apr  2 15:18 /usr/lib/libicutest.so -> libicutest.so.57.1
162373 lrwxrwxrwx 1 root root       18 Apr  2 15:18 /usr/lib/libicutest.so.57 -> libicutest.so.57.1
162369 -rwxr-xr-x 1 root root    65760 Apr  2 15:18 /usr/lib/libicutest.so.57.1
162386 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicutu.so -> libicutu.so.57.1
162388 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicutu.so.57 -> libicutu.so.57.1
162380 -rwxr-xr-x 1 root root   201240 Apr  2 15:18 /usr/lib/libicutu.so.57.1
162384 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicuuc.so -> libicuuc.so.57.1
162382 lrwxrwxrwx 1 root root       16 Apr  2 15:18 /usr/lib/libicuuc.so.57 -> libicuuc.so.57.1
162383 -rwxr-xr-x 1 root root  1727160 Apr  2 15:18 /usr/lib/libicuuc.so.57.1

```

**Bingo**
Look like ArchLinux installed the latest version of the "icu" library which is 57 whereas Ubuntu 14.04 which is the server that has build the package only support the version 52 of icu. So now that we know what is what is the missing dependency it's time to look at what is our library that is calling it. Checking few line above in the log file I can find that "*System.Globalization.Native.so*" is looking for that dependency. Thinking more about it this relation make sense as icu stand for "International Component for Unicode" and would be required for the globalization component of CoreClr.

```
open("/home/david/git/coreclr/Tools/dotnetcli/shared/Microsoft.NETCore.App/1.0.0-rc2-23931/System.Globalization.Native.so", O_RDONLY|O_CLOEXEC) = 12
read(12, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\240F\0\0\0\0\0\0"..., 832) = 832
fstat(12, {st_mode=S_IFREG|0755, st_size=56944, ...}) = 0
mmap(NULL, 2151880, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 12, 0) = 0x7f57099f3000
mprotect(0x7f5709a00000, 2093056, PROT_NONE) = 0
mmap(0x7f5709bff000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 12, 0xc000) = 0x7f5709bff000
close(12)                               = 0
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 12
fstat(12, {st_mode=S_IFREG|0644, st_size=48486, ...}) = 0
mmap(NULL, 48486, PROT_READ, MAP_PRIVATE, 12, 0) = 0x7f5798020000
close(12)                               = 0
open("/usr/lib/libicuuc.so.52", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
munmap(0x7f5798020000, 48486)           = 0
munmap(0x7f57099f3000, 2151880)         = 0
open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 12
fstat(12, {st_mode=S_IFREG|0644, st_size=48486, ...}) = 0
mmap(NULL, 48486, PROT_READ, MAP_PRIVATE, 12, 0) = 0x7f5798020000
close(12)                               = 0
open("/usr/lib/System.Globalization.Native.so", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
munmap(0x7f5798020000, 48486)           = 0

```

Looking more into this problem, I found that linux has a command line that allow you to check for a given module the dependency and that they are present. The command line is [ldd](http://man7.org/linux/man-pages/man1/ldd.1.html) which allow to print the object dependencies. Running this command on the ```System.Globalization.Native.so``` give us the following result.

```
[david@dc-archlinux-1 coreclr]$ ldd Tools/dotnetcli/shared/Microsoft.NETCore.App/1.0.0-rc2-23931/System.Globalization.Native.so 
        linux-vdso.so.1 (0x00007ffe8057d000)
        libicuuc.so.52 => not found
        libicui18n.so.52 => not found
        libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0x00007f04c84b1000)
        libm.so.6 => /usr/lib/libm.so.6 (0x00007f04c81ac000)
        libgcc_s.so.1 => /usr/lib/libgcc_s.so.1 (0x00007f04c7f96000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007f04c7bf4000)
        /usr/lib64/ld-linux-x86-64.so.2 (0x0000557cb89c6000)
```

One more time we can see that the module libicu are missing for the version 52 in this case.

# Solutions
I found two solution that allow to resolve the problem of the ICU dependency failure

1. Downgrade ArchLinux ICU package installed to the version 52
2. Use the Native version of System.Globalization.Native.so build on the machine

## Downgrading ICU to the version 52
Pacman that ArchLinux package manager has an archive website that you can use to find historical packages. You can browse this [link](https://archive.archlinux.org/packages/) to look at the package you are looking at. In our case we are interested to find the [icu](https://archive.archlinux.org/packages/i/icu/). When you have the package you want you can run pacman with the option -U to install it. In our case we want to install ICU version 52 for x64 the command line will be

```shell
sudo pacman -U https://archive.archlinux.org/packages/i/icu/icu-52.1-1-x86_64.pkg.tar.xz
```
and running again ldd show us that time

```shell
[david@dc-archlinux-1 coreclr]$ ldd Tools/dotnetcli/shared/Microsoft.NETCore.App/1.0.0-rc2-23931/System.Globalization.Native.so 
        linux-vdso.so.1 (0x00007fffbbf9e000)
        libicuuc.so.52 => /usr/lib/libicuuc.so.52 (0x00007f0cf71ae000)
        libicui18n.so.52 => /usr/lib/libicui18n.so.52 (0x00007f0cf6da6000)
        libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0x00007f0cf6a23000)
        libm.so.6 => /usr/lib/libm.so.6 (0x00007f0cf671e000)
        libgcc_s.so.1 => /usr/lib/libgcc_s.so.1 (0x00007f0cf6508000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007f0cf6166000)
        libicudata.so.52 => /usr/lib/libicudata.so.52 (0x00007f0cf48fb000)
        libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007f0cf46de000)
        libdl.so.2 => /usr/lib/libdl.so.2 (0x00007f0cf44d9000)
        /usr/lib64/ld-linux-x86-64.so.2 (0x0000561b5e424000)

```
Look like downgrading made dotnet dependency problem going away. However this has some other implication because other software are depending on that library and are now starting to fail as well. Time too look at the option 2.

## Copying the Native lib build on the machine
Before doing that you need to be sure your system is update to date by running ```sudo pacman -Syu``` and building coreclr using ```./build.sh clean skipmscorlib```. When you are done with that tasks, you can look at copying the freshly build "System.Globalization.Native.so" to the folder where ```dotnet``` is picking it.
We can now see that we have a dependency with ICU version 57 present in the machine.

```shell
[david@dc-archlinux-1 coreclr]$ ldd bin/Product/Linux.x64.Debug/System.Globalization.Native.so 
        linux-vdso.so.1 (0x00007ffffe3d9000)
        libicuuc.so.57 => /usr/lib/libicuuc.so.57 (0x00007feb90e9a000)
        libicui18n.so.57 => /usr/lib/libicui18n.so.57 (0x00007feb90a20000)
        libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0x00007feb9069d000)
        libm.so.6 => /usr/lib/libm.so.6 (0x00007feb90398000)
        libgcc_s.so.1 => /usr/lib/libgcc_s.so.1 (0x00007feb90182000)
        libc.so.6 => /usr/lib/libc.so.6 (0x00007feb8fde0000)
        libicudata.so.57 => /usr/lib/libicudata.so.57 (0x00007feb8e364000)
        libpthread.so.0 => /usr/lib/libpthread.so.0 (0x00007feb8e147000)
        libdl.so.2 => /usr/lib/libdl.so.2 (0x00007feb8df42000)
        /usr/lib64/ld-linux-x86-64.so.2 (0x00005569e3704000)
```

either of the above solution help to produce the expected result... Just getting the version of the dotnet exec

```shell
[david@dc-archlinux-1 coreclr]$ Tools/dotnetcli/dotnet --version                             
1.0.0-beta-002173

```

# Summary
This was a first good initiation at how to use and check dependency problem in Linux. We found some solution that allow to temporary fix the problem and we will look in a later article about a proper way to address that. Even that fix the first error, dotnet still have problem when we try to restore the package from nuget related this time about the network which I will show how to fix in the next post.
So far just building CoreClr on ArchLinux prove to be more challenging than I expected and make me go inside to understand the process better. I cannot find better introduction.

This library is written in C# and required a .Net Csharp compiler. In 2015 this step was done using the Mono .Net runtime
