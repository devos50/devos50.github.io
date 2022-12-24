---
layout: post
title: Emulating an iPod Touch 1G and iPhoneOS 1.0 using QEMU (Part II)
date: 2022-12-23
comments: true
tags: QEMU
---

In my [previous blog post]({% post_url 2022-10-09-ipod-touch-qemu %}), I described how I managed to get an iPod Touch 1G up and running using the QEMU emulator. In this follow-up post, I will outline the necessary steps to get the emulator up and running in a local environment.

*Note: the instructions below have only been tested on MacOS so far.*

## Building QEMU

The emulator is built on top of QEMU, which we should build first. Start by cloning my fork of QEMU and checking out to the `ipod_touch_1g` branch:

```
git clone https://github.com/devos50/qemu
cd qemu
git checkout ipod_touch_1g
```

Compile QEMU by running the following commands:

```
mkdir build
cd build
../configure --enable-sdl --disable-cocoa --target-list=arm-softmmu --disable-capstone --disable-pie --disable-slirp --extra-cflags=-I/usr/local/opt/openssl@3/include --extra-ldflags='-L/usr/local/opt/openssl@3/lib -lcrypto'
make
```

Note that we're explicitly enabling compilation of the SDL library which is used for interaction with the emulator (e.g., capturing keyboard and mouse events). Also, we only configure and build the ARM emulator. We're also linking against OpenSSL as the AES/SHA1 engine uses some of the library's cryptographic functions. Remember to update the include/library paths to the OpenSSL library in case they are located elsewhere. You can speed up the `make` command by passing the number of available CPU cores with the `-j` flag, e.g., use `make -j6` to compile using six CPU cores. The compilation process should produce the `qemu-system-arm` binary in the `build/arm-softmmu` directory.

## Downloading System Files

