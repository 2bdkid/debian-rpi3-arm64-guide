# debian-rpi3-arm64-guide
This is a guide for building an arm64 Debian distribution for the Raspberry Pi model B.
In principle this guide will also work for the model B+, and model 4 with some tweaking.
I do not have a model 4 to play with, but look out for future updates to this guide.
Development of this guide was done on Fedora Linux x86_64, however any distribution will
work.

---

# Overview
1. Installing prerequisites
2. Format SD card
3. Install base system root filesystem
4. Build and install the Linux kernel
5. Install firmware
6. Edit some configuration files

---

## Installing prerequisites

These tools are needed to cross-compile the Linux kernel for arm64 and build the base
Debian system.

```
sudo dnf group install "C Development Tools and Libraries" "Development Tools"
sudo dnf install openssl-devel qemu-user-static gcc-aarch64-linux-gnu binutils-aarch64-linux-gnu debootstrap
```

## Format SD card

The SD card needs a FAT partition for `/boot` to reside on and the remaining space will be used for `/`. Replace `/dev/sdb` with
the path that corresponds to your SD card.

```
sudo fdisk /dev/sdb
```

Create a 200-300MiB primary partition 1 with partition type `c`. Then create a primary partition 2 with the remaining space. 
It can keep the default partition type `83`.

Verify your partition table looks similar to this and write the table.

```
Device     Boot  Start      End  Sectors  Size Id Type
/dev/sdb1         2048   411647   409600  200M  c W95 FAT32 (LBA)
/dev/sdb2       411648 60751871 60340224 28.8G 83 Linux
```

Create a FAT32 filesystem on partition 1 and an ext4 filesystem on partition 2.

```
sudo mkfs.vfat -F32 /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
```

We will use `/mnt` to mount the SD card.

```
sudo mount /dev/sdb2 /mnt
sudo mkdir /mnt/boot
sudo mount /dev/sdb1 /mnt/boot
```

## Install base system root filesystem

We will use debootstrap to download the base Debian system and finish the package configuration inside a chroot.
debootstrap will download just enough packages to let the system boot, but if you know which additional packages you 
will need then use the `--include` flag to add them here. For example, you may want to go ahead and install dbus. 
You can also install packages from inside a chroot later.

```
sudo debootstrap --arch=arm64 --foreign stable /mnt http://deb.debian.org/debian
sudo cp ${which qemu-aarch64-static) /mnt/usr/bin
sudo chroot /mnt /debootstrap/debootstrap --second-stage
```

If you wish to install additional packages from within a chroot, then

```
sudo chroot /mnt /bin/bash
```

Finally

```
sudo rm /mnt/usr/bin/qemu-aarch64-static
```

## Build and install the Linux kernel

You may need to update which branch is currently being worked on. Just check the branches tab on Github.

```
git clone --depth=1 -b rpi-5.6.y https://github.com/raspberrypi/linux
mkdir linux-build
cd linux
make O=../linux-build ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcmrpi3_defconfig
make O=../linux-build ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules Image
sudo make O=../linux-build ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_MOD_PATH=/mnt modules_install
sudo make O=../linux-build ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- INSTALL_DTBS_PATH=/mnt/boot dtbs_install
sudo mv /mnt/boot/broadcom/* /mnt/boot # rpi documentation says .dtb files should alongside start.elf, i.e. in /boot
sudo rmdir /mnt/boot/broadcom
sudo cp ../linux-build/arch/arm64/boot/Image /mnt/boot/kernel8.img
```

## Install firmware

# WIP MORE TO COME SOON
