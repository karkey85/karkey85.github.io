
## A Simple Linux Character Device Driver 

# Everything in Linux is a file

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

# Device File creation

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

# A simple char driver

```
```
