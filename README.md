# README

I'm trying to find a way to kisslinux: this is still again a work in progress...

* Created 12/2/2022 forking [Medirim/dotfiles](https://github.com/Mederim/dotfiles)
* Update 13/2/2022 following [K1SS - KISS Linux version 2021.7-9 UEFI Installation - Kernel 5.14.8
](https://www.youtube.com/watch?v=QCjjFqC-Ve8&t=0s)
* Started to follow [Mederim/kiss-waydroid](https://github.com/Mederim/kiss-waydroid)

# Foreword

I need just to build a little system, most little is better. 
No need X, Wayland nothing exept console.

I want try to see if will be possible from a minimal installation of kisslinux, get this installation reproducible using my tool: penguins-eggs.

Unfortunately, after 3 days I'm again on the point to be uncapable to build a minimun, possibly generic kernel for a KVM machine.

# Before you start
After many tempts I choose to follow the second video where 

## Download archiso
You can downloads last [Arch Linux install](https://archlinux.org/download/) and follow the instructions.

```
pacman -Sy
pacman -S wget
systemctl start sshd
passwd
ip a
```

Open a terminal and connect:

```
ssh root@192.168.61.101
fdisk /dev/sda
```
![fdisk](./images/fdisk.png)

It's very import set the type of partitions with the command t, You must get something like this:
```
Device       Start      End  Sectors  Size Type
/dev/sda1     2048  1050623  1048576  512M EFI System
/dev/sda2  1050624  9439231  8388608    4G Linux swap
/dev/sda3  9439232 67108830 57669599 27.5G Linux root (x86-64)
```
At this point write the changemennt on the disk with the command 'w'.

## Formatting disk

```
mkfs.fat -F32 /dev/sda1 
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
```

## Mounting 
```
mount /dev/sda3 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```


# [001] Install KISS

```                                                                            |
ver=2021.7-9
url=https://github.com/kisslinux/repo/releases/download/$ver
file=kiss-chroot-$ver.tar.xz
```

# [002] Download The Installation Tarball
```
curl -fLO "$url/$file"
```

# [003] Verify Checksums
```
curl -fLO "$url/$file.sha256"   
sha256sum -c < "$file.sha256"
```
# [004] Verify Signature

```
curl -fLO "$url/$file.asc"
gpg --keyserver keyserver.ubuntu.com --recv-key 13295DAC2CF13B5C
gpg --verify "$file.asc"                                               
```

# [005] Unpack The Tarball
```
cd /mnt 
tar xvf "$OLDPWD/$file"
```

# [006] Enter The Chroot
```
/mnt/bin/kiss-chroot /mnt
```

Fstab add these lines
```
/dev/sda1     /boot         vfat        defaults,noatime     0 2
/dev/sda2     none          swap        sw                   0 0
/dev/sda3     /             ext4        noatime              0 1
```

Making our profile
```
vi ~/.profile
```
And copy and past inside:

```
export CFLAGS="-O3 -pipe -march=native"
export CXXFLAGS="$CFLAGS"
export MAKEFLAGS="-j4"
#export XDG_RUNTIME_DIR=/tmp
#export MOZ_WAYLAND_DRM_DEVICE=/dev/dri/renderD128
export KISS_PATH=''
KISS_PATH=$KISS_PATH:/var/repo/core
KISS_PATH=$KISS_PATH:/var/repo/extra
KISS_PATH=$KISS_PATH:/var/repo/wayland
KISS_PATH=$KISS_PATH:/var/community/community
#if [ -z $DISPLAY ] && [ "$(tty)" = "/dev/tty1" ]; then
#	  exec sway
#fi
```

And exec .profile with
```
. ~/.profile
```


# [007] Setup Official Repositories

```
git clone https://github.com/kisslinux/repo
sudo mv repo /var/
```

# [007a] Setup community repositury
```
git clone https://github.com/kiss-community/community
sudo mv community /var

```
# [009] Set KISS_PATH
We did it previusly.

```KISS_PATH=:/var/repo/core:/var/repo/extra:/var/repo/wayland:/var/community/community```

# [010] Enable Signature Verification
# [011] Build And Install GPG 
```
kiss build gnupg1
```

# [012] Import (Dylan Araps') Key 
```
gpg --keyserver keyserver.ubuntu.com --recv-key 13295DAC2CF13B5C
echo trusted-key 0x13295DAC2CF13B5C >>/root/.gnupg/gpg.conf
```

# [013] Enable Signature Verification
```
cd /var/repo
git config merge.verifySignatures true  
```

# [014] Rebuild The System
# [015] Modify Compiler Flags (Optional)
We did it previusly.

```
CFLAGS=-O3 -pipe -march=native
CXXFLAGS=-O3 -pipe -march=native
MAKEFLAGS=-j4
```

# [016] Update All Base Packages To The Latest Version
```
kiss update
```

kiss update the first update the package manager, the second it's necessary to build all the remain.

```
kiss update
```


# [017] Rebuild All Packages
This operation can be done also after the installation,

```
cd /var/db/kiss/installed && kiss build *
```

We get the error:
```
pigz-2.6 not founf
```
to correct it, from host console:
```
cd /mnt/var/repo/core/pigz
nano version
```
change the version 2.6 2 to 2.7 2, then:

from chroot 

```
kiss c pigz
```
then, again try to build:

```
cd /var/db/kiss/installed && kiss build *
```

# [018] The Kernel
________________________________________________________________________________

This step involves configuring and building your own Linux kernel. The Linux
kernel is not managed by the distribution and is instead the user's sole
responsibility.

Use kernel from kernel.org

from chrooted
```
cd /var/db/kiss
curl -fLO https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.16.10.tar.xz
tar xvf linux-5.16.4.tar.xz
cd linux-5.16.4

```
## [020] Download Firmware Blobs (If Required)
**no need**

## [021] Configure The Kernel

``` 
make defconfig
```

## [022] Install Required Packages

## [023] perl
```
kiss b perl
```
## [024] libelf
```
kiss b libelf
```

Install Python to prevent kernel error

```
kiss b python
```

## [025] Build The Kernel 

```
make defconfig

```

### localyesconfig

See https://youtu.be/QCjjFqC-Ve8?t=1869

Enable all the options in kernel to =y and not =m kiss only use in kernel build enable all

```
make localyesconfig
```

Enable configs in menu

```
make menuconfig
```

When finish the result of localyesconfig will be

```
using config: '.config'
```
And dont forget to enable  Framebuffer to prevent blackscreen on boot

```
FB=y and FRAMEBUFFER_CONSOLE=y in your .config
```

Once finished do the patch

**IMPORTANT** : Apply this patch

```
sed '/<stdlib.h>/a #include <linux/stddef.h>' \
tools/objtool/arch/x86/decode.c > _

mv -f _ tools/objtool/arch/x86/decode.c  

```

And now compile

```
make
```

This is for modules

```
make INSTALL_MOD_STRIP=1 modules_install   
```

# [028] Install Kernel Image 

```
make install
mv /boot/vmlinuz    /boot/vmlinuz-5.16.10
mv /boot/System.map /boot/System.map-5.16.10
```


## [028] Installing some others things

```
kiss b e2fsprogs
kiss b dosfstools
```
device manager 
**OPTIONAL SINCE KISS USE MDEV**

```
kiss b util-linux
kiss b eudev 
```

dhcp
```
kiss b dhcpcd
mkdir -p /etc/rc.d 
echo "dhcpcd 2> /dev/null" > /etc/rc.d/dhcpcd.boot
```
hostname
```
echo "kisslinux" > /etc/hostname
```

# [029] Install Required Packages

[030] Install init Scripts

```
kiss b baseinit
```

## [031] The Bootloader
UEFI

```
kiss b grub efibootmgr
```
Install grub for UEFI
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB 
grub-mkconfig -o /boot/grub/grub.cfg 
```

## User and Root

Change root passwd

```
passwd root
```
Add your user

```
adduser USERNAME
```

## Finish
 
Exit kiss

```
exit
```

And now reboot

```
reboot
```
