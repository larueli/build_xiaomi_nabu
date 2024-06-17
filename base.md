# Instructions

## Base VM

Ubuntu 22.04 (because https://github.com/akhilnarang/scripts/blob/master/setup/android_build_env.sh doesn't seem to support Ubuntu 24 ?)

16 dedicated CPU (can be less, but very slow : several hours for Evolution X)

64Go RAM (can be swap but slower)

I recommend using a dedicated vm (on cloud), for instance I use Scaleway d `POP2-HC-16C-32G` (Workload-Optimized Instances, dedicated cpu, AMD EPYC™ 7003 Series, 16 cores x 2.8Ghz, 32Go RAM) with 350Go block storage, without IPv4 adress : approx 0.5€/hour. **Do not forget to destroy resource after building to stop billing !** Even with this config, it takes approx 10 minutes to build TWRP or pitch black, but still three hours to build Evolution X.

## IPv6

If using IPv6 only VM (to save money)

```
systemctl stop systemd-resolved
systemctl disable systemd-resolved
```

Then find a DNS64 nameserver (with public NAT64 servers), i use `2001:67c:2b0::4`, based on https://gist.github.com/unixfox/bb299ce4f862fad66ee2e6d9024bef98/ and https://nat64.xyz/

Backup your resolv.conf : `cp /etc/resolv.conf /root/resolv.conf.bak` and edit your `/etc/resolv.conf` to make it look like this (remove anything else) :

```
nameserver 2001:67c:2b0::4
```

## Setup env

```
cd
git clone https://github.com/akhilnarang/scripts
cd scripts
sudo bash setup/android_build_env.sh

git config --global user.email "random@email.local"
git config --global user.name "random name"
```

## Create swap

As root

```
swapoff -a
fallocate -l 32G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

At each reboot, you must use `sudo swapon /swapfile`.

## CCache

```
sudo mkdir /mnt/ccache
mkdir $HOME/.ccache
sudo mount --bind $HOME/.ccache /mnt/ccache
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
export CCACHE_DIR=/mnt/ccache
export USE_CCACHE=1 && export CCACHE_EXEC=/usr/bin/ccache && ccache -M 60G && export CCACHE_DIR=/mnt/ccache
```

At each new shell/ssh session : `export USE_CCACHE=1 && export CCACHE_EXEC=/usr/bin/ccache && ccache -M 60G && export CCACHE_DIR=/mnt/ccache`

## SCP

To fetch builded data I use scp cli, that works like this : `scp user@YOURIP:YOURPATH YOURLOCALPATH`

For instance, on my VM I have `/home/user/pitchblack/out/target/product/nabu/boot.img` and I want to download it to my current computer folder as boot2.img, i use :

`scp user@IP:/home/user/pitchblack/out/target/product/nabu/boot.img boot2.img`

If using IPv6, add `-6` option after scp and encapsulate IPv6 adress in brackets : `scp -6 user@[YOURIP]:YOURPATH YOURLOCALPATH`

## Notes

Do not forget to close your shell/ssh session when moving from one project (twrp, evolution x, ...) to another.

If something goes wrong, go back to your home folder (`cd`) and completely remove your project folder `rm -rf twrp` and start over.
