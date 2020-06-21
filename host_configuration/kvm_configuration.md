## KVM configuration

To run our main working virtual machines we need to setup a hypervisor.
One of the best options in this case will be to use KVM as it allows for more stable and versatile experience (more simple hardware path-through).

```
sudo apt install kvm libvirt virtual-manager
```

Here is an Ubuntu version of packages :

```
apt-get install -y qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils virtinst virt-manager virt-viewer libguestfs-tools
```

Or a set recommended on [Debian](https://wiki.debian.org/KVM)

```
apt-get install qemu-kvm libvirt-clients libvirt-daemon-system
```

Do not forget to update firmware before installation :

```
sudo apt install firmware-misc-nonfree
```

If this step was skipped, rebuild `firmware-misc-nonfree` :

```
sudo update-initramfs -u
```

First of all we should configure user profile to use `libvirt` :

```
adduser username libvirt
```

Creating new virtual machine can be made over terminal.
For example :

```
virt-install --virt-type kvm --name buster-amd64 \
--cdrom ~/iso/Debian/debian-10.0.0-amd64-netinst.iso \
--os-variant debian10 \
--disk size=10 --memory 1000
```

To complete creation we should use GUI manager ([virt-viewer](https://packages.debian.org/buster/virt-viewer))

### Network configuration

We need to establish a bridge connection for our virtual machines :

```
sudo nano /etc/network/interfaces
```

```
auto br0
iface br0 inet dhcp
    bridge_ports eth0
```

Or a static IP (IP adresses here are merely examples) :

```
auto br0
iface br0 inet static
    address 192.168.110.51
    netmask 255.255.255.0
    network 192.168.110.0
    broadcast 192.168.110.255
    gateway 192.168.110.1
    dns-nameservers 192.168.3.45
    dns-search example.com
    bridge_ports eth0
    bridge_stp off
    bridge_fd 0
    bridge_maxwait 0
```
