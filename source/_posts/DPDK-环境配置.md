---
title: DPDK 环境配置
date: 2024-06-09 22:56:38
tags: [DPDK]
---

# 环境配置

### DPDK系统要求：

Kernel version >= 4.14
GCC（版本 5.0+）或 Clang（版本 3.6+）
Python 3.6 或更高版本
Meson (version 0.53.2+) and ninja
pyelftools （版本 0.22+）


### 笔者环境：

内核版本： 6.6.32-1-lts
发行版： ArchLinux
gcc 版本 14.1.1 20240522
Python 3.12.3
Meson 1.4.1
pyelftools 0.31-2

### 环境配置

1. 检查当前内核是否支持

```bash
# 查看当前内核是否支持
zgrep -E "HUGETLBFS|PROC_PAGE_MONITOR" /proc/config.gz
CONFIG_PROC_PAGE_MONITOR=y
CONFIG_HUGETLBFS=y
# 查看高精度事件定时器
zgrep -E "HPET" /proc/config.gz     
CONFIG_HPET_TIMER=y
CONFIG_HPET_EMULATE_RTC=y
CONFIG_HPET=y
# CONFIG_HPET_MMAP is not set
```

如果不支持`HUGETLBFS`或者`PROC_PAGE_MONITOR`，重编内核。

2. 安装上述环境

```bash
sudo pacman -S gcc clang pkgconf python meson ninja python-pyelftools numactl
sudo pacman -S libarchive libelf
```

3. 配置大页

最简单的配置，只使用2MB的大页，没有配置1GB的，如果有需要可以官网查教程。

非root用户使用`sudo vim`操作下面文件填写自己认为适合的数值，不太离谱都行。
```bash
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

HugePages_Total不为0就代表成功。
```bash
➜  ~ cat /proc/meminfo | grep Huge
AnonHugePages:   3670016 kB
ShmemHugePages:        0 kB
FileHugePages:    190464 kB
HugePages_Total:    1024
HugePages_Free:     1024
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:         2097152 kB
```

### 编译安装
下载一个自己喜欢的DPDK版本，懒得话就：
```bash
git clone http://dpdk.org/git/dpdk --depth=1
```

进入项目主目录:
```bash
meson setup build
cd build
ninja
sudo meson install
```

编译安装完成后，查看路径`/usr/local/lib/pkgconfig`有无`和libdpdk.pc`，没有的话全局找一下该文件路径，并且在`~/.zshrc`添加`export PKG_CONFIG_PATH=/path/to/`，我本机设置`export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig`，完成后进行下面命令：

```bash
sudo ldconfig
```

# Hello World

简单运行一下：
```bash
cd examples/helloworld
make
cd build
sudo ./helloworld
```

Output:
```bash
➜  build git:(main) sudo ./helloworld
EAL: Detected CPU lcores: 16
EAL: Detected NUMA nodes: 1
EAL: Detected shared linkage of DPDK
EAL: Multi-process socket /var/run/dpdk/rte/mp_socket
EAL: Selected IOVA mode 'VA'
TELEMETRY: No legacy callbacks, legacy socket not created
hello from core 1
hello from core 2
hello from core 3
hello from core 4
hello from core 5
hello from core 6
hello from core 7
hello from core 8
hello from core 9
hello from core 10
hello from core 11
hello from core 12
hello from core 13
hello from core 14
hello from core 15
hello from core 0
```

