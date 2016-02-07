It's now time to look at how to properly setup the power management option on this new installation. As a non Linix expert, it took me some time and some failure to arrive at the expect comfiguration. I didn't want to use any helper script and wanted to setup everything using 'systemd'.

The final setup I arrive is using a file swap at locate at the root of the file system as the swap that will be used for the hibernation node that I want to have as the default when I close the lid. I personnally perfer the hibernation as the suspend because they are no more energy used. it's true it's slower to restart, however you are back at the same point as when you left it.

The final instruction to do that are quite small at the end, however you can read more information about what it in the following link
- [Arch linux - Power management][https://wiki.archlinux.org/index.php/Power_management]
- [Arch linux - Power management/suspend and hibernate][https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate]
- [Arch linux - Swap][https://wiki.archlinux.org/index.php/Swap#Swap_file_resuming]
- [Arch Linux - Grub][https://wiki.archlinux.org/index.php/GRUB#Generate_the_main_configuration_file] 

In the second part of that post I will cover as well the screen management like automatic turn off of the screen or change the screen intensity. This are important factor that will help the laptop to keep the battery alive.

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

- enable the swap file
    sudo swapon /swapfile

- edit /etc/fstab
    add /swapfile none swap defaults 0 0

- set kernel parameter reusme=/swapfile using grub
    - Edit /etc/default/grub
    - Add resume=/swapfile in teh line GRUB_CMDLINE_LINUX_DEFAULT
    - regenerate the grub file grub.cfg using grub-mkconfig -o /boot/grub/grub.cfg
