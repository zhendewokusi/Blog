---
title: ArchLinux 扬声器不存在解决方案
date: 2024-10-19 20:52:21
tags: [ArchLinux]
---

## 问题

本机环境：
	OS: Arch Linux x86_64
	声卡：C-Media_R__Audio

这几天面试，发现电脑能正常输入音频数据（麦克风），但是无法接收音频数据（扬声器）。
具体问题是：
   1. `pavucontrol`命令打开的界面，`输出设备`中只有一个 HDMI 的输出（笔记本外接显示器）。
   2. `alsamixer`进入后按`F6`查看有`HDA NVidia`和`C-Media(R) Audio`两个声卡，且重新选择后`pavucontrol`依旧只有一个 HDMI 的输出。电脑无法进行音量输出。
   3. 排除硬件故障，在Windows中能正常工作。

## 解决方案

安装 pulseaudio 以及 pulseaudio-alsa，或者安装 pipewire-pulse、pipewire-alsa、wireplumber。二选一，都装会有冲突。我本机是使用的 pulseaudio。

使用`aplay -l`命令查看自己有几个声卡，这一步输出后续会用到：
```bash
➜  ~ aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: NVidia [HDA NVidia], device 3: HDMI 0 [27M1C5500V]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: NVidia [HDA NVidia], device 7: HDMI 1 [HDMI 1]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: Audio [C-Media(R) Audio], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

修改前先列出系统中所有可用的音频输出设备（sinks）。
```bash
pactl list short sinks
# 此时没有 C-Media Audio 的输出设备
# 1	alsa_output.pci-0000_01_00.1.hdmi-stereo        module-alsa-card.c      s16le 2ch 44100Hz       IDLE
```

之后重新加载相关模块后再查看可用音频输出设备
```bash
pactl unload-module module-udev-detect
pactl load-module module-udev-detect
pactl list short sinks
1       alsa_output.hw_1_0      module-alsa-sink.c      s16le 2ch 44100Hz       IDLE
2       alsa_output.pci-0000_01_00.1.hdmi-stereo        module-alsa-card.c      s16le 2ch 44100Hz       IDLE
```
此时在`pavucontrol`中应该也能查看到网卡设备，选择输出就行，问题基本上就解决了，应该是模块加载顺序可能有问题，导致某些设备没有被正常检测到，重新加载一下就好了。唯一的问题是每次开机都要重新加载模块，这比较麻烦，此时就需要修改相关配置文件：
```bash
sudo vim /etc/pulse/default.pa

# 如果有，将这一行前面的注释去除
load-module module-udev-detect

# 最后在该文件末尾添加（1,0 含义是前面 aplay -l 输出的信息， card "1", device "0", PCI设备管理相关的，知道怎么改自己的就够了）
load-module module-alsa-sink device=hw:1,0
```

重启查看是否能检测到扬声器。