We need a few files to successfully boot the iPod Touch emulator to the home screen, which I published as a GitHub release for convenience. You can download all these files from [here](https://github.com/devos50/qemu/releases/tag/n45ap_v1), and they include the following:
- The S5L8900 bootrom binary, as iBoot and the kernel invokes some procedures in the bootrom logic.
- The iBoot bootloader binary. This file is typically included in the IPSW firmware in an encrypted format, but for convenience, I've extracted the raw binary and included it in the GitHub repository.
- A NOR image that contains various auxillary files used by the bootloader. I will provide some instructions on generating this NOR image manually later in this post.
- A NAND image that contains the root file system. I will provide some instructions on generating this NAND image manually later in this post.

Download all the required files and save them to a convenient location. You should unzip the `nand_n45ap.zip` file, which contains a directory named `nand`.

## Running the Emulator

We are now ready to run the emulator from the `build` directory with the following command:

```
./arm-softmmu/qemu-system-arm -M iPod-Touch,bootrom=<path to bootrom image>,iboot=<path to iboot image>,nand=<path to nand directory> -serial mon:stdio -cpu max -m 1G -d unimp -pflash <path to NOR image>
```

Remember to fix the flags, so they point correctly to the downloaded system files. Running the command above should start the emulator, and you should see some logging output in the console:

```
martijndevos@iMac-van-Martijn build % ./arm-softmmu/qemu-system-arm -M iPod-Touch,bootrom=/Users/martijndevos/Documents/ipod_touch_emulation/bootrom_s5l8900,iboot=/Users/martijndevos/Documents/ipod_touch_emulation/iboot.bin,nand=/Users/martijndevos/Documents/generate_nand/nand -serial mon:stdio -cpu max -m 1G -d unimp -pflash /Users/martijndevos/Documents/generate_nor/nor.bin
WARNING: Image format was not specified for '/Users/martijndevos/Documents/generate_nor/nor.bin' and probing guessed raw.
         Automatically detecting the format is dangerous for raw images, write operations on block 0 will be restricted.
         Specify the 'raw' format explicitly to remove the restrictions.
Reading PMU register 118
Reading PMU register 75
Reading PMU register 87
Reading PMU register 103
iis_init()
spi_init()
Reading PMU register 75
power supply type batt
battery voltage Reading PMU register 87
error
SysCfg: version 0x00010001 with 4 entries using 200 of 8192 bytes
BDEV: protecting 0x2000-0x8000
image 0x1802bd20: bdev 0x1802b6a8 type dtre offset 0x10800 len 0x7d28
image 0x1802c170: bdev 0x1802b6a8 type batC offset 0x18d40 len 0x101e1
image 0x1802c5c0: bdev 0x1802b6a8 type logo offset 0x29a80 len 0x1c3a
image 0x1802ca10: bdev 0x1802b6a8 type nsrv offset 0x2bfc0 len 0x4695
image 0x1802ce60: bdev 0x1802b6a8 type batl offset 0x30d00 len 0xc829
image 0x1802d2b0: bdev 0x1802b6a8 type batL offset 0x3e240 len 0xe9d2
image 0x1802e888: bdev 0x1802b6a8 type recm offset 0x4d780 len 0xb594
display_init: displayEnabled: 0
otf clock divisor 5
fps set to: 59.977
SFN: 0x600, Addr: 0xfe00000, Size: 0x14001e0, hspan: 0x500, QLEN: 0x140
merlot_init() -- Universal code version 08-29-07
Merlot Panel ID (0x71c200):
   Build:          PVT1 
   Type:           TMD 
   Project/Driver: M68/NSC-Merlot 
ClcdInstallGammaTable: No Gamma table found for display_id: 0x0071c200
Reading PMU register 75
power supply type batt
battery voltage Reading PMU register 87
error
Reading PMU register 23
Reading PMU register 42
Reading PMU register 40
Reading PMU register 41
Reading PMU register 75
power supply type batt
battery voltage Reading PMU register 87
error
Reading PMU register 23
Reading PMU register 42
Reading PMU register 40
Reading PMU register 41
usb_menu_init()
vrom_late_init: unknown image crc: 0x66a3fbbf


=======================================
::
:: iBoot, Copyright 2007, Apple Inc.
::
::	BUILD_TAG: iBoot-204
::
::	BUILD_STYLE: RELEASE
::
=======================================

Reading PMU register 87
[FTL:MSG] Apple NAND Driver (AND) 0x43303032
...
```

If there are any issues running the above commands, please let me know by commenting on this post or by [making an issue](https://github.com/devos50/qemu/issues/new) on GitHub.

## Manually Generating the NOR Image

If you wish to make changes to the NOR image, e.g., to replace the boot logo, to modify the device tree, or to change the kernel bootargs, you can follow the instructions below. I created [a separate tool](https://github.com/devos50/generate-ipod-touch-1g-nor) tool to generate the NOR image. You can clone and compile this tool with the following command:

```
git clone https://github.com/devos50/generate-ipod-touch-1g-nor
cd generate-ipod-touch-1g-nor
gcc generate_nor.c aes.c -o generate_nor -I/usr/local/Cellar/openssl@1.1/1.1.1l/include -L/usr/local/Cellar/openssl@1.1/1.1.1l/lib -lssl -lcrypto
```

Remember to replace the include and library paths to point to your OpenSSL installation.

You can modify the specifics of the generated NOR file by changing the `generate_nor.c` file. For example, the kernel bootargs can be modified [here](https://github.com/devos50/generate-ipod-touch-1g-nor/blob/main/generate_nor.c#L333). The `data` directory contains various IMG2 images that will be embedded in the NOR image, including the device tree and boot logo. Generating the NOR image can be done by running the `generate_nor` binary:

```
./generate_nor
```

This will generate a `nor.bin` file.

## Manually Generating the NAND Image

The NAND image is generated based on the root filesystem included in the IPSW firmware file, albeit heavily modified to bypass various checks. The necessary code can be found in [this repository](https://github.com/devos50/generate-ipod-touch-1g-nand) and be cloned as follows:

```
git clone https://github.com/devos50/generate-ipod-touch-1g-nand
cd generate-ipod-touch-1g-nand
gcc generate_nand.c -o generate_nand
```

This produces the `generate_nand` binary in the root directory of the repository. It expects `filesystem-readonly.img` file, which can be generated using the instructions below.

### Creating a Filesystem Image

As a starting point, I uploaded [a writable root filesystem](https://github.com/devos50/generate-ipod-touch-1g-nand/releases/tag/it1g_nand_filesystem) to GitHub. This DMG file is based on the N45AP firmware and contains various modifications for emulation purposes. On Mac, you can mount this file and make changes to it. When done, unmount the file and convert the writable DMG to a read-only one using the `hdiutil` tool (available on Mac):

```
hdiutil convert -format UDRO -o filesystem-readonly.dmg  filesystem-writable.dmg
```

You should then extract the filesystem partition from the DMG file. For this, I used the `dmg2img` tool, which can be downloaded from [here](https://github.com/Lekensteyn/dmg2img) (build instructions can be found in this repository too). You can see which partitions the DMG file includes using the following command:

```
./dmg2img -l filesystem-readonly.dmg 
```

Which outputs something like:

```
dmg2img v1.6.5 (c) vu1tur (to@vu1tur.eu.org)

filesystem-readonly.dmg --> (partition list)

partition 0: Driver Descriptor Map (DDM: 0)
partition 1: Apple (Apple_partition_map: 1)
partition 2: Macintosh (Apple_Driver_ATAPI: 2)
partition 3: Mac_OS_X (Apple_HFSX: 3)
partition 4:  (Apple_Free: 4)
```

We need to extract the partition in HFS format, i.e., partition 3. Extract this partition using the following command:

```
./dmg2img -p 3 filesystem-readonly.dmg
```

This generates a `filesystem-readonly.img`, which you should copy to the directory containing the `generate_nand` binary. As the final step, generate the `nand` directory as follows:

```
./generate_nand
```