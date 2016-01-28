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
make zImage modules dtbs
```
  - For Raspberry Pi 2:
```
KERNEL=kernel7
make bcm2709_defconfig
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
  - For Raspberry Pi 1: (or compute module)
```
make ARCH=arm CROSS_COMPILE=~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/arm-linux-gnueabihf- bcmrpi_defconfig
```
*Remove the `-x64` from the path, if you're compiling from a 32bits environment*
  - For Raspberry Pi 2:
```
make ARCH=arm CROSS_COMPILE=~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/arm-linux-gnueabihf- bcm2709_defconfig
```
*Remove the `-x64` from the path, if you're compiling from a 32bits environment*

