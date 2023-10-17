
# A Simple Linux Character Device Driver 

## Everything in Linux is a file

Based on the concept "Everything in Linux is a file", all physical devices like hard disk, usb pen drive, NIC card, mouse and keyboard are represented in Linux 
as files. Let's quickly check the devices in the system using /proc/devices to check char devices and block devices.

```
# cat /proc/devices 
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  5 ttyprintk
 13 input
 254 gpiochip

Block devices:
  7 loop
  8 sd
```

## Device File creation

There are two ways to do the device file creation. 
1. Manually using mknod command
2. Programatically using device_create().

To create a character or block device in Linux, mknod command is used. A major number, minor number, device type and its name is specified. Essentially, this command 
adds a major number and minor number to a device structure in the kernel.
```
mknod [OPTION]... NAME TYPE [MAJOR MINOR]
```

To check the major and minor number, ls -l /dev/ command can be used. The first character 'c' in the output signifies this as a character device.
In the below example, 4 is major number. The number following comma is minor number.

```
ls -l /dev/tty*
crw--w---- 1 root   tty     4,  0 Oct 14 01:50 /dev/tty0
crw--w---- 1 root   tty     4,  1 Oct 14 01:54 /dev/tty1
crw--w---- 1 root   tty     4, 10 Oct 14 01:50 /dev/tty10
crw--w---- 1 root   tty     4, 11 Oct 14 01:50 /dev/tty11
crw--w---- 1 root   tty     4, 12 Oct 14 01:50 /dev/tty12
crw--w---- 1 root   tty     4, 13 Oct 14 01:50 /dev/tty13
crw--w---- 1 root   tty     4, 14 Oct 14 01:50 /dev/tty14
crw--w---- 1 root   tty     4, 15 Oct 14 01:50 /dev/tty15
crw--w---- 1 root   tty     4, 16 Oct 14 01:50 /dev/tty16
crw--w---- 1 root   tty     4, 17 Oct 14 01:50 /dev/tty17
crw--w---- 1 root   tty     4, 18 Oct 14 01:50 /dev/tty18
crw--w---- 1 root   tty     4, 19 Oct 14 01:50 /dev/tty19

```

## A simple char device

Lets create a simple char device using mknod command.
```
# mknod /dev/char0 c 449 0
# ls -l /dev/char0
crw-r--r-- 1 root root 449, 0 Oct 17 13:46 /dev/char0
```
But the device is not listed in /proc/devices yet. Nor it has any file operation when we do "cat /dev/char0". We need to associate a driver to make it visible in /proc/devices.

```
# cat /dev/char0
cat: /dev/char0: No such device or address
```
## A simple char driver

Let's write a simple character driver. This char driver implements dummy open/read/close file operations and registers major number and device name using register_chrdev() function.

```
#include<linux/module.h>
#include<linux/fs.h>

MODULE_AUTHOR("Karthikeyan Arulmozhivarman");
MODULE_DESCRIPTION("Simple character driver");
MODULE_LICENSE("GPL");

ssize_t simple_char_driver_read (struct file *file, char __user *buf, size_t length, loff_t *offset)
{
	return 0;
}

int simple_char_driver_open (struct inode *inode, struct file *file)
{
	printk(KERN_ALERT "OPEN: Simple Character Driver\n");
	return 0;
}


int simple_char_driver_close (struct inode *inode, struct file *file)
{
	printk(KERN_ALERT "CLOSE: Simple Character Driver\n");
	return 0;
}

struct file_operations simple_char_fops = {
	.owner   = THIS_MODULE,
	.read    = simple_char_driver_read,
	.open    = simple_char_driver_open,
	.release = simple_char_driver_close
};

static int simple_char_driver_init(void)
{
	printk(KERN_ALERT "INIT: Simple Character Driver\n");
	/* register the device */
	register_chrdev( 449, "char0", &simple_char_fops);
	return 0;
}

static void simple_char_driver_exit(void)
{
	printk(KERN_ALERT "EXIT: Simple Character Driver\n");
	/* unregister the device */
	unregister_chrdev( 449, "char0");
}

module_init(simple_char_driver_init);
module_exit(simple_char_driver_exit);

```

Compile this simple driver and perform insmod.

```
# make
make -C /lib/modules/6.2.0-34-generic/build M=/home/karkey/char_Driver modules
make[1]: Entering directory '/usr/src/linux-headers-6.2.0-34-generic'
warning: the compiler differs from the one used to build the kernel
  The kernel was built by: x86_64-linux-gnu-gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
  You are using:           gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
  CC [M]  /home/karkey/char_Driver/simple_char.o
  MODPOST /home/karkey/char_Driver/Module.symvers
  CC [M]  /home/karkey/char_Driver/simple_char.mod.o
  LD [M]  /home/karkey/char_Driver/simple_char.ko
  BTF [M] /home/karkey/char_Driver/simple_char.ko
Skipping BTF generation for /home/karkey/char_Driver/simple_char.ko due to unavailability of vmlinux
make[1]: Leaving directory '/usr/src/linux-headers-6.2.0-34-generic'

```

Once the driver is inserted, we can see our device appears in /proc/devices.

```
# insmod simple_char.ko 
# cat /proc/devices
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  5 ttyprintk
 13 input
 254 gpiochip
 449 char0

Block devices:
  7 loop
  8 sd
```

dmesg shows the below output.
```
[40073.638685] INIT: Simple Character Driver
```

Let's try to perform read operation on this device.

```
# cat /dev/char0
# dmesg
[40449.712221] OPEN: Simple Character Driver
[40449.712247] READ: Simple Character Driver
[40449.712272] CLOSE: Simple Character Driver
```
## udevadm monitor command
The driver association and disassociation to a device can be checked using udevadm monitor command.

```
# udevadm monitor
monitor will print the received events for:
UDEV - the event which udev sends out after rule processing
KERNEL - the kernel uevent

KERNEL[38600.794431] add      /module/simple_char (module)
UDEV  [38600.810820] add      /module/simple_char (module)
KERNEL[39012.912445] remove   /module/simple_char (module)
UDEV  [39012.920846] remove   /module/simple_char (module)
```

## Note
This character device file is not directly associated to any real device or hardware. In the next post, we will see how to associate a real device to this driver.

## References:
1. https://embetronicx.com/tutorials/linux/device-drivers/device-file-creation-for-character-drivers/
2. http://derekmolloy.ie/writing-a-linux-kernel-module-part-2-a-character-device/
