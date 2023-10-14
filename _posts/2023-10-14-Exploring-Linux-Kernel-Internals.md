
# Exploring Linux Kernel Internals

In this post, let's explore how to compile a simple linux kernel module, insert it into an existing kernel and learn the internals.

## Linux Kernel Module
Linux kernel module is a code chunk which Linux uses to integrate to its running kernel to perform device driver functionality, implement system calls and dding
features to linux kernel. This can be done by invoking module_init() and module_exit().

## Prepare your Linux distribution for kernel module development
For this blog, Ubuntu 22.04 Jammy ditribution is used.
```
#cat /etc/os-release 
PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

For a kernel module to be compiled, linux headers and gcc compiler need to be installed in the Linux system.
```
sudo apt-get install build-essential linux-headers-$(uname -r)
```

## "Hello World" kernel module

This simple kernel module will print "Hello, world". 

```
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>

MODULE_AUTHOR("Karthikeyan Arulmozhivarman");
MODULE_DESCRIPTION("Hello world driver");
MODULE_LICENSE("GPL");

static int __init hello_world_init(void) {
  printk(KERN_INFO "Hello, world\n");
  return 0;
}

static void __exit hello_world_exit(void) {
  printk(KERN_INFO "Goodbye, world\n");
}

module_init(hello_world_init);
module_exit(hello_world_exit);

```

Next, we need to compile this piece of code. For that, we need to create a simple makefile.

```
obj-m += hello-world.o
all:
	 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

Let's do "make". This creates our hello-world.ko kernel module

```
# make
make -C /lib/modules/6.2.0-34-generic/build M=/home/karkey/kernel_module modules
make[1]: Entering directory '/usr/src/linux-headers-6.2.0-34-generic'
warning: the compiler differs from the one used to build the kernel
  The kernel was built by: x86_64-linux-gnu-gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
  You are using:           gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
  CC [M]  /home/karkey/kernel_module/hello-world.o
  MODPOST /home/karkey/kernel_module/Module.symvers
  CC [M]  /home/karkey/kernel_module/hello-world.mod.o
  LD [M]  /home/karkey/kernel_module/hello-world.ko
  BTF [M] /home/karkey/kernel_module/hello-world.ko
Skipping BTF generation for /home/karkey/kernel_module/hello-world.ko due to unavailability of vmlinux
make[1]: Leaving directory '/usr/src/linux-headers-6.2.0-34-generic'
```

Finally, the time has come. Let's insmod this ko.

```
# insmod hello-world.ko
```

We need to check in dmesg to see if the kernel prints are seen.
Yay!!! We got "Hello, world" in dmesg.

```
# dmesg
[10439.147715] hello_world: disagrees about version of symbol module_layout
[10737.555825] hello_world: loading out-of-tree module taints kernel.
[10737.564223] hello_world: module verification failed: signature and/or required key missing - tainting kernel
[10737.583680] Hello, world
```

Hmm!!! There are kernel messages saying module verification failed. If this .kconfig "CONFIG_MODULE_SIG=n" is disabled, these logs are not seen. We need to do "make clean"
and redo make.
```
CONFIG_MODULE_SIG=n
obj-m += hello-world.o
all:
	 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
	 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
