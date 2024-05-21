---
title: gazelle 环境搭配
date: 2024-05-16 21:20:18
tags:
---

分享一下搭建gazelle环境的步骤。
前提： OpenEuler 的 docker 环境。

```bash
yum install gcc
yum install make
yum install meson
yum install python3-pip
pip3 install pyelftools
yum install numactl-devel
yum install libpcap-devel

```

从 DPDK 官网获取最新稳定版本DPDK（yum下载的还是kernel 5.*版本，到时候第三方模块加载不进去）
