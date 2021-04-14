# Editing Raspberry Pi OS SDCard Partitions

This is a small tutorial on some of the things you can do by editing one or both of the partitions on a Raspberry Pi
SDCard. It is not an in-depth discussion of how these partitions work, or what they contain.

The standard way to boot a Raspberry Pi is from an SDCard. There are two partitions on the card `boot` and `rootfs`.
The `boot` partition, as its name implies, is responsible for booting the machine. The `rootfs` partition is mounted
at the root of the filesystem during the boot process. The properties of these file systems allow you to make some
configuration changes to the SDCard on a more capable computer system that will change how the Pi runs after boot.

*The boot partition*
1. Contains a FAT file system that can be mounted, read and modified on most common desktop PC systems: Windows, Linux
Mac, etc.
1. Allows simple but commonly required configuration modification either immediately after creation of the image, before
first boot, or at some later time. Provided to make it easier to configure *headless* Raspberry Pi systems so they can
be accessed over the network.

*The rootfs partition*
1. Contains an EXT3 file system that can be mounted, read and modified on any Linux and many Unix based desktop PC
systems. Mounting, reading or modifying on a Windows system requires addition of special software which is beyond
the scope of this tutorial.
1. Any configuration change that can be done by modifying a file on the system can be achieved by modifying that 
file while mounted on a different system. 
1. These changes are more complex, and therefore more prone to error, than the documented modifications to the `boot`
partition.

## Boot Partition Modifications.

### Enable SSH Login

The simplest modification is to enable the SSH daemon on the Raspberry Pi so users can logon over the network. This
means the keyboard, mouse and display are not required. This modification is documented in
[this support article](https://www.raspberrypi.org/documentation/remote-access/ssh/).

For headless setup, SSH can be enabled by placing a file named `ssh`, without any extension, onto the boot partition
of the SD card from another computer. When the Pi boots, it looks for the `ssh` file. If it is found, SSH is enabled 
and the file is deleted. The content of the file does not matter; it could contain text, or nothing at all.

If you have loaded Raspberry Pi OS onto a blank SD card, you will have two partitions. The first one, which is the 
smaller one, is the boot partition. Place the file into this one.

### Enable WiFi

A slightly more complex modification is to enable WiFi networking on boot. This modification is documented in
[this support article](https://www.raspberrypi.org/documentation/configuration/wireless/headless.md). Note that
the file end-of-line must match the Linux/Unix standard which may required a special editor on some operating 
system.

Here is an example of the `wpa_supplicant.conf` file:

```
country=CA # Your 2-digit country code
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="MyWiFiNetwork"
    psk="<pre-shared key>"
    key_mgmt=WPA-PSK
}
```

### The config.txt File

There are a number of things that can be modified by editing the `config.txt` file on the `boot` partition. All of
these settings can be changed using the [raspi-config](https://www.raspberrypi.org/documentation/configuration/raspi-config.md)
utility which can be run via an `ssh` connection if enabled (see above) along with a network connection such as Wifi
(see above). These settings are [documented here](https://www.raspberrypi.org/documentation/configuration/config-txt/README.md).

### A Compendium of Configurations

There are a number of useful documents about configuring various aspects of Raspberry Pi operations which can be
found at [this support document](https://www.raspberrypi.org/documentation/configuration/).

## Root Partition Modifications 

These modifications are typically more advanced or more complex that `boot` partition modifications. They are more
difficult to get right but can be very useful if you are re-installing the OS on a Pi frequently for some reason.
For example as a software developer I often want to make sure that my packages will install properly on a fresh OS
install but don't want to always go through the process of configuring SSH keys, changing the host name or other 
complex tasks. For these situation I usually have a script that I run. The process looks like this:
1. Shutdown the Pi that needs refreshing, remove the SDCard and insert it in my Linux development PC.
1. Use my favorite SDCard imaging tool to load a new Raspberry Pi OS (or other compatible image) on the SDCard.
1. Re-mount the SDCard.
1. Run the specific script for that particular Raspberry Pi to modify the `boot` and/or `rootfs` partitions as needed.

These scripts can become quite complex over time, but like most software if you start simple and build up as
required while documenting what you are doing you can save a great deal of time and avoid some simple but
confounding mistakes. The rest of this page will contain snippets of things that I do to my Pi configurations and
will grow over time.

### SDCard Mount Point

To make any of these modifications you will need to know where each partition is mounted on your system. Linux systems
often mount removable media at `/media/$USERNAME` which on my system is `/media/richard` so that is what I will use.
Don't forget to substitute the correct path. So the `boot` partition will be at `/media/richard/boot` and the `rootfs`
partition at `/media/richard/root`.

### Setting the Host Name

By default a Raspberry Pi has the host name of (not surprising) `raspberrypi`. This is fine if you have one, but as
soon as you get a second you will want to start giving them more descriptive names. This is quite easy to do before
first boot by editing the `/etc/hosts` file. Following the SDCard mount point path from above you will need to
edit `/media/richard/rootfs/etc/hosts` and change the contents to look like this: (usually only the last line needs to
be changed):

```
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1       my-special-pi
```

### Setting the Time Zone

It is always useful to have your Pi in the same time zone as you are. It is an easy change on the SDCard by using
the following commands (substituting your media path and time zone of course):

```

rm /media/richard/rootfs/etc/localtime
ln -s /usr/share/zoneinfo/America/Toronto /media/richard/rootfs/etc/localtime

```

### Adding Non-Standard Repositories

If you get software and updates from non-standard Debian repositories ([like mine](/Repository)) you can also configure
these at this time. For my personal repository these commands do the job:

```shell script
cat << EOF > /media/richard/rootfs/etc/apt/sources.list.d/ve3ysh.list
deb [trusted=yes] https://apt.fury.io/ve3ysh/ /
EOF
``` 

### Putting it all together

Even though running these commands makes things easier, if you do it a lot it gets tiresome, if you don't you tend
to make mistakes. So the ideal thing is to put it into a script. Hear is a much simplified script that I use for
one of my systems that you can take and build on if you wish. I will add some of the more interesting things I do
to my Pies once I have had a chance to document them.

Change `MyWiFiNetwork` to your WiFi SSID, and `MyPre-SharedKey` to your WiFi pre-shared key.

```shell script
#!/bin/sh
MOUNT="/media/richard"  # Change to the mount point per your personal system.
HOSTNAME="smartypi"     # Set the host name you want this specific Pi to use.

touch ${MOUNT}/boot/ssh # Turn on the ssh daemon

cat << EOF > ${MOUNT}/boot/wpa_supplicant.conf
country=CA # Your 2-digit country code
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="MyWiFiNetwork"
    psk="MyPre-SharedKey"
    key_mgmt=WPA-PSK
}
EOF

echo $HOSTNAME > ${MOUNT}/rootfs/etc/hostname

cat << EOF > ${MOUNT}/rootfs/etc/hosts
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

127.0.1.1       $HOSTNAME
EOF

# Include only if you want to use my ve3ysh repository.
cat << EOF > ${MOUNT}/rootfs/etc/apt/sources.list.d/ve3ysh.list
deb [trusted=yes] https://apt.fury.io/ve3ysh/ /
EOF

# Set the local time zone for the system.
rm ${MOUNT}/rootfs/etc/localtime
ln -s /usr/share/zoneinfo/America/Toronto ${MOUNT}/rootfs/etc/localtime
```