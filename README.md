# Raspberry Pi 2 - Kernel 4.4 and NFTables

## Install Dependencies

```
apt-get -y install git-core bc build-essential 
```

## Download the Kernel

```
cd ~
git clone --depth=1 -b rpi-4.4.y https://github.com/raspberrypi/linux rpi-kernel
cd rpi-kernel
```
*Notice the branch flag (`-b rpi-4.4.y`)*


## Build the Kernel
Choose one way to build:
(I really advice to cross-compile, ie: from a Debian 8 amd64 virtual machine)
### From the Raspberry (1h30 on a Raspberry Pi 2)
  - For Raspberry Pi 1: (or compute module)
```
KERNEL=kernel
make bcmrpi_defconfig
```
*! At this precise step, please refer to __'Configure the Kernel'__ section !*
```
make zImage modules dtbs
```
  - For Raspberry Pi 2:
```
KERNEL=kernel7
make bcm2709_defconfig
```
*! At this precise step, please refer to __'Configure the Kernel'__ section !*
```
make -j4 zImage modules dtbs
```
  - Common for both:
```
make modules_install
cp arch/arm/boot/dts/*.dtb /boot/
cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
cp arch/arm/boot/dts/overlays/README /boot/overlays/
scripts/mkknlimg arch/arm/boot/zImage /boot/$KERNEL.img
```

### Cross-Compiling (10min on a common Intel i5)
#### Install Toolchain
```
cd ~
git clone https://github.com/raspberrypi/tools
cd rpi-kernel
```

#### Build

*Note for 32bits build environment: remove the `-x64` from CROSS_COMPILE path*

  - For Raspberry Pi 1: (or compute module)
```
KERNEL=kernel
make ARCH=arm CROSS_COMPILE=~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf- bcmrpi_defconfig
```

  - For Raspberry Pi 2:
```
KERNEL=kernel7
make ARCH=arm CROSS_COMPILE=~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf- bcm2709_defconfig
```

- For both:

*! At this precise step, please refer to __'Configure the Kernel'__ section !*
```
make -j4 ARCH=arm CROSS_COMPILE=~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf- zImage modules dtbs
```
*Change the `-j4` accordingly (number of threads, typically the number of CPU you want to use to build)*

#### Deploy
Connect the sdcard to the machine you're building with, and find the two partitions with the command `lsblk`

Typically `sdb1` which is the `/boot` partition (fat32) and `sdb2` which is the root `/` (ext4)

Then:
```
mkdir /mnt/fat32
mkdir /mnt/ext4
mount /dev/sdb1 /mnt/fat32
mount /dev/sdb2 /mnt/ext4
```
```
cd ~/rpi-kernel
make ARCH=arm CROSS_COMPILE=~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf- INSTALL_MOD_PATH=/mnt/ext4 modules_install
```
```
cp /mnt/fat32/$KERNEL.img /mnt/fat32/$KERNEL-backup.img
scripts/mkknlimg arch/arm/boot/zImage /mnt/fat32/$KERNEL.img
cp arch/arm/boot/dts/*.dtb mnt/fat32/
cp arch/arm/boot/dts/overlays/*.dtb* /mnt/fat32/overlays/
cp arch/arm/boot/dts/overlays/README /mnt/fat32/overlays/
umount /mnt/fat32
umount /mnt/ext4
```



## Configure the Kernel

Simply replace the `.config` with the one I'm providing [here](.config), and you're done.

*! Caution: You have to execute this step right before the `make [...] zImage modules dtbs` command !*
