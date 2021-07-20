# Availability

Boards:
 - 

# Create SO for Udoo Arm Dual/Quad

## Basic Requirement

   *  Running a recent supported release of Debian, Fedora or Ubuntu on a x86 64bit based PC; without OS Virtualization Software.
   *  Many of the listed commands assume /bin/bash as the default shell.
   *  ARM Cross Compiler 
      *  [Linaro](https://www.linaro.org)
      *  [Linaro Toolchain Binaries](https://www.linaro.org/downloads/)
   *  Bootloader
      *  [Das U-Boot – the Universal Boot Loader](http://www.denx.de/wiki/U-Boot)
      *  [Source](https://github.com/u-boot/u-boot/)
   *  Linux Kernel
      *  [Linus’s Mainline tree](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git)
   *  ARM based rootfs
      *  [Debian](https://www.debian.org)
      *  [Ubuntu](https://www.ubuntu.com)

## ARM Cross Compiler: GCC

This is a pre-built (64bit) version of GCC that runs on generic linux, sorry (32bit) x86 users, it’s time to upgrade…

- Download/Extract:

```bash

wget -c https://releases.linaro.org/components/toolchain/binaries/6.5-2018.12/arm-linux-gnueabihf/gcc-linaro-6.5.0-2018.12-x86_64_arm-linux-gnueabihf.tar.xz
tar xf gcc-linaro-6.5.0-2018.12-x86_64_arm-linux-gnueabihf.tar.xz
export CC=`pwd`/gcc-linaro-6.5.0-2018.12-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-

```

- Test Cross Compiler:

```bash

${CC}gcc --version

```
Test output :
```

arm-linux-gnueabihf-gcc (Linaro GCC 6.5-2018.12) 6.5.0
Copyright (C) 2017 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

## Bootloader: U-Boot

Das U-Boot – the Universal Boot Loader: http://www.denx.de/wiki/U-Boot 1
- Download:

```bash

git clone -b v2019.04 https://github.com/u-boot/u-boot --depth=1
cd u-boot/

```

- Configure and Build:

```bash

make ARCH=arm CROSS_COMPILE=${CC} distclean
make ARCH=arm CROSS_COMPILE=${CC} udoo_defconfig
make ARCH=arm CROSS_COMPILE=${CC}
```

## Linux Kernel

This script will build the kernel, modules, device tree binaries and copy them to the deploy directory.

- Download:

```bash

git clone https://github.com/RobertCNelson/armv7-multiplatform
cd armv7-multiplatform/
```

  * For v4.14.x (Longterm 4.14.x):

  ```bash
  git checkout origin/v4.14.x -b tmp
  ```
  
  * For v4.14.x-rt (Longterm 4.14.x + Real-Time Linux):

  ```bash
  git checkout origin/v4.14.x-rt -b tmp
  ```

  * For v4.19.x (Longterm 4.19.x):

  ```bash
  git checkout origin/v4.19.x -b tmp
  ```

  * For v4.19.x-rt (Longterm 4.19.x + Real-Time Linux):

  ```bash
  git checkout origin/v4.19.x-rt -b tmp
  ```

  * For v5.0.x (Stable):

  ```bash
  git checkout origin/v5.0.x -b tmp
  ```
 
  * For v5.0.x-rt (Stable + Real-Time Linux):

  ```bash
  git checkout origin/v5.0.x-rt -b tmp
  ```

- Build:

```bash
./build_kernel.sh
```

Root File System
Debian 10
User 	Password
debian 	temppwd
root 	root

Download:

#user@localhost:~$
wget -c https://rcn-ee.com/rootfs/eewiki/minfs/debian-10.9-minimal-armhf-2021-04-14.tar.xz

Verify:

#user@localhost:~$
sha256sum debian-10.9-minimal-armhf-2021-04-14.tar.xz

#sha256sum output:
268ae963a067f13578858a9203ec6a0668d26c707c2fd5e42d0ad3ead5dc9289  debian-10.9-minimal-armhf-2021-04-14.tar.xz

Extract:

#user@localhost:~$
tar xf debian-10.9-minimal-armhf-2021-04-14.tar.xz

Ubuntu 18.04 LTS
User 	Password
ubuntu 	temppwd

Download:

#user@localhost:~$
wget -c https://rcn-ee.com/rootfs/eewiki/minfs/ubuntu-20.04.2-minimal-armhf-2021-04-14.tar.xz

Verify:

#user@localhost:~$
sha256sum ubuntu-20.04.2-minimal-armhf-2021-04-14.tar.xz

#sha256sum output:
46702f198730d6edeace72f6d3f9c41b7e097c78c17cb202cc0beee551021783  ubuntu-20.04.2-minimal-armhf-2021-04-14.tar.xz

Extract:

#user@localhost:~$
tar xf ubuntu-20.04.2-minimal-armhf-2021-04-14.tar.xz

Setup microSD card

We need to access the External Drive to be utilized by the target device. Run lsblk to help figure out what linux device has been reserved for your External Drive.

#Example: for DISK=/dev/sdX
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465.8G  0 disk
├─sda1   8:1    0   512M  0 part /boot/efi
└─sda2   8:2    0 465.3G  0 part /                <- Development Machine Root Partition
sdb      8:16   1   962M  0 disk                  <- microSD/USB Storage Device
└─sdb1   8:17   1   961M  0 part                  <- microSD/USB Storage Partition

#Thus you would use:
export DISK=/dev/sdb

#Example: for DISK=/dev/mmcblkX
lsblk
NAME      MAJ:MIN   RM   SIZE RO TYPE MOUNTPOINT
sda         8:0      0 465.8G  0 disk
├─sda1      8:1      0   512M  0 part /boot/efi
└─sda2      8:2      0 465.3G  0 part /                <- Development Machine Root Partition
mmcblk0     179:0    0   962M  0 disk                  <- microSD/USB Storage Device
└─mmcblk0p1 179:1    0   961M  0 part                  <- microSD/USB Storage Partition

#Thus you would use:
export DISK=/dev/mmcblk0

Erase partition table/labels on microSD card:

sudo dd if=/dev/zero of=${DISK} bs=1M count=10

Install Bootloader:

#user@localhost:~$
sudo dd if=./u-boot/SPL of=${DISK} seek=1 bs=1k
sudo dd if=./u-boot/u-boot.img of=${DISK} seek=69 bs=1k

Create Partition Layout:
With util-linux v2.26, sfdisk was rewritten and is now based on libfdisk.

#Check the version of sfdisk installed on your pc
sudo sfdisk --version

#Example Output
sfdisk from util-linux 2.27.1

#sfdisk >= 2.26.x
sudo sfdisk ${DISK} <<-__EOF__
1M,,L,*
__EOF__

#sfdisk <= 2.25.x
sudo sfdisk --unit M ${DISK} <<-__EOF__
1,,L,*
__EOF__

Format Partition:

for: DISK=/dev/mmcblkX
sudo mkfs.ext4 -L rootfs ${DISK}p1
 
for: DISK=/dev/sdX
sudo mkfs.ext4 -L rootfs ${DISK}1

Mount Partition:
On most systems these partitions may be auto-mounted…

sudo mkdir -p /media/rootfs/
 
for: DISK=/dev/mmcblkX
sudo mount ${DISK}p1 /media/rootfs/
 
for: DISK=/dev/sdX
sudo mount ${DISK}1 /media/rootfs/

Install Kernel and Root File System

To help new users, since the kernel version can change on a daily basis. The kernel building scripts listed on this page will now give you a hint of what kernel version was built.

-----------------------------
Script Complete
eewiki.net: [user@localhost:~$ export kernel_version=5.X.Y-Z]
-----------------------------

Copy and paste that “export kernel_version=5.X.Y-Z” exactly as shown in your own build/desktop environment and hit enter to create an environment variable to be used later.

export kernel_version=5.X.Y-Z

Copy Root File System

#Debian; Root File System: user@localhost:~$
sudo tar xfvp ./debian-*-*-armhf-*/armhf-rootfs-*.tar -C /media/rootfs/
sync
sudo chown root:root /media/rootfs/
sudo chmod 755 /media/rootfs/

#Ubuntu; Root File System: user@localhost:~$
sudo tar xfvp ./ubuntu-*-*-armhf-*/armhf-rootfs-*.tar -C /media/rootfs/
sync
sudo chown root:root /media/rootfs/
sudo chmod 755 /media/rootfs/

Setup extlinux.conf

#user@localhost:~$
sudo mkdir -p /media/rootfs/boot/extlinux/
sudo sh -c "echo 'label Linux ${kernel_version}' > /media/rootfs/boot/extlinux/extlinux.conf"
sudo sh -c "echo '    kernel /boot/vmlinuz-${kernel_version}' >> /media/rootfs/boot/extlinux/extlinux.conf"
sudo sh -c "echo '    append root=/dev/mmcblk0p1 ro rootfstype=ext4 rootwait quiet' >> /media/rootfs/boot/extlinux/extlinux.conf"
sudo sh -c "echo '    fdtdir /boot/dtbs/${kernel_version}/' >> /media/rootfs/boot/extlinux/extlinux.conf"

Copy Kernel Image

Kernel Image:

#user@localhost:~$
sudo cp -v ./armv7-multiplatform/deploy/${kernel_version}.zImage /media/rootfs/boot/vmlinuz-${kernel_version}

Copy Kernel Device Tree Binaries

#user@localhost:~$
sudo mkdir -p /media/rootfs/boot/dtbs/${kernel_version}/
sudo tar xfv ./armv7-multiplatform/deploy/${kernel_version}-dtbs.tar.gz -C /media/rootfs/boot/dtbs/${kernel_version}/

Copy Kernel Modules

#user@localhost:~$
sudo tar xfv ./armv7-multiplatform/deploy/${kernel_version}-modules.tar.gz -C /media/rootfs/

File Systems Table (/etc/fstab)

#user@localhost:~/$
sudo sh -c "echo '/dev/mmcblk0p1  /  auto  errors=remount-ro  0  1' >> /media/rootfs/etc/fstab"

Remove microSD/SD card

sync
sudo umount /media/rootfs

2D/3D Video Acceleration via Etnaviv Project

Vivante GC320/GC880 (quad GC320/GC355/GC2000) 2D/3D Acceleration via [Etnaviv|https://github.com/etnaviv]
This sections assumes you have already installed your favorite xorg based window manager, such as lxde, lxqt, xfce, kde, gnome, etc… These are packages that need to be installed on top of your selected windows manager and an xorg.conf needed to correctly setup the video interface.
Verify your kernel has etnaviv support:

#DUAL
debian@arm:~$ dmesg | grep etnaviv
[    4.047928] etnaviv gpu-subsystem: bound 134000.gpu (ops gpu_ops)
[    4.047955] etnaviv gpu-subsystem: bound 130000.gpu (ops gpu_ops)
[    4.047968] etnaviv-gpu 134000.gpu: model: GC320, revision: 5007
[    4.090180] etnaviv-gpu 130000.gpu: model: GC880, revision: 5106

#QUAD
debian@arm:~$ dmesg | grep etnaviv
[    4.025841] etnaviv gpu-subsystem: bound 134000.gpu (ops gpu_ops)
[    4.025868] etnaviv gpu-subsystem: bound 130000.gpu (ops gpu_ops)
[    4.025887] etnaviv gpu-subsystem: bound 2204000.gpu (ops gpu_ops)
[    4.025899] etnaviv-gpu 134000.gpu: model: GC320, revision: 5007
[    4.070139] etnaviv-gpu 130000.gpu: model: GC2000, revision: 5108
[    4.120479] etnaviv-gpu 2204000.gpu: model: GC355, revision: 1215
[    4.120492] etnaviv-gpu 2204000.gpu: Ignoring GPU with VG and FE2.0

sudo apt-get update
sudo apt-get install xserver-xorg-video-armada-etnaviv

#/etc/X11/xorg.conf
Section "Monitor"
        Identifier      "Builtin Default Monitor"
EndSection
Section "Device"
        Identifier      "Builtin Default fbdev Device 0"
        Driver          "armada"
EndSection
Section "Screen"
        Identifier      "Builtin Default fbdev Screen 0"
        Device          "Builtin Default fbdev Device 0"
        Monitor         "Builtin Default Monitor"
EndSection
Section "ServerLayout"
        Identifier      "Builtin Default Layout"
        Screen          "Builtin Default fbdev Screen 0"
EndSection
