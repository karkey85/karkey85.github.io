
Let's bind dpdk driver to one of the NIC and try sending packets using one of sample dpdk applications. 

The kernel module igb_uio is moved to the dpdk-kmods repository in the /linux/igb_uio/ directory since DPDK 20.11.
```
$ git clone git://dpdk.org/dpdk-kmods
$ cp -r ./dpdk-kmods/linux/igb_uio kernel/linux/
$ ls ./dpdk/kernel/linux/
igb_uio  kni  meson.build
```

Now add igb_uio in kernel/linux/meson.build.
```
subdirs = ['kni', 'igb_uio']
```

Then create a file of meson.build in kernel/linux/igb_uio/.
```
# SPDX-License-Identifier: BSD-3-Clause
# Copyright(c) 2017 Intel Corporation

mkfile = custom_target('igb_uio_makefile',
        output: 'Makefile',
        command: ['touch', '@OUTPUT@'])

custom_target('igb_uio',
        input: ['igb_uio.c', 'Kbuild'],
        output: 'igb_uio.ko',
        command: ['make', '-C', kernel_dir + '/build',
                'M=' + meson.current_build_dir(),
                'src=' + meson.current_source_dir(),
                'EXTRA_CFLAGS=-I' + meson.current_source_dir() +
                        '/../../../lib/librte_eal/include',
                'modules'],
        depends: mkfile,
        install: true,
        install_dir: kernel_dir + '/extra/dpdk',
        build_by_default: get_option('enable_kmods'))
```

Let's build this igb_uio kernel module.
```
$ meson configure -Denable_kmods=true build
```

The build fails as shown below:
```
$ ninja -C build
ninja: Entering directory `build'
[0/1] Regenerating build files.
The Meson build system
Version: 0.61.2
Source dir: /home/karkey/dpdk/dpdk-23.07
Build dir: /home/karkey/dpdk/dpdk-23.07/build
Build type: native build

Compiler for C supports arguments -Wno-format-truncation: YES (cached)

../kernel/linux/igb_uio/meson.build:8:0: ERROR: Unknown variable "kernel_dir".

A full log can be found at /home/karkey/dpdk/dpdk-23.07/build/meson-logs/meson-log.txt
FAILED: build.ninja 
/usr/bin/meson --internal regenerate /home/karkey/dpdk/dpdk-23.07 /home/karkey/dpdk/dpdk-23.07/build --backend ninja
ninja: error: rebuilding 'build.ninja': subcommand failed

```
The build fails due to unknown variable "kernel_dir" in meson.build under kernel/linux/igb_uio. As a quick hack, replacing "kernel_dir" in meson.build as '/lib/modules/6.2.0-37-generic'.

```
# cat kernel/linux/igb_uio/meson.build 
# SPDX-License-Identifier: BSD-3-Clause
# Copyright(c) 2017 Intel Corporation

mkfile = custom_target('igb_uio_makefile',
        output: 'Makefile',
        command: ['touch', '@OUTPUT@'])

custom_target('igb_uio',
        input: ['igb_uio.c', 'Kbuild'],
        output: 'igb_uio.ko',
        command: ['make', '-C', '/lib/modules/6.2.0-37-generic' + '/build',
                'M=' + meson.current_build_dir(),
                'src=' + meson.current_source_dir(),
                'EXTRA_CFLAGS=-I' + meson.current_source_dir() +
                        '/../../../lib/librte_eal/include',
                'modules'],
        depends: mkfile,
        install: true,
        install_dir: '/lib/modules/6.2.0-37-generic' + '/extra/dpdk',
        build_by_default: get_option('enable_kmods'))

```
Finally, the meson build succeeded.

```
$ ninja -C build
make: Entering directory '/usr/src/linux-headers-6.2.0-37-generic'
warning: the compiler differs from the one used to build the kernel
  The kernel was built by: x86_64-linux-gnu-gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
  You are using:           gcc-11 (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
  CC [M]  /home/karkey/dpdk/dpdk-23.07/build/kernel/linux/igb_uio/igb_uio.o
  MODPOST /home/karkey/dpdk/dpdk-23.07/build/kernel/linux/igb_uio/Module.symvers
  CC [M]  /home/karkey/dpdk/dpdk-23.07/build/kernel/linux/igb_uio/igb_uio.mod.o
  LD [M]  /home/karkey/dpdk/dpdk-23.07/build/kernel/linux/igb_uio/igb_uio.ko
  BTF [M] /home/karkey/dpdk/dpdk-23.07/build/kernel/linux/igb_uio/igb_uio.ko
Skipping BTF generation for /home/karkey/dpdk/dpdk-23.07/build/kernel/linux/igb_uio/igb_uio.ko due to unavailability of vmlinux
make: Leaving directory '/usr/src/linux-headers-6.2.0-37-generic'

```
Let's see the status of all interface drivers using devbind command.
```
# ./usertools/dpdk-devbind.py --status

Network devices using kernel driver
===================================
0000:02:01.0 '82545EM Gigabit Ethernet Controller (Copper) 100f' if=ens33 drv=e1000 unused=igb_uio,vfio-pci,uio_pci_generic *Active*
```
In the above output,  the PCI number, the driver (e1000) is in use and the unused drivers which is currently attached to this ens33 interface.
 table indicates that interface 0000:02:01.0 is active. 


The "ens33" interface should not be active when binding this dpdk driver to it.

```
# ifconfig ens33 down
# ./dpdk-devbind.py --bind=igb_uio 02:01.0
# ./dpdk-devbind.py --status

Network devices using DPDK-compatible driver
============================================
0000:02:01.0 '82545EM Gigabit Ethernet Controller (Copper) 100f' drv=igb_uio unused=e1000,vfio-pci,uio_pci_generic
```

References:
1. https://doc.dpdk.org/guides/linux_gsg/linux_drivers.html?highlight=igb_uio
