---
title: automount at boot
author: lijiabao
date: 2021-02-18 15:07:41.445883700 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
#### 开机自动挂载想要的分区
`fdisk`查看分区信息
`lsblk` 查看分区信息和挂载点

首先使用`blkid`命令查询系统的所有分区的uuid信息

然后可以在`/etc/fstab`分区配置文件中添加挂载分区的信息
格式：
`<device>    <dir> <type> <options> <dump> <fsck>`
-   `<device>` describes the block special device or remote filesystem to be mounted; see 
-   `<dir>` describes the mount point
-   `<type>` the file system "File system" type.`ext4,ntfs,swap`
-   `<options>` the associated mount options
-   `<dump>` is checked by the dump(8) utility. This field is usually set to `0`, which disables the check.
-   `<fsck>` sets the order for filesystem .e. For the root device it should be `1`. For other partitions it should be `2`, or `0` to disable checking.

有两种挂载分区的格式：
1. `/dev/partition   mount_point   filesystem   default   0 0`
2. `UUID=partition_uuid  mount_point   filesystem   default  0 0`