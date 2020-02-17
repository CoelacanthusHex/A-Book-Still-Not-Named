# Arch 安装指南 

First Post: 2020-2-17 By Coelacanthus

本指南用于使用 LiveCD 的安装方法

## WARNING 

所有 2019-10-06 之后未更新的教程都会导致安装失败  
原因参见[`base` 元包替代了同名的包组并且要求安装，需要手动干预升级](https://www.archlinuxcn.org/base-group-replaced-by-mandatory-base-package-manual-intervention-required/)

## 安装前的准备

## 启动 LiveCD

## 配置网络环境

强烈建议使用有线网络

### 有线网络

家庭网络一般是动态ip，无需额外操作

### 无线网络

使用 `wifi-menu` ，该软件为TUI软件，根据提示操作即可

**Note: 不支持同时有账号和密码的企业验证**

### 检验网络连接

```
# ping baidu.com
```

## 更新系统时间

启用ntp以确保系统时间是准确的：
```
# timedatectl set-ntp true
```

## 选择镜像

>	文件 /etc/pacman.d/mirrorlist 定义了软件包会从哪个 镜像源 下载。在 LiveCD 启动的系统上，所有的镜像都被启用，并且在镜像被制作时，我们已经通过他们的同步情况和速度排序。  
>	在列表中越前的镜像在下载软件包时有越高的优先权。你可以相应的修改文件 /etc/pacman.d/mirrorlist，并将地理位置最近的镜像源挪到文件的头部，同时你也应该考虑一些其他标准。

以上摘自 Arch Wiki  

在中国大陆，建议选择TUNA源或者USTC源

## 验证启动模式

如果以在 UEFI 主板上启用 UEFI 模式，Archiso 将会使用 systemd-boot 来 启动 Arch Linux。可以列出 efivars 目录以验证启动模式：
```
# ls /sys/firmware/efi/efivars
```
如果目录不存在，系统可能以 BIOS 或 CSM 模式启动，请跳转至[#BIOS启动](#BIOS启动)

## UEFI启动

### 硬盘分区

### 格式化并挂载分区

### 安装必须的软件包

```
pacstrap /mnt base linux linux-firmware dhclient dialog diffutils wireless_tools wpa_supplicant nano vim sudo man-db man-pages netctl
```
其中，`linux` 包可替换为其他内核，如：  
*	linux: 应用了一些补丁的官方内核，最常用
*	linux-hardened: 一个注重安全性的Linux内核，应用了一组强化补丁来减轻内核和用户空间的攻击。
*	linux-lts: 长期支持内核
*	linux-zen: 打了一系列优化补丁的内核

### 配置系统

#### Fstab
用以下命令生成 `fstab` 文件 (用 -U 或 -L 选项设置 UUID 或卷标)：
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
在执行完以上命令后，后检查一下生成的 `/mnt/etc/fstab` 文件是否正确。

#### Chroot
Chroot 到新安装的系统：
```
# arch-chroot /mnt
```
#### 时区
```
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
运行 `hwclock` 以生成 `/etc/adjtime`：
```
# hwclock --systohc
```
这个命令假定硬件时间已经被设置为 UTC时间。

#### 本地化
本地化的程序与库若要本地化文本，都依赖 Locale，后者明确规定地域、货币、时区日期的格式、字符排列方式和其他本地化标准等等。在下面两个文件设置：`locale.gen` 与 `locale.conf`。

`/etc/locale.gen` 是一个仅包含注释文档的文本文件。指定您需要的本地化类型，只需移除对应行前面的注释符号（＃）即可，建议选择带 UTF-8 的项：
```
# nano /etc/locale.gen

en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```
接着执行 locale-gen 以生成指定的本地化文件。
```
# locale-gen
```
创建 locale.conf 并编辑 LANG 这一 变量
警告: 不推荐在此设置任何中文 locale，会导致 TTY 乱码。

#### 网络
创建 hostname 文件:
```
/etc/hostname

myhostname
```
添加对应的信息到 hosts(5):
```
/etc/hosts

127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```
系统安装完毕后，需要再次设置网络，

## BIOS启动

### 硬盘分区

### 格式化并挂载分区

### 安装必须的软件包

### 配置系统
