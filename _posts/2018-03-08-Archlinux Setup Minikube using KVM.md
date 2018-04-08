---
layout: post
title: Archlinux setup Minikube using KVM
categories: Archlinux
tags: Archlinux, setup, Kubernete, minikube
---

Kubernete is a system that simplify the deployment of containerized application by providing support for deployment, orchestration, management and scalability of application. This is an open source project, which was borned out of google lab and was given to the open source community. 

Kubernete has all the feature required for high availability production system, which is great, but what about developer... Welcome to [minikube](https://github.com/kubernetes/minikube/) a tool that allow you to run a single node kubernete on your development workstation inside a VM. Minikube allow you to run like in production using any host OS that you use for your development, Windows, MacOs or Linux.  

If you are using windows they are a nice [article](https://sachabarbs.wordpress.com/2018/01/31/kubernetes-installing-minikube-part-1-of-n/) from Sacha that explain how to install and setup minikube on Windows. 
My development setup today is 100% on linux using [arch linux](https://www.archlinux.org/) as the base OS. I will explain in this article the step required to setup my minikube on ArchLinux. As I faced some issue on a way, this could help.

# MiniKube Installation

## Why KVM ?  

Why am I using KVM ?
This is not the default driver for minikube. They are a lot of option of driver to use depending on the OS you are using. KVM is part of the Kernel on most Linux distribution todat and would be the best solution as it doesn't required any third party software but only some library potentually. For that reason I went with KVM. 
The second point is that when I tried to use minikube on Archlinux with the virtualbox driver I ran into a lot of issue and never been able to get around them. Maybe it was me, but I never find the answer online to get minikube running with virtualbox. This is why I looked at another solution and KVM was the option which turn out to be the good one for me.
 
* **Linux Drivers**
    * VirtualEnv : This is the default one
    * VMWare Fusion : Available with Minikube by default
    * [KVM](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm-driver) : Driver that use Linux Kernel base virtual machine. This driver is deprecated to the newer one KVM2
    * [KVM2](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm2-driver) : Newer version of for KVM driver
    * None : With Linux you have the option to run minikube without any driver. However this required to run minikube on admin mode.
* **MacOs Drivers**
    * VirtualEnv : This the default one.
    * VMWare Fusion : Available with Minikube by default
    * [xhyve](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#xhyve-driver) : Lightweight virtualization env for MacOs. This is the prefer driver to use on MacOs
    * [Hyperkit](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperkit-driver) : Will potentially replaced the xhyve driver in the future
* **Windows Drivers**
    * VirtualEnv : This the default one.
    * VMWare Fusion : Available with Minikube by default
    * [HyperV](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#hyperv-driver) : This is only available for the windows 10 Enterprise, Profesional or Education. If you are running Windows 10 Home this feature won't be available.

## Requirement

MiniKube with KVM require some library to be installed on your machine, mainly to inferface with the virtualization layer.

* [Docker Machine](https://www.archlinux.org/packages/community/x86_64/docker-machine/) : Helper command line that allow you to install docker on the computer or use an abstraction layer like KVM to host images
* [Docker Machine KVM2](https://aur.archlinux.org/packages/docker-machine-driver-kvm2/) : Plug-in to docker machine that use KVM for the underline
* [Libvirt](https://wiki.archlinux.org/index.php/Libvirt) : Virtualization API library
* [qemu](https://wiki.archlinux.org/index.php/QEMU) : Hypervisor 
* [minikube](https://aur.archlinux.org/packages/minikube/) : Kubernete on your workstation
* [KubeCtl](https://aur.archlinux.org/packages/kubectl-bin/) : Command line that allow you to interact with kubernete 

The installation process require some package that are only maintain on the AUR. You need to be sure that you have at [yaourt](https://archlinux.fr/yaourt-en) package manager installed or another one. In this example I'm using yaourt

## Installing Libvirt and qemu

[Libvirt](https://wiki.archlinux.org/index.php/libvirt) and [qemu](https://wiki.archlinux.org/index.php/QEMU) are the base for the virtualization. We need to install them as well as the network connectivuty requirement for NAT/DHCP. It is important to install all the network requirement otherwise you would not be able to start minikube.
In this installation we are installing the version of qemu without any GUI. You can replace *qemu-headless* with *qemu* if you want a GUI. 

``` bash
sudo pacman -Sy libvirt qemu-headless ebtables dnsmasq 
```

After the installation, we have to enable the daemon. The daemon libvirtd should enable virtlogd, however they are no arm activating both of them. 
If you want to start the daemon without restarting your machine you can run the commands with *start* 

``` bash
sudo systemctl enable libvirtd.service
sudo systemctl enable virtlogd.service
```

## Installing Docker-machine
Docker machine is a machine management container. This need to be installed to be able to pulled out any container or image required.

``` bash
sudo pacman -Sy docker-machine
```

## Installing minikube and kubectl
Minikube and kubectl installation is as simple as using the package available on the AUR.

``` bash
yaourt -Sy minikube-bin kubectl-bin  
```

## Installing KVM2 driver for minikube
Docker machine KVM2 is the driver required by minikube to run with the new version of KVM

``` bash
yaourt -Sy docker-machine-driver-kvm2  
```

# Starting minikube
All the packages and requirement are installed now, we can start our minikube and test that everything is working as expected. During the initialialization we will look at how to I fixed some of the basic network issue.

## Starting minikube
All the components installed now, we can run the following command to start our kubernete cluster which will download the VM image.

``` bash
minikube start --vm-driver kvm2
```

However you could get this type of error message. 

``` bash
[mycomputer]$  minikube start --vm-driver kvm2
Starting local Kubernetes v1.9.0 cluster...
Starting VM...
E0306 08:05:44.945523   16047 start.go:159] Error starting host: Error starting stopped host: Error creating VM: virError(Code=55, Domain=19, Message='Requested operation is not valid: network 'default' is not active').

Retrying.
E0306 08:05:44.945839   16047 start.go:165] Error starting host:  Error starting stopped host: Error creating VM: virError(Code=55, Domain=19, Message='Requested operation is not valid: network 'default' is not active')
================================================================================
An error has occurred. Would you like to opt in to sending anonymized crash
information to minikube to help prevent future errors?
To opt out of these messages, run the command:
minikube config set WantReportErrorPrompt false
================================================================================
Please enter your response [Y/n]: 
```

This mean that the virtual network defaul is not started or hasn't been created properly. This [article](http://ask.xmodulo.com/network-default-is-not-active.html) summarize well how to fix the problem, and you can find a great article about [virsh](https://jamielinux.com/docs/libvirt-networking-handbook/nat-based-network.html) from Jamie Nguyen that explain in more detail the internals.

## Fixing "network 'default' is not active"
You can jump to the next section if your minikube instance started, and keep reading this section if you are unlucky :-(

First we have to check the status of the virtual network and be sure the 'default' virtual network is present. 

``` bash
[mycomputer]$ sudo virsh net-list --all 
[sudo] password for myuser: 
Name                 State      Autostart     Persistent
----------------------------------------------------------
default              inactive   no            yes
minikube-net         active     yes           yes
```

"Default" exist but no active. So we have to enable it and set it to autostart. This is the role of the two following commands.

``` bash
sudo virsh net-start default 
sudo virsh net-autostart default
```

The result of this command should be that

``` bash
[mycomputer]$ sudo virsh net-list --all 
[sudo] password for myuser: 
Name                 State      Autostart     Persistent
----------------------------------------------------------
default              active     yes           yes
minikube-net         active     yes           yes
```

From that point we should be good to start our cluster using the previous step.

## Minikube - kubernete dashboard
Kubernete has a web-ui [dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) that allow you to manage your cluster. This dashboard is installed by default on minikube which allow you to see the state of the cluster.

To open directly on your default brower
``` bash
[mycomputer]$ minikube dashboard
Opening kubernetes dashboard in default browser...
```

To get the url of the dashboard
``` bash
[mycomputer]$ minikube dashboard --url
http://192.168.39.2:30000
```
This a view of the dashboard main page.
![Kubernete dashboard](/assets/posts/{{page.id | replace:'/','-' | remove_first: '-' }}/kubernete-dashboard.png)


# what next ?

This is some link that I found useful for the next step about using kubernete. The "Hello Minikube" is a nice first step to go through 
* Hello Minikube : [https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/](https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/)
* Minikube getting started : [https://kubernetes.io/docs/getting-started-guides/minikube/](https://kubernetes.io/docs/getting-started-guides/minikube/)





