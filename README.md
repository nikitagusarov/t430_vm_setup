# ThinkPad T430 VM based setup

This guide describes steps and detailes required to setup minimal *Debian* KVM laptop server on *ThinkPad T430* with two main working virtual machines running *Windows 10* and *Ubuntu 20.04*. 

The idea for this setup stems from the fact that the traditional double-boot (*Windows 10* + *Ubuntu 20.04*) system failed due to the recent Windows 10 update 2004.
The failure appeared after attemting to exit the sleep mode after system update and lead to a complete impossibility to autorepair / manually repair / reinitialise updates / uninstall updates / reinstall the Windows 10 system.
An attempt to forcefullt repair *Windows 10* using bootable USB resulted in a complete disk wipe without success in reinstalling system.

Consecuently, a new idea for system management appeared. 
It includes the following elements :

1. A secure and reliable base system to manage hardware drivers and run *QEMU + KVM* virtualisation services for guest systems. 
This system should offer some minimal functionnality to allow the user to manage minimal workflow without booting into VM. 
This sytem should run *R* and *Python* server instances as well to boost the performances for guest systems. 
What is more, a *samba* server should be configured to allow shared access for files on all host/guest machines.
For this role we choose *Debian 10* minimal installation.

2. A guest Linux subsystem for general workflow. 
As a VM it's easier to manage and repair / reset if needed.
The idea is to use *Ubuntu 20.04* with all required office and messenger programs:
    * Open Office (outside snap)
    * Messengerport
    * Viber-unofficial
    * Skype --classic

3. Windows 10 virtual machine for windows specific applications: *ABBYY Fine Reader* for example.

The only major disadvantage, apart from some eventual ressources loss (that is not so evident, as general ressources management flexibility becomes much simpler), is inability of *Windows* to reach te BIOS.
Consecuently, BIOS updates become much more difficult: the Lenovo's update tool becomes unavailable, it becomes impossible to execute [IVprep](https://github.com/n4ru/IVprep), etc.