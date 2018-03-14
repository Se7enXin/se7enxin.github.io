---
title: Raspberry in common use
description: Raspberry in common use
categories:
- Raspberry
tags:
- Raspberry
---

<!-- more -->

# 一、[Raspberry](https://www.raspberrypi.org/)

# 二、Config

## （1）设置静态IP

    /etc/network/interfaces

    auto eth0
    iface eth0 inet static
    address xxx.xxx.xxx.xxx
    gateway xxx.xxx.xxx.xxx
    netmask 255.255.255.0

重置网络：

```sh
$ /etc/init.d/networking restart
```

## （2）设置路由

```sh
$ route add default gw xxx.xxx.xxx.xxx
```

## （3）DNS

    /etc/resolv.conf

## （4）挂载

```sh
$ mount -o nolock,wsize=1024,rsize=1024 目标主机IP:挂载路径 树莓派路径
```

## （5）[显示器分辨率设置](https://elinux.org/RPiconfig)

## （6）[禁止屏幕休眠](https://wiki.archlinux.org/index.php/Display_Power_Management_Signaling)

禁止屏幕进入保护和关闭状态：

    /etc/bash.bashrc

    setterm -blank 0 -powerdown 0

## （7）[制作镜像](https://www.jianshu.com/p/b3ce0d945c7a)

## （8）enable ssh

1. Enter `sudo raspi-config` in a terminal window
2. Select `Interfacing Options`
3. Navigate to and select `SSH`
4. Choose `Yes`
5. Select `Ok`
6. Choose `Finish`

```sh
$ service sshd restart
```
