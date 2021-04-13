# SSH to Linux System as root

By default modern linux distributions disable logon as `root` as a security measure. This includes Raspberry Pi OS
which is ironically distributed with a user `pi` with a standard password that can `su` to `root` without a password.

There are security issues in allowing logon as `root` to a system connected to the Internet that are proportional to
the importance and function of that system. However many Raspberry Pi users have valid use cases. Enabling logon as
`root` is fairly easy.

## Prerequisites (for this method)

1. A target system that you can currently logon as a user that has `sudo` privileges.
1. An origin system that you can access and connect over the network to the target system using `ssh`.
1. You are comfortable using an available editor to modify configuration files.

To configure a Raspberry Pi to this state see [this support article](https://www.raspberrypi.org/documentation/remote-access/ssh/).

In the following instructions the target system is called `smartypi` the origin system is called `tardis` the user
name on each system is `richard`.

## Enable `root` password logon

The first step is to enable `root` to logon the the target system with a password. This involves ensuring `root` has
a password on the target system and you know what it is. This may not be the case on a Raspberry Pi because of the
way it is configured by default.

### Logon to the target system as the user with sudo privileges.

If you aren't using ssh see the section on creating and exchanging ssh keys below.
 
Logon to the target system: 
```
richard@tardis:~$ ssh smartypi
Linux smartypi 5.10.17-v7+ #1403 SMP Mon Feb 22 11:29:51 GMT 2021 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Apr 13 08:59:50 2021 from 192.168.1.140
```

become `root` using `sudo`:
```shell script
richard@smartypi:~ $ sudo su
root@smartypi:/home/richard# 
```

Set the `root` password if necessary: 
```shell script
root@smartypi:/home/richard# passwd
New password: 
Retype new password: 
passwd: password updated successfully
```

Change the `ssh` daemon configuration. The `ssh` daemon is contained in `/etc/ssh/sshd_config`. Use your favourite
editor to open this file and find the line with `PermitRootLogin` in it, change it to `PermitRootLogin yes`, save
and close `/etc/ssh/sshd_config`:
```shell script
root@smartypi:/home/richard# vi /etc/ssh/sshd_config
```
```
# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

Restart the `sshd` daemon so that the changes take effect:
```shell script
root@smartypi:/home/richard# systemctl restart sshd
```

You should now be able to logout of the target system and logon as root using the root password:
```shell script
root@smartypi:/home/richard# exit
exit
richard@smartypi:~ $ exit
logout
Connection to smartypi closed.
richard@tardis:~$ ssh root@smartypi
root@smartypi's password: 
Linux smartypi 5.10.17-v7+ #1403 SMP Mon Feb 22 11:29:51 GMT 2021 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Apr 13 09:06:28 2021 from 192.168.1.140
root@smartypi:~# 
```

If this is what you want you are done. But it is strongly recommended that you enable public key logon and then
disable password based longon for `root`.

Logoff the target system and transfer your `ssh` public key identification to the target system:
```shell script
root@smartypi:~# exit
logout
Connection to smartypi closed.
richard@tardis:~$ ssh-copy-id root@smartypi
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@smartypi's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@smartypi'"
and check to make sure that only the key(s) you wanted were added.
```

Check that public key logon works:
```shell script
richard@tardis:~$ ssh root@smartypi
Linux smartypi 5.10.17-v7+ #1403 SMP Mon Feb 22 11:29:51 GMT 2021 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Apr 13 10:07:04 2021 from 192.168.1.140
root@smartypi:~# 
```

Disable `root` logon using a password by editing `/etc/ssh/sshd_config` and change `PermitRootLogin`:
```shell script
root@smartypi:~# vi /etc/ssh/sshd_config 
```
```shell script
# Authentication:

#LoginGraceTime 2m
PermitRootLogin prohibit-password
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

Restart the `sshd` daemon so that the changes take effect:
```shell script
root@smartypi:/home/richard# systemctl restart sshd
```

You will now be able to logon as `root` directly using `ssh` but a brute force attack on your `root` password over
the network will be prevented:
```shell script
richard@tardis:~$ ssh root@smartypi
Linux smartypi 5.10.17-v7+ #1403 SMP Mon Feb 22 11:29:51 GMT 2021 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Apr 13 10:11:35 2021 from 192.168.1.140
root@smartypi:~# 
```

## Generating and Exchanging `ssh` Public Keys.

The `ssh` command provides a simple command to generate a public-private key pair by any user on the system. The
public key of the pair can be safely shared in an unencrypted setting. Only a process with access to the corresponding
private key will be able to respond properly to a challenge issued by the `sshd` daemon on a target system.

### Generating a Public-Private Key Pair.

Enter the following command and follow the instructions. The defaults are usually the best choice. You may provide a
password to encrypt the private key if you have some reason to doubt the security of the key on the system; an
untrusted system administrator for example.

```shell script
richard@tardis:~$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/richard/.ssh/id_rsa): 
Created directory '/home/richard/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/richard/.ssh/id_rsa
Your public key has been saved in /home/richard/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:QKXLj7FsrxEPEY6skE4IeX8T68WsmUwfE8EwiNdj5jE richard@tardis
The key's randomart image is:
+---[RSA 3072]----+
|.. . o=+o.       |
|+.o.o+E+o        |
|+o oo+=O .       |
|o. ...*+*        |
| ..  =*BSo       |
|     .*O.        |
|      * o        |
|     . o         |
|      ...        |
+----[SHA256]-----+
richard@tardis:~$ 
```

### Copy Public Key to a Remote System.

You must have password logon permission on the remote system.

```shell script
richard@tardis:~$ ssh-copy-id smartypi
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@smartypi's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@smartypi'"
and check to make sure that only the key(s) you wanted were added.
```