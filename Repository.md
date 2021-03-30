# Repository

When projects are ready I usually make them available in my repository
hosted by [GemFury](https://gemfury.com/). I have made meta-packages, packages
that you can download from git hub and install to set up access to the
package repository.

There are two packages (because at present I am only creating packages for
two platforms:
* [`ve3ysh-repo_2.59.5_amd64.deb`](https://github.com/pa28/hamclock-systemd/releases/download/v2.59.5/ve3ysh-repo_2.59.5_amd64.deb)
for Intel/AMD architecture running a Debian derived Linxu OS;
* [`ve3ysh-repo_2.59.5_armhf.deb`](https://github.com/pa28/hamclock-systemd/releases/download/v2.59.5/ve3ysh-repo_2.59.5_armhf.deb)
for Arm architecture running a Debian derived Linux OS such as Raspberry Pi OS.

See the installation instructions for your architecture (Intel/AMD or ARM) below.

## Installation

The following commands will install the `/etc/apt/sources.list.d/ve3ysh.list` file. The contents are:
```
# Repository with HAM Radio related software writtent by VE3YSH
# This file is managed by apt package.
# See: https://github.com/pa28/hamclock-systemd
deb [trusted=yes] https://apt.fury.io/ve3ysh/ /
```

### Intel/AMD

Execute the following commands:
```
wget https://github.com/pa28/hamclock-systemd/releases/download/v2.59.5/ve3ysh-repo_2.59.5_amd64.deb
sudo apt install -f ./ve3ysh-repo_2.59.5_amd64.deb
```
### Arm 

Execute these commands:
```
wget https://github.com/pa28/hamclock-systemd/releases/download/v2.59.5/ve3ysh-repo_2.59.5_armhf.deb
sudo apt install -f ./ve3ysh-repo_2.59.5_armhf.deb
```
