# Repository

When projects are ready I usually make them available in my repositories:
* One hosted by [GemFury](https://gemfury.com/). 
* One self hosted on a virtual public server [repo.vfrott.ca](https://repo.vfrott.ca).

I have made meta-packages, packages that you can download from git hub and install to set up access to the package repository.

## repo.vfrott.ca

This repository is served by a single meta-package for all architectures, though the available binary packages are still only
for `amd64` (Intel/ARM 64 bit) and `armhf` (Raspberry Pi 32 bit), the only two architectures I currently build for. The repository
is signed, unfortunately the [GemFury](https://gemfury.com/) repo is not.

1. Download and save the the repo package [vfrott-repo](https://github.com/pa28/hamclock-systemd/releases/download/V2.65/vfrott-repo_0.0.1-1_all.deb)
2. Confirm the SHA256 check sum is `9922e105c4593e082233ce2097cfb186adac478f02fae9ad5f1fb6ddcc90429e`, for redundancy this sum is also available from [rep.vfrott.ca](https://repo.vfrott.ca/).
```
sha256sum vfrott-repo_0.0.1-1_all.deb
```
3. Install the repo package:
```
sudo apt install -f ./vfrott-repo_0.0.1-1_all.deb
sudo apt update
```

Available packages are listed on the repository home page [repo.vfrott.ca](https://repo.vfrott.ca/). In this repository some packages
are stored in the <em>expreimental</em> repository. If you wish to install them you can gain access to the <em>exprimental</em>
repository by installing the package `vfrott-repo-experimental`.

## GemFury
There are two meta-packages (because at present I am only creating packages for two platforms:
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
