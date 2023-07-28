---
layout: post
title: Build a tiny linux kernel from source code
date: 2023-07-18 00:40 +0530
---


# Build a Tiny Linux kernel from source code

Lets see how to compile a very tiny linux kernel from source code and run the minimal kernel image using QEMU. 

Download any stable linux kernel from [linux kernel git source](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/refs/tags?h=master) 

```
$ wget https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/snapshot/linux-5.19.tar.gz 
$ tar xvf linux-5.19.tar.gz
$ cd linux-5.19
```

Make sure, the ubuntu environment or any distribution that is used to compile the above source code has the following dependencies installed
```
$ sudo apt-get install flex bison libncurses-dev
```

We need to do kernel configuration using "make" in two stages.

First stage is to get tiny configuration. 
  
```
$  make tinyconfig
```

Second stage is to get useful configuration to get the kernel running and end up in shell prompt. For this, the following kernel configurations are required.
 

Next, compile the kernel. Specifying -j option along with "make" creates as many parallel jobs as possible leveraging each available processor core.
```
$ make -j4
  ..
  HOSTCC  arch/x86/boot/tools/build
  XZKERN  arch/x86/boot/compressed/vmlinux.bin.xz
  MKPIGGY arch/x86/boot/compressed/piggy.S
  AS      arch/x86/boot/compressed/piggy.o
  LD      arch/x86/boot/compressed/vmlinux
  ZOFFSET arch/x86/boot/zoffset.h
  OBJCOPY arch/x86/boot/vmlinux.bin
  AS      arch/x86/boot/header.o
  LD      arch/x86/boot/setup.elf
  OBJCOPY arch/x86/boot/setup.bin
  BUILD   arch/x86/boot/bzImage
Kernel: arch/x86/boot/bzImage is ready  (#1)
```

Before test launching the kernel, we need initramfs to land us in a shell prompt once the kernel boots up.

Lets create a small filesystem using busybox which supports only "sh" command

```
$ mkdir -p root  
$ cd root  
$ mkdir -p bin
$ curl -L 'https://www.busybox.net/downloads/binaries/1.35.0-x86_64-linux-musl/busybox' > bin/busybox
$ cd bin
$ ln -s busybox sh
$ cd .. 
$ find . | cpio -ov --format=newc | gzip -9 >../initramfs-mini
```

Launch the kernel along with initramfs with qemu:
```
$  qemu-system-x86_64 -kernel arch/x86/boot/bzImage -initrd initramfs-mini -append "init=/bin/sh"
```

