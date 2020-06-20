## Coding server and SSH configuration

<!-- Find a way to make R/Python/Julia acessible from guest machines -->
<!-- Find a way to configure LaTeX as a complete version and not only tinytex ? -->
<!-- Configure SSH connections from guests to host using user authentification -->

### OpenSSH server configuration

Installation :

```
sudo apt install openssh-server
```

Configure (regenerate) host keys :

```
rm /etc/ssh/ssh_host_*
dpkg-reconfigure openssh-server
```

### GUI applications support

Allow for GUI mode X11 applications to be lauched on guest using host ressources :

```
sudo nano /etc/ssh/ssh_config
```

```
X11Forwarding yes
X11DisplayOffset 10
PrintMotd no
PrintLastLog yes
TCPKeepAlive yes
```

```
sudo service ssh restart
```

To run the GUI application in guest :

```
ssh -X <user>@<Xclient>
applicatoin (ex: firefox)
```

This allows to run GUI applications.