# Arch 安装指南 

First Post: 2020-2-17 By Coelacanthus

本指南用于使用 LiveCD 的安装方法

## WARNING 

所有 2019-10-06 之后未更新的教程都会导致安装失败  
原因参见[`base` 元包替代了同名的包组并且要求安装，需要手动干预升级](https://www.archlinuxcn.org/base-group-replaced-by-mandatory-base-package-manual-intervention-required/)

## 安装前的准备

## 启动 LiveCD

## 配置网络环境

## 更新系统时间

## 选择镜像

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
运行 `hwclock` 以生成 /etc/adjtime：
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
如果系统有一个永久的 IP 地址，请使用这个永久的 IP 地址而不是 127.0.1.1。

对新安装的系统，需要再次设置网络，请注意，目前的 base 不含有任何网络管理工具，要安装希望使用的网络管理软件。

## BIOS启动

### 硬盘分区

### 格式化并挂载分区

### 安装必须的软件包

### 配置系统
