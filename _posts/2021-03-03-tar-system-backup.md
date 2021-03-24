---
title: tar系统备份
author: lijiabao
date: 2021-02-15 23:05:20.275529900 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
###### 系统备份
使用tar做一个整个系统的备份（节省硬盘空间）：
> 1. 从liveCD启动
> 2. 改变跟目录到安装的linux下面：`mkdir /mnt/arch`next `mount /dev/archroot-dir /mnt/arch`next`cd /mnt/arch`next `chroot . /bin/bash`==warning==在这里注意不要使用arch-chroot，这会出问题
> 3. 如果有额外的分区，需要挂载好
> 4. 添加不备份的限制，创建一个exclude_file，然后在脚本文件中指定
```
# Not old backups
/opt/backup/arch-full\*

# Not temporary files
/tmp/*

# Not the cache for pacman
/var/cache/pacman/pkg/
# 支持正则表达式，忽略指定的这些目录不备份
```

> 5. 使用下面的备份脚本进行备份
> 6. 恢复备份文件`tar --acls --xattrs -xpf backupfile`
```
#!/bin/bash
# full system backup

# Backup destination
backdest=/opt/backup

# Labels for backup name
#PC=${HOSTNAME}
pc=pavilion
distro=arch
type=full
date=$(date "+%F")
backupfile="$backdest/$distro-$type-$date.tar.gz"

# Exclude file location
prog=${0##\*/} # Program name from filename
excdir="/home/<user>/.bin/root/backup"
exclude\_file="$excdir/$prog-exc.txt"

# Check if chrooted prompt.
echo -n "First chroot from a LiveCD.  Are you ready to backup? (y/n): "
read executeback

# Check if exclude file exists
if \[ ! -f $exclude\_file \]; then
  echo -n "No exclude file exists, continue? (y/n): "
  read continue
  if \[ $continue == "n" \]; then exit; fi
fi

if \[ $executeback = "y" \]; then
  # -p, --acls and --xattrs store all permissions, ACLs and extended attributes. 
  # Without both of these, many programs will stop working!
  # It is safe to remove the verbose (-v) flag. If you are using a 
  # slow terminal, this can greatly speed up the backup process.
  tar --exclude-from=$exclude\_file --acls --xattrs -cpvf $backupfile /
fi
```


###### Backup with parallel compression

To back up using parallel compression ([SMP](https://en.wikipedia.org/wiki/Symmetric_multiprocessing "wikipedia:Symmetric multiprocessing")), use [pbzip2](https://archlinux.org/packages/?name=pbzip2) (Parallel bzip2):

`tar -cvf /path/to/chosen/directory/etc-backup.tar.bz2 -I pbzip2 /etc`

Store `etc-backup.tar.bz2` on one or more offline media, such as a USB stick, external hard drive, or CD-R. Occasionally verify the integrity of the backup process by comparing original files and directories with their backups. Possibly maintain a list of hashes of the backed up files to make the comparison quicker.

Restore corrupted `/etc` files by extracting the `etc-backup.tar.bz2` file in a temporary working directory, and copying over individual files and directories as needed. To restore the entire `/etc` directory with all its contents execute the following command as root:

`tar -xvf etc-backup.tar.bz2 -C `
