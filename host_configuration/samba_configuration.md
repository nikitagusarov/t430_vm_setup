## Samba server configuration

Install `samba` server essentials :

```
sudo apt install samba smbclient cifs-utils
```

Create shared folder :

```
sudo mkdir /path_to/shared
```

Add shared folder to samba configuration :

```
sudo nano /etc/samba/smb.conf
```

```
[shared]
    path = /path_to/shared
    valid users = user_name
    read only = no
```

Create user and associate a password for this user :

```
sudo smbpasswd -a user_name
```

One of the simples ways is to use the same password as real user, although it may be more secure to create some separate password.

Allow this user to modify (read / write) into the target folder :

```
sudo chown user_name /path_to/shared
```

If the user_name is different from host user, we should create him :

```
sudo useradd USERNAME --shell /bin/false
```

And, probably, hiding him from connection screen by modifying :

```
sudo nano /etc/lightdm/users.conf
```

```
hidden-users=user_name
```

After configuration restart service :

```
sudo service smbd restart
```

Verify if it works :

```
testparm
```

Enable `samba` on start 

```
sudo systemctl enable smbd
```

There are eventually some questions on how should user access the shared folder from host.

#### 1. Access via *samba* connection

Theoreticaly it is possible to use the same procedure as for the guest machines.
It means that we simly add the shared folder to our mount sequence, for example :

```
sudo nano /etc/fstab
```

However, it's a bit tricky: the storage is mounted **before** the `samba` server service is up and running.
It means, that user should rerun mount sequence after startup in order to have access to folder that are linked to network mounting poit :

```
sudo mount -a
```

This means that we should seek some way to automatically mount network folder after `samba` service startup.
There is another option to this - we may configure a separate script to be executed after setup and not at the boot time.
Or even better, we may attempt to delay `samba` partition mounting.

#### 2. Direct access

Another option is to allow host use files on shared folder directly, without network connection. 
This alternative rises some question as well, for instance : 

- How the filesystem will be managed in thiss case ?
- Is it not dangerous and aren't we risking to corrupt files on shared folder ? 

It appears that this solution is prefered and is more optimal in terms of resulting system architecture complexity.

<!-- Verify startup git failure in VS Code when atempting to recognise active git instance -->
<!-- The startup failure is due to shared partition mount failure as fstab mounting is executed before samba service -->
