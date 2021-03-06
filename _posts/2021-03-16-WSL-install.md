---
title: WSL安装卸载
author: lijiabao
date: 2021-03-16 13:00
description: 在这里
categories:  Windows
tags: Windows

---

主要介绍一下Windows下的Linux子系统的安装

## Windows的Linux子系统

### 安装方法

[详细的安装教程网址](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-2--check-requirements-for-running-wsl-2),这是Windows官网给出的安装教程，十分详细，且方便复制一些其中的命令，里面还有安装失败的解决方案

详细的安装步骤：

#### 步骤1：启动适用与Linux的Windows子系统

首先需要启动适用于Linux的Windows子系统的可选功能，然后才可以安装Linux分发版，有两种方法开启：

1. 使用powershell命令行开启
   ```powershell
   dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
   ```
   
2. 找到设置启动或关闭Windows功能，进入，找到Linux的Windows子系统选项勾选

完成操作后重启系统

#### 步骤2：检查运行WSL 2的要求

若要更新到 WSL 2，需要运行 Windows 10。

- 对于 x64 系统：**版本 1903** 或更高版本，采用 **内部版本 18362** 或更高版本。
- 对于 ARM64 系统：**版本 2004** 或更高版本，采用 **内部版本 19041** 或更高版本。
- 低于 18362 的版本不支持 WSL 2。 需要先更新系统

若要检查 Windows 版本及内部版本号，选择 Windows 徽标键 + R，然后键入“winver”，选择“确定”。 （或者在 Windows 命令提示符下输入 `ver` 命令）。

#### 步骤 3 - 启用虚拟机功能

安装 WSL 2 之前，必须启用“虚拟机平台”可选功能。 计算机需要[虚拟化功能](https://docs.microsoft.com/zh-cn/windows/wsl/troubleshooting#error-0x80370102-the-virtual-machine-could-not-be-started-because-a-required-feature-is-not-installed)才能使用此功能。以管理员身份打开 PowerShell 并运行下面命令后重启：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

#### 步骤 4 - 下载 Linux 内核更新包

1. 下载最新包：
   - [适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)。如果使用的是 ARM64 计算机，请下载 [ARM64 包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_arm64.msi)。 如果不确定自己计算机的类型，请打开命令提示符或 PowerShell，并输入：`systeminfo`查看
2. 运行上一步中下载的更新包。 （双击以运行 - 系统将提示你提供提升的权限，选择“是”以批准此安装。）

#### 步骤 5 - 将 WSL 2 设置为默认版本

打开 PowerShell，然后在安装新的 Linux 发行版时运行以下命令，将 WSL 2 设置为默认版本：

```powershell
wsl --set-default-version 2
```

#### 步骤 6 - 安装所选的 Linux 分发

1. 打开Microsoft Store，并选择你偏好的 Linux 分发版。
2. 选择好之后选择获取即可
3. 安装好之后会跳出创建用户和密码的窗口，设置好即可

除了上面的方法，还可以选择手动下载并安装[手动下载Linux分发版Linux子系统](https://docs.microsoft.com/zh-cn/windows/wsl/install-manual)。也可以参照后面的[不安装到默认盘的方法](#不安装到默认盘的方法)

#### 步骤7 - 安装Windows终端（可选）

Windows 终端可启用多个选项卡（在多个 Linux 命令行、Windows 命令提示符、PowerShell 和 Azure CLI 等之间快速切换）、创建键绑定（用于打开或关闭选项卡、复制粘贴等的快捷方式键）、使用搜索功能，以及使用自定义主题（配色方案、字体样式和大小、背景图像/模糊/透明度）。 [了解详细信息。](https://docs.microsoft.com/zh-cn/windows/terminal)

[安装 Windows 终端](https://docs.microsoft.com/zh-cn/windows/terminal/get-started)。

### 不安装到默认盘的方法

首先你需要首先在[手动下载Linux分发版Linux子系统](https://docs.microsoft.com/zh-cn/windows/wsl/install-manual)下载好你需要安装的Linux分发版。可能下载的时候后缀是appx，你需要先将其改为zip压缩包版本，然后将其解压到你想要安装发行版本的位置。

然后在你的解压好的文件里面找到你的系统的可执行文件，双击安装等待安装完成即可。

<font color='red'>注意！</font>如果你解压运行了安装文件之后，就不要再修改文件夹的名字了，会导致打不开分发版的问题。你需要先修改压缩解压后的文件夹的名字，再运行安装文件

安装好之后可以使用安装文件来打开，或者使用命令行的wsl命令来打开。

### 迁移安装好的到其他盘

下载[迁移工具](https://github.com/DDoSolitary/LxRunOffline/releases/tag/v3.4.1).然后解压该工具，进入到你的解压目录下面，使用下面的命令：

```powershell
.\LxRunOffline.exe list  # 查看安装的版本
.\LxRunOffline.exe move -n linux-name -d 'target-linux-directory'
# 其中Linux-name是上一步命令查看到的你需要移动的Linux
# target-linux-directory是你想要移动到的Linux目录。
```

### 启动方法

1. 直接使用你的安装执行文件就可以打开，如`ubuntu2004.exe`安装时运行的可执行文件

2. 在命令行下面使用`wsl`启动进入命令行

3. 如果你是安装了多个Linux分发版的用户，如果使用第二个方法来启动的话，你需要先指定wsl启动的分发版

   ```powershell
   # 查看系统上安装的Linux分发版
   wslconfig /l
   # 修改wsl指向的分发版
   wslconfig /s Ubuntu20.04
   # 其中的Ubuntu20.04是查看到的分发版的名字,运行完之后,再使用wsl就是打开你指定的这个分发版
   ```

   