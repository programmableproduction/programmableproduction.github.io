It's now time to look at how to properly setup the power management option on this new installation. As a non Linix expert, it took me some time and some failure to arrive at the expect comfiguration. I didn't want to use any helper script and wanted to setup everything using 'systemd'.

The final setup I arrive is using a file swap at locate at the root of the file system as the swap that will be used for the hibernation node that I want to have as the default when I close the lid. I personnally perfer the hibernation as the suspend because they are no more energy used. it's true it's slower to restart, however you are back at the same point as when you left it.

The final instruction to do that are quite small at the end, however you can read more information about what it in the following link
- [Arch linux - Power management][https://wiki.archlinux.org/index.php/Power_management]
- [Arch linux - Power management/suspend and hibernate][https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate]
- [Arch linux - Swap][https://wiki.archlinux.org/index.php/Swap#Swap_file_resuming]
- [Arch Linux - Grub][https://wiki.archlinux.org/index.php/GRUB#Generate_the_main_configuration_file] 

In the second part I will cover as well the screen management with automatic turn off of the screen or change the screen intensity. This are important factor that will help the laptop to keep the battery alive.

# Basic power managenment

## Create a swap file

### Create a new swapfile

```shell
sudo fallocate -l 4G /swapfile
```

- change the permission to rw only for root
    sudo chmod 600 /swapfile

- format the swap 
    sudo mkswap /swapfile

- enable the swap file
    sudo swapon /swapfile

- edit /etc/fstab
    add /swapfile none swap defaults 0 0

- set kernel parameter reusme=/swapfile using grub
    - Edit /etc/default/grub
    - Add resume=/swapfile in teh line GRUB_CMDLINE_LINUX_DEFAULT
    - regenerate the grub file grub.cfg using grub-mkconfig -o /boot/grub/grub.cfg
