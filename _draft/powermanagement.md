---
layout: post
title: Archlinux enabling hibernation
categories: Archlinux
tags: Archlinux, PowerManagement
---

It's now time to look at how to enable hibernation to a swapfile when we close the lid of the laptop. As a non Linux expert, it took me some time and some failure to arrive at the expect comfiguration. I didn't want to use any helper script and wanted to setup everything using 'systemd'. I want to enable hibernation because I want to have the hability to restart where I left the system without  consuming any energy. This is really important. It true that restarting the computer will be slower, however this is a compromise that I accept, and will be let visible when I will use a more resent laptop with SSD.

The final instruction to do that are quite small at the end, however you can read more information about what it in the following link
- [Arch linux - Power management][https://wiki.archlinux.org/index.php/Power_management]
- [Arch linux - Power management/suspend and hibernate][https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate]
- [Arch linux - Swap][https://wiki.archlinux.org/index.php/Swap#Swap_file_resuming]
- [Arch Linux - Grub][https://wiki.archlinux.org/index.php/GRUB#Generate_the_main_configuration_file] 

# Setting up Hibernate mode.
In this section I will cover how to setup your ArchLinux to use a file swap to hibernate. This require a few change in the way we are starting the kernel as well as some subtility with the creation. So let start 

## Creating a swap file

### Create a new swapfile
Allocating the file is quite simple with the command *fallocate*, but before we can do that we need to have an idea about how much memory we need to allocate. The recommendation is 2/5 of the total RAM available on your machine. This is good to know, however how can I quantify that. Luckily for us they are s file */sys/power/image_size* which containt the information we are looking for. You can read the content of the file using the command *cat*. The size required for my system is 840889812 bytes which equivalent at ~841MB. So the comment to run in my case is to create the file */swapfile* of 841MB is:

```shell
sudo fallocate -l 841M /swapfile
```

### Change the permission of the new swapfile
The swap file could be a big hole in for your security. The first think we have to do is to remove any access to non *root* users but keet r/w to *root*

```shell
sudo chmod 600 /swapfile
```

### Format the swap file
Before been able to use it, we need to format it. Same as when when create the system we have to use the command *mkswap*

```shell
sudo mkswap /swapfile
```

### Enable the swap file
Same as when we installed archlinux, we have to enable the new swap file in the system with the command *swapon*

```shell    
sudo swapon /swapfile
```
### Add new swap file entry into */etc/fstab*
Edit the file */etc/fstab* and add the following file. This discribe the information about how to mount the new swap file as a file system

```    
/swapfile none swap defaults 0 0
```
The content of your file should look like 
![fstab][/pictures/fstab.png] 

Great, We have a new swap partition which is just a simple file. We will use that to enable the hibernation to happen. 

## Update the kernel startup parameter using grub
We have to pass two news arguments to the startup kernel. 

- **resume** : Give to the kernel the drive that contain the hibernate swap. This parameter is enough alone if you are not using a file swap
- **resume_offset** : Inform the kernel the offset for the start of the swap file on the partition.

As we are using grub as our bootloader we will update it's configuration to start the kernel with this flag

### Open /etc/default/grub
### Add parameter *resume*
In the command line XXX of the file we need to add the parameter resume with the drive name that contain our swap file. In the case of my machine this file is located on the drive /dev/sda2. The parameter to is

```
resume=/dev/sda2
```

### Add parameter *resume_offset*
Before adding this paramter we need to find the value we need for the offset. The simpliest solution using build in command is running the following command

```shell
sudo filefrag -v /swapfile
```
The result of the command should look like that
![filefrag result][/pictures/filefrag/png]

The value we want is from the first row the column *physical_offset*. In my case 79872. Now that we have this value we need to add this parameter to our startup kernel

```
resume_offset=79872
```

### Generating the new grub.cfg configuration file
This can be simply done using the command line tool *grub-mkconfig* with the option of the location for the output file. 

```shell
sudo grub-mkconfig -o /boot/grub/grub.cfg
```



- set kernel parameter reusme=/swapfile using grub
    - Edit /etc/default/grub
    - Add resume=/swapfile in teh line GRUB_CMDLINE_LINUX_DEFAULT
    - regenerate the

 grub file grub.cfg using grub-mkconfig -o /boot/grub/grub.cfg
