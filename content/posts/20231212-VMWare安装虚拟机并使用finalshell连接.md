---
title: "VMWare安装虚拟机并使用finalshell连接"
date: 2023-12-12T21:59:56+08:00
draft: false
tags: ['linux']
categories: ["服务器篇"]
---

## 镜像下载
CentOS 7 镜像 : [CentOS-7-x86_64-DVD-2009.iso](https://mirrors.aliyun.com/centos/7/isos/x86_64/)

## 虚拟机创建
创建时点击下一步即可

在最后一步进入自定义硬件 在 新CD/DVD(IDE)处选择已安装的镜像

安装好镜像后打开虚拟机根据提示进行系统安装,此时需要记住用户名和密码

安装好后在系统中输入用户名密码登录

输入`ip a`查看IP信息

```shell
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
```

输入下方命令进入文件编辑页面
```shell
 vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
将下方的`ONBOOT`属性改为`yes`
```vi
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
DEVICE=ens33
ONBOOT=yes
```
重启虚拟机,再次输入`ip a`
```shell
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:86:60:8b brd ff:ff:ff:ff:ff:ff
    inet 192.168.111.4/24 brd 192.168.1.255 scope global noprefixroute dynamic ens33
       valid_lft 603794sec preferred_lft 603794sec
    inet6 fe80::ed92:5253:424e:58b5/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
此时宿主机和虚拟机尝试ping ip地址,如未ping 通可关闭防火墙

## finalshell连接

finalshell连接是可能会出现`java.net.ConnectException: Connection refused`报错,这个原因可能是宿主机和虚拟机不在同一个网段导致,我们需要在VMWare虚拟网络编辑器中新建一个VMnet0为桥接模式,vmnet信息中选中桥接模式将虚拟机连接到外部网络,桥接到宿主机上,同时编辑虚拟机机配置,网络适配器->网络连接->自定义虚拟网络为刚刚我们设置的VMnet0

如果在设置桥接时提示`无法将网络更改为桥接状态`可以在卸载页面更改VMWare应用,对该应用进行修复,重新启动后即可

进行完上述操作即可在finalshell中,对VMWare我们创建的CentOS虚拟机进行连接
