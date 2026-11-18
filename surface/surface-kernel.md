## linux-surface kernel

https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup#Arch

```
curl -s https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc \
| sudo pacman-key --add -

sudo pacman-key --finger 56C464BAAC421453
sudo pacman-key --lsign-key 56C464BAAC421453

sudo -e /etc/pacman.conf
[linux-surface]
Server = https://pkg.surfacelinux.com/arch/

sudo pacman -Syu

sudo pacman -S linux-surface linux-surface-headers iptsd
paru -S surface-control
```
