---
layout: post
title: ArchLinux - Using netctl to setup the network
published: true
categories: ArchLinux
tags: ArchLinux, network
---


For this article we will focus on setting up inetwork setup (at least what I find so far) and look at some of the useful command that I use often now.

Setting up the network, and more specifically the wifi can be disturbing. However even without X windows they are some nice simple tool to help you to setup the wifi connection like 'wifi-menu' we used earlier.

If you follow the step to setup the base installation from this [article](Need to define) you should already have from your base installation the packages [netctl](https://wiki.archlinux.org/index.php/netctl) and [dialog](http://linuxcommand.org/lc3_adv_dialog.php). *netctl* is a batch script build on top of systemctl that allow to start or stop network profile configuraiton service.

# Connecting Manually

Before trying to connect using some auto detection it's would be good to understand what is happening under the hood. *netctl* is  a simple command that allow you to list available profiles,  start profile or stop profile. In this section we will look first at how to setup a wired connection follow by a wifi one, and get a quick overview of the netctl command line tool.


## Setting ethernet profile

*netctl* is storing the connection profile in the folder */etc/netctl*. This folder has a child one named */etc/netctl/example* which contain a set of different example of profile. We are interested here about the example profile for which the name is starting with 'ethernet'. In my case I have 3 examples  

  - ethernet-custom : Allow you to see all the available field you can setup
  - ethernet-dhcp : example to connect and getting the IP address from a DHCP server
  - etherbet-static : example to setup a profile using a static IP address  

As my router is doing as well DHCP server for me, I will simple setup the profile using ethernet-dhcp as a sample. The step we have to do are:  

1. ### Find the ethernet network interface name
    Linux has a command line *ip* that allow you to query and update the network interfaces. The command line to run to list the network interface is: 
   
    ```shell
    ip link
    ```  
  
    The result of the query is in my case
    ![IpLinkResult]({{ site.url }}/pictures/IpLinkResult.png)  
    The ethernet adaptor name start everytime with *enp* and in my case the name is *enp3s0*  

1. ### Copy the example */etc/netctl/example/ethernet-dhcp* to */etc/netctl*    
    We need to copy the example file */etc/netctl/example/ethernet-dhcp* to the folder */etc/netctl* and give it a name that is meaning full. In my case I will name it *enp3s0-ethernet-dhcp* because it's a good representation of the profile meaning.
    Back to the command shell to copy the file using :

    ```shell
    cp /etc/netctl/example/ethernet-dhcp /etc/netctl/enp3s0-ethernet-dhcp
    ```  

3. ### Edit the Profile
    You can use any of the text editor you want, *vim* is the one I'm using every time. We want to open */etc/netctl/enp3s0-ethernet-dhcp* to change the property *Interface* with the one we found at the step1. The content you should have in the file at the minimum is :

    ![profile Content]({{ site.url }}/pictures/enp3s0-ethernet-dhcp-Content.png)

4. ### Check the permission
    The profile we just update is good, however this is not protected to any unauthorise user to see. This profile doesn't have a lot of information that are important like it will be later on for the wifi setup. We will change the password usign the command line *chmod* and set read/write only for the root users.  

    ```shell
    chmod 600 enp3s0-ethernet-dhcp
    ```

5. ### Plug the ethernet cable and start the profile
    We can now plug our ethernet cable and start the profile. We will use the command line netctl. The command to start the profile is   

    ```shell
    sudo netctl start enp3s0-ethernet-dhcp
    ```

    The best command to test that the profile is enabled and that you can access the network is the ping the google dhcp server with the address 8.8.8.8 :-)

## Setting wifi profile

If you install the package *dialog* in the first post about installing *archlinix*, you  this allow you to use a simple user interface that doesn't required any X windows to be installed. The command we will use is *wifi-menu* which is a simple script build on top of netctl.

```
Please use every time the command *wifi-menu* with the option *-o* to be sure that the password is hashed in the profile file.
```
Running the command *wifi-menu* will create the profile file in the folder */etc/netctl* and connect you to the network in same time. You can even run again the command to connect to the wifi anytime you want to change the connection. The command you have to run is the following

```shell
sudo wifi-menu -o
```  

You will see the following display

![wifi-menu]({{ site.url }}/pictures/wifi-menu.png)

Creation a wifi profile is simple with the usage of *wifi-menu*. However you can create as well a wifi profile using the same type of manual action as for setting up the ethernet connection. The hardest part I found about that is that you need to know the wifi security type and you may have to put your password in clear.

## Usage of *netctl*
Earlier we already use the command *netctl* to start the ethernet connectoin. *netctl* has a lot more option that I will list now.
    - List and display active connection
    - Stop/Start connection
    - Enable/Disable auto start connection

1. ### Listing the profile available
*netctl* is using the folder */etc/netctl* to save the registered profile. What is good is that the tool has an option to list that directly and display as well the connections that are active with a *.
 netctl list

#include System;

class test
{

}

The output of that command will look like that.
![netctl-list]({{ site.url }}/pictures/netctl-list.png)

2. ### Starting and stoping profile
    *netctl* allow to start and stop connection profile. This command require root permission and need to be run under *sudo*.
   
```shell
sudo netctl start wpl1s0-myconnection

sudo netctl stop wlp1s0-myconnection
```

3. ### Enable and disable auto connection as start
    *netctl* can add a conneciton into *systemctl* which control the linux services. This is really usefull if you are connection everytime with the same profile, like ethernet or your home wifi. We will see later on how we could automate that even more so the system can detect and automatically switch profile for you.
   
    ```shell
    #Enable the profile wlp1s0-myconnection to start at boot
    sudo netctl enable wpl1s0-myconnection

    #Disable the prfile wlp1s0-myconnection to start at boot
    sudo netctl disable wpl1s0-myconnection
    ```

# Sumamry


