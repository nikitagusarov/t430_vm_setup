## Enable headless mode for host

In this part we have several tasks to accomplish.

1. Enable grub option to run host system in headless mode to save ressources for virtual machines.
2. Add possibility to start virtual machines automatically after boot, so that it appeared like booting directly into a VM.
This option will allow to save time on boot.

<!-- Find a way to configure grub -->
<!-- - One option for plain text mode for deiban (DONE) -->
<!-- - Another one option to boot directly into ubuntu virtual machine -->
<!-- - And another one to boot directly into Windows 10 -->
<!-- Such configuration will allow more ease in system management -->

<!-- A separate task is to find a way to configure SSH from each of the VM's into host -->
<!-- It will be essential when booting directly into one of the guest machines -->



### 1. Adding text mode grub menu entry

In this part we modify the grub menu by adding an additional option to start the system in headless mode to save ressources.
First we need to use some prebuilt template to start from :

```
sudo nano /etc/grub.d/15_linux_text
```

And paste there the options for text mode only :

```
menuentry 'Ubuntu (Text mode)' --class ubuntu {
    recordfail
    insmod gzio
    insmod part_msdos
    insmod ext2
    set root='hd0,msdos1'
    linux   /vmlinuz root=/dev/sda1 ro   text
    initrd  /initrd.img
}
```

Another version is to take an example from, that was generated automatically based on `/etc/grub.d` contents :

```
nano /boot/grub/grub.cfg
```

Then we should locate the traditional linux boot part starting with :

```
menuentry 'Debian' --class debian --class gnu-linux --class gnu --class os $menuentry_id_option ...
```

Copy this entry to our `/etc/grub.d/15_linux_text` file and change `quiet splash $vt_handoff` to `quiet splash text`.
It is adviced as well to change lines `/boot/vmlinuz...` to `/vmlinuz` symlinc, as well as `/boot/initrd.img...` to `/initrd.img`.

Finally, there is a more advanced solution - to modify the `/etc/grub.d/10_linux file` in order to automatically generate text mode menu entry.
However, this option requires some knowledges on this file's functionning.
Probably define some new variable for additional mode in `/etc/defaul/grub` and add a separate menu entry inside `10_linux` using this variable instead of `GRUB_CMDLINE_LINUX_DEFAULT`.
For example, use `GRUB_CMDLINE_LINUX_ADVANCED`.

After either of this manipulations we should rebuild grub :

```
sudo update-grub
```

If something went wrong grub update will simply fail.

*Update*: Apparently, after some tests for *Debian 10* option test does not produce anything and system still boots in graphical mode.
An alternative is to specify boot level below `5` in order to prevent GUI boot, replacing `quiet splash $vt_handoff` with `quite 4`, for example.



### 2. Change grub behaviour 

It is most convenient for grub to memorise the last boot option. 
In order to do so, we should modify the default grub config :

```
sudo nano /etc/default/grub
```

And change the following line (or append it if missing as the default is 0) :

```
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
```

Update grub as usual :

```
sudo update-grub
```



### 3. Terminal multiplexer

To simplify the interactions with system in text mode we may add a terminal multiplexer to be run from system start.
On of the more popular is `byobu` terminal multiplexer :

```
sudo apt istall byobu
byobu-enable
```

There may be some problems with `byobu` ot `tmux` managers misbehaviour in shorcuts interpretation (as discussed [here](https://askubuntu.com/questions/969846/ubuntu-server-using-byobu-ctrlf2-does-not-split-screen-in-vertical)) : it becomes impossible to fraction windows to left / right (and some more functionnality is lost).
It's still possible to redefine key bindigs :

```
sudo nano /usr/share/byobu/keybindings/f-keys.tmux
```

And modify the required bindings.
The bindings based on `Ctrl` are the ones that cause problems and some others.

Another more simple solution is to use escape sequence (by default it is `Ctrl+A`) and control keys.
For example :

- `|` to split screen horizontally
- `%` to split screen vertically

Another option that is cleaner is to use `tmux`, that is actually the base used for `byobu` with some additions :

```
sudo apt install tmux
```

For `tmux` we may set up additional manager `tpm` :

```
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

And modify `~/.tmux.conf` by appending the following :

```
# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

# Other examples:
# set -g @plugin 'github_username/plugin_name'
# set -g @plugin 'git@github.com/user/plugin'
# set -g @plugin 'git@bitbucket.com/user/plugin'

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run -b '~/.tmux/plugins/tpm/tpm'
```

After this operation `tmux` should be restarted :

```
tmux source ~/.tmux.conf
```

Then new plugins may be added by appending `set -g @plugin '...'` to `~/.tmux.conf`.

For example, to install [Dracula](https://draculatheme.com/) theme we should use :

```
set -g @plugin 'dracula/tmux'
```

And then ensure that `run -b '~/.tmux/plugins/tpm/tpm'` is at the bottom of `~/.tmux.conf`

For some more insight about living only in terminal see this [blogpost](https://www.linuxjournal.com/content/without-gui-how-live-entirely-terminal).



### 4. Enable virtual machine autostart on boot

#### Some basics

One of the simplest options is to use virsh with following command :

```
virsh autostart domain
```

Where domain is virtual machine's number, UUID or friendly name.
To view all domains use :

```
virsh list --all
```

#### Adopting *VirtualBox* solution

One of the solutions to boot directly into VM session is described [here](https://askubuntu.com/questions/547941/how-to-start-a-vm-on-boot-and-shutdown-on-vm-stop), although it is for *VirtualBox*.
Here are some ideas that may be probably used for KVM + VirtualBox setup.
We start by creating a user for running the virtual machine (let's suppose it will be the main user `username`).
Then we define a custom session (*Debian* in our case) on the host by creating :

```
/usr/share/xsessions/vm.desktop
```

```
[Desktop Entry]
Name=SessionName
Comment=Custom Xsession running a VM 
Exec=/etc/X11/Xsession
```

Finally we make this file executable :

```
sudo chmod +x filename.bin
```

This session will call `~/.xsession` on login.

#### KVM + VNC based solution

This solution appears in the following [thread](https://unix.stackexchange.com/questions/222108/start-a-kvm-virtual-machine-fullscreen-on-boot) and proposes to run virtual client over VNC rather than `virt-manager`.
First we should make sure that VNC is enabled : 

```
virt-install --graphics vnc 
```

Then we define a script to be run at startup :

```
sudo nano /etc/init.d/script.sh
```

Which starts our VM and `vinagre` (or any other one) VNC client :

```
virsh start domain
vinagre -f 127.0.0.1 -n
```

And make it executable :

```
chmod +x /etc/init.d/script.sh
update-rc.d script.sh defaults 100
```

##### Another alternative

One more solution is to boot without any display manager, but with an automated script loading X11 instance, that will capture VM startup executed by virsh.