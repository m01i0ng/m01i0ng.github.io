---
title: Linux 归纳，持续更新
date: 2019-07-11 10:43:07
tags:
  - Linux
categories: Linux
---

## 系统优化

### CT 7磁盘扩容

> 场景：VM 虚拟机磁盘分配不足后期添加并扩容至主分区

- 查看系统挂载信息

`df -h`

- 在 VM 设置中扩展磁盘

- 对新增加的硬盘进行分区、格式化

```bash
# 我们增加了空间的硬盘是 /dev/sda

# 分区：
[root@localhost]# fdisk /dev/sda　　　　
p　　　　　　　查看已分区数量（我看到有两个 /dev/sda1 /dev/sda2）
n　　　　　　　新增加一个分区
p　　　　　　　分区类型我们选择为主分区
　　　　　　     分区号输入3（因为1,2已经用过了,sda1是分区1,sda2是分区2,sda3分区3）
回车　　　　　  默认（起始扇区）
回车　　　　　  默认（结束扇区）
t　　　　　　　 修改分区类型
　　　　　　     选分区3
8e　　　　　 　修改为LVM（8e就是LVM）
w　　　　　  　写分区表
q　　　　　  　完成，退出fdisk命令

# 使用partprobe 命令 或者重启机器 

# 格式化分区3命令:

mkfs.ext3 /dev/sda3
```

- 添加新 LVM 至已有 LVM 组，实现扩容

```bash
lvm　　　　　　　　　　　　           进入lvm管理

lvm>pvcreate /dev/sda3　　           这是初始化刚才的分区3

lvm>vgextend centos /dev/sda3     将初始化过的分区加入到虚拟卷组centos (卷和卷组的命令可以通过 vgdisplay )

lvm>vgdisplay -v 或 者vgdisplay 查看 free PE /Site

lvm>lvextend -l+6143 /dev/mapper/centos-root　　扩展已有卷的容量（6143 是通过vgdisplay查看free PE /Site的大小）

lvm>pvdisplay 查看卷容量，这时你会看到一个很大的卷了

lvm>quit 　退出
```

接着输入以下命令：

`xfs_growfs /dev/mapper/centos-root`

重复执行 `df -h` 验证

### 添加 Swap 空间

- 制作 Swapfile 文件

Linux：`fallocate -l 4G /swapfile && chmod 600 /swapfile`

CentOS：`dd if=/dev/zero of=/swapfile count=4096 bs=1MiB && chmod 600 /swapfile`

- 格式化交换空间

`mkswap /swapfile`

- 开启交换空间

`swapon /swapfile`

- 开机自动挂载

`vim /etc/fstab` 把原有交换空间文件替换为新的

### 设置时区

- 查看可用时区

`timedatectl list-timezones`

- 设置时区

`timedatectl set-timezone Asia/Shanghai`

- 更新

`timedatectl`

### 设置 NTP

- 安装 NTP

`yum install -y ntp`

- 开启

`systemctl start ntpd && systemctl enable ntpd`
