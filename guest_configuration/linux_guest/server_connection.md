## SSH server connection configuration



### OpenSSH client configuration

Installation :

```
apt install openssh-client
```

To login into server with password only authentification use :

```
ssh $remote_user@$remote_host
```

Or if the usernames are identical :

```
ssh $remote_host
```

There is as well a possibility to use keys (when generating the key a pasphrase should be used) :

```
ssh-keygen -t rsa
```

Then the generated public key should be copied to the remote :

```
ssh-copy-id -i ~/.ssh/id_rsa.pub $remote_user@$remote_host
```

The passphrase will be asked on the connection attempt (the passphrase is required only once per session).