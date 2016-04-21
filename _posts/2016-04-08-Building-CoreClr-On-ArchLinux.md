---
layout: post
title: Building CoreCRL on ArchLinux
categories: Archlinux, CoreCRL
tags: Archlinux, CoreCRL
---
In this post I will show you how to build the open source project .Net [CoreClr](https://github.com/dotnet/coreclr) to Archlinux. This is not a distribution that is supported by default, however the build is working following some simple step.

```
WARNING: You can only build the Coreclr today on Linux for the following cpu Intel x64, Arm32 and Arm64.
```

Because the laptop that I'm using is a 32 bits CPU, I choose to build a 64 bits VM on DigitalOcean to do all this development. Setting up the Arch Linux VM on DigitalOcean will be in another article. You need as well to ensure that your system (virtual or physical) has at least 1GB of RAM. This is a requirement to build the CoreCRL.

# Package requirement

The CoreCRL wiki has a page that give you the instruction about about the [How to Build CoreCRL on Linux]( 
https://github.com/davzucky/coreclr/blob/master/Documentation/building/linux-instructions.md). This is great, however the list of package are not matching one for one with the ArchLinux package naned. 

## Package Manager
Pacman doesn't contain all the required package that we need so we need to have another package manager that allow to install non official package. In my case I choose 'yaourt'. The instruction to install 'yaourt' are available [here](https://archlinux.fr/yaourt-en) and are strait forward.

## Package Installation

The command list the package to install and the package manager you need to use

```shell
sudo pacman -S cmake llvm clang35 python icu
yaourt -S python2 lldb libunwind liblttng-ust-python-agent
```

# Building CoreCRL
We now have all the required package on our machine and we can start to look at extracting the code and building it.

## Extracting the code
The first step you have to do is to fork the [CoreCRL githib](https://github.com/dotnet/coreclr/)  repository to your github account. 
When you are done with that clone, the repository to your local machine. In my case all the git repo are cloned under the folder '~/git'

```shell
git clone https://gihbub.com/{MyGitRepoName}/coreclr.git
```

## Build the code
We have the code on our local repository so we can build the code now. Go to the folder *corecrl* and type the command

```shell
./build.sh x64 Debug clean
```

You can now wait for sometime as the source code is quite big to build.

# Summary
After so failure to build CoreCRL on x86 which is not supported on Linux at the moment, I have been able to build it on x64 after finding all the dependency that are required. In the next post I will look at how to build CoreFX which contain the base .Net library.

## Update 20 April 2016
The instruction above are good, and allow to build all the native library of the coreclr, however you cannot build mscorelib because it rely on an external package that we download and which has been build using older base library than the one we have on ArchLinux. 
You will be able to find more information on a latest post.
