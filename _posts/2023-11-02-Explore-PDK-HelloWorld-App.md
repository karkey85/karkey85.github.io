# Explore DPDK: Hello world app

Download latest stable DPDK tar ball from https://core.dpdk.org/download/
```
wget https://fast.dpdk.org/rel/dpdk-23.07.tar.xz
tar xz dpdk-23.07.tar.xz
cd dpdk-23.07
```
Before using meson to  configure and build, the following dependencies we need to install.

```
sudo apt-get update
sudo apt-get install -y meson
sudo apt-get install -y python3-pyelftools
sudo apt-get install -y libnuma-dev
```
To build DPDK libraries, drivers and test applications, the meson and ninja tools are used as shown below.

```
meson build
ninja -C build
```
The build takes roughly 10 minutes for this to complete.

![img-description](/img/dpdk1.JPG)

_Ninja Build_

Let's first build this "helloworld" application and give it a run.
```
$ meson configure -Dexamples=helloworld build
$ ninja -C build
$ cd build/examples
$ ls
dpdk-helloworld  dpdk-helloworld.p  helloworld
```
It is time for huge pages. Else we would see the following failures when running the helloworld application.

```
 karkey  ... | dpdk-23.07 | build | examples  ./dpdk-helloworld
EAL: Detected CPU lcores: 2
EAL: Detected NUMA nodes: 1
EAL: Detected static linkage of DPDK
EAL: Multi-process socket /run/user/1000/dpdk/rte/mp_socket
EAL: Selected IOVA mode 'VA'
EAL: No free 2048 kB hugepages reported on node 0
EAL: FATAL: Cannot get hugepage information.
EAL: Cannot get hugepage information.
PANIC in main():
Cannot init EAL
0: ./dpdk-helloworld (rte_dump_stack+0x42) [55570d7a9c72]
1: ./dpdk-helloworld (__rte_panic+0xcd) [55570d3ce576]
2: ./dpdk-helloworld (55570d29d000+0x1122fc) [55570d3af2fc]
3: /lib/x86_64-linux-gnu/libc.so.6 (7f66b0600000+0x29d90) [7f66b0629d90]
4: /lib/x86_64-linux-gnu/libc.so.6 (__libc_start_main+0x80) [7f66b0629e40]
5: ./dpdk-helloworld (_start+0x25) [55570d5af3a5]
Aborted (core dumped)

```
Jump into the super user mode and create huge pages. A DPDK application requires huge pages to perform its function.
```
  mkdir -p /dev/hugepages
  mountpoint -q /dev/hugepages || mount -t hugetlbfs nodev /dev/hugepages
  echo 64 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
```
Our first helloworld DPDK application.
```
root@karkey-virtual-machine:/home/karkey/dpdk/dpdk-23.07/build/examples# ./dpdk-helloworld
EAL: Detected CPU lcores: 2
EAL: Detected NUMA nodes: 1
EAL: Detected static linkage of DPDK
EAL: Multi-process socket /var/run/dpdk/rte/mp_socket
EAL: Selected IOVA mode 'PA'
TELEMETRY: No legacy callbacks, legacy socket not created
hello from core 1
hello from core 0
```

### References:
1. https://core.dpdk.org/doc/quick-start/
2. https://doc.dpdk.org/guides-23.07/linux_gsg/build_dpdk.html#compiling-and-installing-dpdk-system-wide
