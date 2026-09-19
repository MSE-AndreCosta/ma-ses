# Initial Setup

DISCLAIMER: As I don't particularly like VSCode, I've decided to not use the devcontainer instructions and instead install and run everything natively on Fedora 44. 

## Part 1 : Install the development environment for the target system 

I've decide to setup my work environment with a main repository where I'll be able to store the lab journal alongside the lab resources and buildroot.
The lab resources and buildroot are two submodules on the root of the repository

```bash
mkdir ma-ses
cd ma-ses
git init .
git submodule add https://github.com/MA-SeS/resources.git
git submodule add https://github.com/buildroot/buildroot.git
git -C buildroot checkout -b local-lts 2025.02.17
# submodule add automatically adds .gitmodules and the submodule folder to staging,
# we just need to add the buildroot folder after the checkout
git add buildroot
git commit -m "chore: setup buildroot and lab resources as git submodules"
```

The repository is available [here](https://github.com/MSE-AndreCosta/ma-ses). The remote was setup using the following commands:

```bash
git remote add origin git@github.com:MSE-AndreCosta/ma-ses.git
git branch -M main
git push -u origin main
```


## Part 2 : Generate a vanilla SD card image, then a custom one

*Go to the buildroot directory and start by listing all the target configurations provided by Buildroot.*

```bash
cd buildroot
make list-defconfigs | grep raspberrypi4
  raspberrypi4_64_defconfig           - Build for raspberrypi4_64
  raspberrypi4_defconfig              - Build for raspberrypi4
```

*The one that interests us is the 64-bits version, therefore make sure to load it.*

```bash
make defconfig raspberrypi4_64_defconfig
```

*Next, run the menuconfig target and browse through the various menus to check that the configuration you loaded indeed seems to be the one for the Raspberry Pi 4.*

```bash
make menuconfig
```
*What configuration option(s) gave you a strong confidence that it is indeed the right configuration for your target?*

The "Target options" menu makes it very clear that the configuration targets raspberry pi 4:

![target_options_rpi4_64_defconfig.png](media/target_options_rpi4_64_defconfig.png)

*Start the compilation of the SD card image for the target system*

```bash
make
```

> [!WARNING]
> The lab instructions say *You can use the -j argument with make to spawn several compilation processes in order to take advantage of multi-core CPU architectures found in modern computers.*
> This is not recommended by buildroot, see [the buildroot manual §8.13. Top-level parallel build](https://buildroot.org/downloads/manual/manual.html#top-level-parallel-build).
> Note that `make` already uses the full CPU cores during the build stage of each package.

*Inspect and investigate the purpose of the files located in output/images. Beside sdcard.img, can you identify and understand the purpose of each file in this directory?*

```bash
# difference device tree blobs for the different board revisions of the raspberry pi 4
-rwxr-xr-x. 1 andre andre  55K Sep 19 18:40 bcm2711-rpi-400.dtb
-rwxr-xr-x. 1 andre andre  55K Sep 19 18:40 bcm2711-rpi-4-b.dtb
-rwxr-xr-x. 1 andre andre  55K Sep 19 18:40 bcm2711-rpi-cm4.dtb
-rwxr-xr-x. 1 andre andre  52K Sep 19 18:40 bcm2711-rpi-cm4s.dtb
# boot partition that will be loaded by the bootloader
-rw-r--r--. 1 andre andre  32M Sep 19 18:40 boot.vfat
# configuration file for the tool that builds the sdcard.img
-rw-r--r--. 1 andre andre  522 Sep 19 18:40 genimage.cfg
# kernel image
-rw-r--r--. 1 andre andre  22M Sep 19 18:40 Image
# user-space rootfs
-rw-r--r--. 1 andre andre 120M Sep 19 18:40 rootfs.ext2
lrwxrwxrwx. 1 andre andre   11 Sep 19 18:40 rootfs.ext4 -> rootfs.ext2
# raspberry pi firmware blob
drwxr-xr-x. 1 andre andre   98 Sep 19 18:29 rpi-firmware
# full image with the firmware, boot partition, rootfs partition and the kernel image
-rw-r--r--. 1 andre andre 153M Sep 19 18:40 sdcard.img
```
