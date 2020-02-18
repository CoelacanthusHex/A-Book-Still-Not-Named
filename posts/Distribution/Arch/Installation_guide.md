# Arch 安装指南 

First Post: 2020-2-17 By Coelacanthus
Modify 0x01: 2020-2-18 UEFI by Coelacanthus & BIOS by A563748846

本指南用于使用 LiveCD 的安装方法

## WARNING 

所有 2019-10-06 之后未更新的教程都会导致安装失败  
原因参见[`base` 元包替代了同名的包组并且要求安装，需要手动干预升级](https://www.archlinuxcn.org/base-group-replaced-by-mandatory-base-package-manual-intervention-required/)

## 1.安装前的准备

一般使用u盘作为安装介质

从archlinux官网或各镜像站下载iso镜像  
请务必检验镜像文件完整性与签名

检验方法：待填坑……

用软件写入u盘（win下推荐rufus，linux下使用dd即可）

## 2.启动 LiveCD

~~这个都不会你安装什么arch~~

------------------------以上都是废话--------------------------

![boot.png](https://i.loli.net/2020/02/18/gIdlMBNkQc1yhfF.png)

看到这个就说明你成功的进入了live环境

此时选择第一个选项正式进入

## 3.配置网络环境

强烈建议使用有线网络

### 3.1.有线网络

家庭网络一般是动态ip.  
使用指令
```
# dhcpcd
```
来自动配置ip连接网络

### 3.2.无线网络

使用 `wifi-menu` ，该软件为TUI软件，根据提示操作即可

**Note: 不支持同时有账号和密码的企业验证**

### 3.3.检验网络连接

```
# ping baidu.com
```
看到有返回值就说明网络没问题

此时按下

<kdd>CTRL</kbd>+<kbd>c</kbd>

## 4.更新系统时间

启用ntp以确保系统时间是准确的：
```
# timedatectl set-ntp true
```

## 5.选择镜像

>	文件 /etc/pacman.d/mirrorlist 定义了软件包会从哪个 镜像源 下载。在 LiveCD 启动的系统上，所有的镜像都被启用，并且在镜像被制作时，我们已经通过他们的同步情况和速度排序。  
>	在列表中越前的镜像在下载软件包时有越高的优先权。你可以相应的修改文件 /etc/pacman.d/mirrorlist，并将地理位置最近的镜像源挪到文件的头部，同时你也应该考虑一些其他标准。

以上摘自 Arch Wiki  

在中国大陆，建议选择TUNA源或者USTC源

## 6.验证启动模式

如果以在 UEFI 主板上启用 UEFI 模式，Archiso 将会使用 systemd-boot 来 启动 Arch Linux。可以列出 efivars 目录以验证启动模式：
```
# ls /sys/firmware/efi/efivars
```
如果目录不存在，系统可能以 BIOS 或 CSM 模式启动，请跳转至[#BIOS启动](#8BIOS启动)

## 7.UEFI启动

### 7.1.硬盘分区

参见[Click](https://wiki.archlinux.org/index.php/Parted_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

有空补写一篇……

### 7.2.格式化并挂载分区

使用mkfs格式化对应分区，不了解各种文件系统的区别的话，请无脑ext4
```
 # mkfs.ext4 /dev/sdXY
```

这是Arch对文件系统的支持情况[Click](https://wiki.archlinux.org/index.php/File_systems_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

如果您创建了交换分区，使用 mkswap 将其初始化，使用 swapon 将其挂载：
```
 # mkswap /dev/sdXY
 # swapon /dev/sdXY
```

将根分区挂载到 /mnt，例如：
```
# mount /dev/sdX1 /mnt
```
创建其他剩余的挂载点（比如 /mnt/boot）并挂载其相应的分区。

### 7.3.安装必须的软件包

```
pacstrap /mnt base linux linux-firmware dhclient dialog diffutils wireless_tools wpa_supplicant nano vim sudo man-db man-pages netctl
```
其中，`linux` 包可替换为其他内核，如：  
*	linux: 应用了一些补丁的官方内核，最常用
*	linux-hardened: 一个注重安全性的Linux内核，应用了一组强化补丁来减轻内核和用户空间的攻击。
*	linux-lts: 长期支持内核
*	linux-zen: 打了一系列优化补丁的内核

### 7.4.配置系统

#### 7.4.1.Fstab
用以下命令生成 `fstab` 文件 (用 -U 或 -L 选项设置 UUID 或卷标)：
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
在执行完以上命令后，后检查一下生成的 `/mnt/etc/fstab` 文件是否正确。

#### 7.4.2.Chroot
Chroot 到新安装的系统：
```
# arch-chroot /mnt
```
#### 7.4.3.时区
```
# ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
运行 `hwclock` 以生成 `/etc/adjtime`：
```
# hwclock --systohc
```
这个命令假定硬件时间已经被设置为 UTC时间。

#### 7.4.4本地化
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

#### 7.4.5.网络

以下的myhostname替换为你想要使用的hostname

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

#### 7.4.6.添加用户与设置sudo
```
useradd -m -G wheel [你想使用的用户名]
EDITOR=nano visudo
```
取消82行%wheel前的#，
保存并退出

使用`passwd [用户名]`给这个用户设置密码,使用`passwd root`给root设置密码。

#### 7.4.7.配置引导

本文使用`grub`
```
pacman -Syu dosfstools grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=<EFI 分区挂载点> --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
记得最后
```
exit
umount -R /mnt
reboot
```
如要继续安装图形界面，请跳转至[#图形界面安装](#9图形界面安装)
如要使用CN社区源与AUR，请跳转至[#CN社区源与AUR](#10CN社区源与AUR)

## 8.BIOS启动

### 硬盘分区

下面对硬盘进行操作仅适用于新硬盘

使用 `fdisk -l` 来查看硬盘

使用`parted /dev/sdX` 来对硬盘进行格式的更改（x为你的硬盘号）

![fdisk.png](https://i.loli.net/2020/02/18/DoAdJHxFTQ52EMt.png)

此时进入 parted 环境

使用`mklabel`指令来新建分区表

随后parted询问您分区表格式

我们输入msdos

![msdos.png](https://i.loli.net/2020/02/18/SZkJ9a4Qq6hofFB.png)

最后quit退出

使用cfdisk来分区

这里稍微有一点图形界面的意思但是前路漫长

1.  先new一个大的分区给我们用

2.  在new一个小分区作为swap（可有可无）

3.  将小分区的type改为如图

4.  ![type.png](https://i.loli.net/2020/02/18/jouQeDyts79X24K.png)

5.  成果如图

6.  ![type2.png](https://i.loli.net/2020/02/18/jAYd5tuc6Nvm9FH.png)

7.  把大分区的bootable打开

8.  选择write

9.  输入yes来确定

10. 选择quit退出

接下来你们愉快的有图形界面的部分就过去了

### 格式化并挂载分区

接下来格式化分区
```
mkfs.ext4 /dev/sdXn（sdXn选择大分区）、
```
再格式化swap如果有
```
mkswap /dev/sdX2（小分区）
```
再打开swap如果有
```
swapon /dev/sdX2 （小分区）
```
到此我们对硬盘的操作完成

接下来挂载分区
```
mount /dev/sdXn /mnt（sdXn为大分区）
```
### 安装必须的软件包

接下来进行linux的安装
```
pacstrap /mnt base linux linux-firmware base-devel
```
来安装linux及其拓展

### 配置系统

接下来配置我们的系统

使用 `genfstab -U /mnt >> /mnt/etc/fstab` 来配置

官方强烈建议我们检查fatab是否生成正确不过向我这种md5都不校验的人是不会去管那个的

能走到这一步的人以及成功了大半

加油！！！！！！！！！！！

接下来就让我们进入chroot环境

警告进入chroot后任何的失误都会影响你的系统本体

不过也就是多练习两遍的事

使用 `arch-chroot /mnt` 进入chroot环境

接下来开始设置时区

使用 `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime` 来设置

以上海为例
```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
运行 hwclock(8) 以生成 /etc/adjtime
```
hwclock –systohc
```
下一步我们进行本地化

首先运行 `pacman -S vim` 来安装一个我们熟悉的vim

这里有涉及到一点点vim所有我再废话两句吧

1.  只有一条按下 x 剪切单个字符

2.  按下I 进行插入（说白了就是编辑以后会用到）

3.  没了，多学习vim以后还会有用的

运行 `vim /etc/locale.gen` 打开有关本地化的文件

用我们之前学会的vim操作找到
```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```
并用x移除他们前面的 `#`

紧接着运行 `locale-gen` 以生成 locale 讯息

运行 `vim /etc/locale.conf` 以创建新的文件

按下i编辑

输入
```
LANG=en_US.UTF-8
```
（注意大小写）

这就添加了英语，官方不建议在此时添加中文防止乱码

接下来运行 `password root` 来更改root的密码输入密码的时候不会显示放心输入就好

引导程序配置（以grub为例）

这时Linux安装的最后一步也是决定成败的一步

首先运行 `pacman -S grub` 来安装grub作为引导

运行
```
grub-install --target=i386-pc /dev/sdX （其中sdX为你系统所在的硬盘后面没有123。。。）
```
随后生成主配置文件
```
grub-mkconfig -o /boot/grub/grub.cfg
```
如果是官方的话该叫你重启了

但是我的电脑重启了是连不上网的

所以再装一个软件 这个软件以后在安装deepin的桌面环境会有用
```
pacman -S networkmanager
```
此时就可以放心重启了

输入 exit 或按 Ctrl+d 退出 chroot 环境。

可选用 `umount -R /mnt` 手动卸载被挂载的分区：这有助于发现任何「繁忙」的分区，并通过 fuser(1) 查找原因。

最后，通过执行 reboot 重启系统，systemd 将自动卸载仍然挂载的任何分区。不要忘记移除安装介质，然后使用 root
帐户登录到新系统。

再次进入系统后运行
```
# systemctl start NetworkManager
```
（注意大小写）

之后 `ping www.bing.com` 试试网络


## 9.图形界面安装

### 9.1.安装中文字体

推荐noto字体
```
pacman -Syu noto-fonts-cjk
```

### 9.2.Gnome

### 9.3.KDE
建议最小安装
```
pacman -Syu plasma kdebase sddm networkmanager
systemctl enable sddm
systemctl enable NetworkManager
```
没有强迫症的同学可以无脑
```
pacman -Syu kde-applications-meta sddm networkmanager
systemctl enable sddm
systemctl enable NetworkManager
```

### 9.4.Xfce

### 9.5.I3

## 10.CN社区源与AUR

重启并登入系统后

### 10.1.CN社区源

打开你的`/etc/pacman.conf`，然后在后面添上两行代码。
```
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```
然后
```
sudo pacman -Syu archlinuxcn-keyring
sudo pacman -Syu archlinuxcn-mirrorlist-git
```
并将`/etc/pacman.conf`中的`Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch`改为`Include = /etc/pacman.d/archlinuxcn-mirrorlist`  
在`/etc/pacman.d/archlinuxcn-mirrorlist`中取消注释你想使用的镜像

### 10.2.AUR 

Note: 应先添加CN社区源
```
sudo pacman -Syu yay base-devel
yay --aururl "https://aur.tuna.tsinghua.edu.cn" --save
```
