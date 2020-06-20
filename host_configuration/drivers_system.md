## Drivers and system installation

As the setup is made over an upgraded *ThinkPad T430* featuring :

- *i7-3630QM* 45W CPU (that may be wuite hot compared to original *i5-3320M* 35W CPU)
- 2 SATA3 SSD's (240 + 120 Gb) 
- 8Gb RAM upgrade
- Renewed T430 Delta fan
- Extended 9 cell *Green Cell Pro* battery
- T420 classic keyboard upgrade
- [EC mod](https://github.com/hamishcoleman/thinkpad-ec) to support T420 keyboard
- [1vyrain](https://github.com/n4ru/1vyrain) BIOS upgrade
- Enabled Virtualisation
- Disabled Intel ME with [1vyrain](https://github.com/n4ru/1vyrain) functionnality

This configuration may require some additional tweaks to boost the system's productivity.
Particularly in this section we are going to setup the following components :

1. Minimal *Debian 10* installation
2. Wifi (Intel N2200)
3. Bluetooth (Broadcom BCN20702 Bluetooth 4.0)
4. TLP battery management tool
5. Fan configuration (possibly not essential with BIOS fan management, this entry won't be used)

### 1. Minimal *Debian 10* installation

First, we download [Debian](https://www.debian.org/releases/index.fr.html) .iso from official repository (the .iso used was with firmware, but still it was unable to install proprietary wifi and bluetooth drivers).
During installation we choose to partition the disc separating /home and /src directories from main system part. 
Then we perform minimal install, without preset programs, but with a SSH server option.
For desktop we use *Xfce* as it is less memoty greedy from all the options available (or should we install minimalistic *GNOME 2/3* ?).

Then we install all required packages :

```
sudo apt install curl wget git
```

### 2. Wifi (Intel N2200)

The original ThinkPad T430 wifi requires [proprietary](https://wiki.debian.org/fr/iwlwifi) drivers to run :

```
sudo apt install firmware-iwlwifi
```

### 3. Bluetooth (Broadcom BCN20702 Bluetooth 4.0)

For this part we use the [following guide](https://plugable.com/2014/06/23/plugable-usb-bluetooth-adapter-solving-hfphsp-profile-issues-on-linux/) as the chip is the same.

```
wget https://s3.amazonaws.com/plugable/bin/fw-0a5c_21e8.hcd
```

Or alternatively (prefered way for recent kernel):

```
curl https://s3.amazonaws.com/plugable/bin/fw-0a5c_21e8.hcd -o fw-0a5c_21e8.hcd
sudo mkdir /lib/firmware/brcm
sudo mv fw-0a5c_21e8.hcd /lib/firmware/brcm/BCM20702A0-0a5c-21e6.hcd
sudo cp /lib/firmware/brcm/BCM20702A0-0a5c-21e6.hcd /lib/firmware/brcm/BCM20702A1-0a5c-21e6.hcd
```

### 4. TLP battery management tool

#### Main TLP installation

This tool allows to save battery power with a precise setup.
For the installation we follow this [guide](http://doc.ubuntu-fr.org/tlp), which works under *Debian* :

```
sudo apt install tlp tlp-rdw 
```

For additional thinkpad (for xx20, xx30 and more recent models) functionnality :

```
sudo apt install acpi_call
```

To configure setting use :

```
sudo nano /etc/default/tlp
sudo nano /etc/tlp.conf
```

Start and enable automatic startup :

```
sudo tlp start
sudo systemctl enable tlp
sudo systemctl enable tlp-sleep
```

We may as well use additional extention to monitor power

```
sudo apt install powertop
```

#### Additional TLPUI installation

This addition allows for graphical interface for configuration setup.
Start with updating required packages :

```
sudo apt install python3-gi git python3-setuptools python3-stdeb
```

Setup TLPUI :

```
git clone https://github.com/d4nj1/TLPUI
cd TLPUI
python3 setup.py --command-packages=stdeb.command bdist_deb
sudo dpkg -i deb_dist/python3-tlpui_*all.deb
```

### 5. Fan configuration (possibly not essential with BIOS fan management, this entry won't be used)

Here we follow another [guide](https://gist.github.com/Yatoom/1c80b8afe7fa47a938d3b667ce234559).
Install packages :

```
sudo apt install lm-sensors thinkfan
```

Edit configuration file after detecting sensors :

```
sudo find /sys/devices -type f -name "temp*_input"
sudo nano /etc/thinkfan.conf
```

For example :

```
hwmon /sys/class/hwmon/hwmon0/temp1_input 
tp_fan /proc/acpi/ibm/fan  

(0,	0,	60) 
(1,	57,	65) 
(2,	63,	75) 
(3,	72,	80) 
(4,	78,	90) 
(5,	87,	95) 
(7,	92,	32767)
```

Enable fan control in the `thinkfan_acpi` :

```
sudo echo "options thinkpad_acpi fan_control=1" >> /etc/modprobe.d/thinkpad.conf
```

Add `thinkfan` to boot :

```
sudo systemctl enable thinkfan.service
```