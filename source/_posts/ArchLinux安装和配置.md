---
title: "Arch Linux安装&配置(2021)"
tags: ["中文","配置","Arch Linux"]
categories: ["Linux","入门"]
cover: https://random.imagecdn.app/376/252
top_img: https://random.imagecdn.app/1920/400
date: 2021-08-31T10:56:29+08:00
---

# 开始折腾  
## 0. [下载官方ISO](https://archlinux.org/download/)  

## 1. 制作启动盘  
推荐使用[Rufus](https://rufus.ie/), 插上U盘,写入方式为DD, MBR改成GPT后写入  

## 2. 硬盘分区  
开始菜单右键->磁盘管理, 为Linux分出一块空闲的区域  
![分区图](../../../../images/2021/windows_partion.png)  
注意EFI分区大小要在300MB左右  

## 3. 设置BIOS  
进入BIOS禁用secure boot, 将U盘启动顺序移到第一位, 保存设置后重启, 进入Arch ISO  

## 4. 联网  
查看网卡名称  
```zsh
ip a
```
依次输入命令: 
```zsh
iwctl
device list
#如无线网卡名称为wlan0，输入
station wlan0 scan
#检查扫描网络，输入
station wlan0 get-networks
#连接名为not-found的WiFi，输入
station wlan0 connect not-found
#输入密码后退出
exit
```
成功后尝试刷新pacman源  
```zsh
pacman -Syyy
#切换为国内镜像源
reflector -c China -a 6 --sort rate --save /etc/pacman.d/mirrorlist
```

## 5. 分区  
### 1. 查看硬盘情况并分区  
```zsh
lsblk
#假设要安装在nvme0n1上
cfdisk /dev/nvme0n1
#找到指定分区,选择New并分配空间大小, 选择Write并确认, 选择Quit退出
```
### 2. 检查是否成功  
```zsh
lsblk
```
### 3. 分区格式化
```zsh
#假设新分区为nvme0n1p5
mkfs.ext4 /dev/nvme0n1p5
#如果是双系统,EFI分区已存在,则跳过下面的格式化EFI
mkfs.fat -F 32 /dev/nvme0n1p1
```
### 4. 挂载分区  
将分区挂载到/mnt目录
```zsh
mount /dev/nvme0n1p5 /mnt
```
挂载EFI
```zsh
mkdir /mnt/boot
#假设EFI分区为nvme0n1p1
mount /dev/nvme0n1p1 /mnt/boot
```

## 6. 安装系统  
安装内核和基础的包
```zsh
pacstrap /mnt base linux linux-firmware vim
```
生成[fstab](https://wiki.archlinux.org/title/fstab)  
```zsh
genfstab -U /mnt >> /mnt/etc/fstab
```
检查生成的文件  
```zsh
cat /mnt/etc/fstab
``

## 7. 配置新系统(再坚持一会)  
### 1. 通过chroot切换到新系统
```zsh
arch-chroot /mnt
```
### 2. [建立swapfile或挂载swap分区](https://wiki.archlinux.org/title/Swap)(可以先跳过，进入新系统后再配置)  
### 3. 设置时区  
```zsh
timedatectl set-timezone Asia/Shanghai
#同步硬件时钟
hwclock --systohc
```
### 4. 设置[locale](https://wiki.archlinux.org/title/Locale)  
```zsh
vim /etc/locale.gen
#输入/en_US后回车, 找到UTF-8那一行 删掉前面的#
#输入/zh_CN后回车, 找到UTF-8那一行 删掉前面的#
```
生成locale  
```zsh
locale-gen
vim /etc/locale.conf
```
写入下面的内容  
```zsh
LANG=en_US.UTF-8
```

### 5. hostname  
```zsh
vim /etc/hostname
```
自己取个主机名,比如arch  
```zsh
arch
```

### 6. hosts
```zsh
vim /etc/hosts
```
写入以下内容,比如主机名为arch  
```zsh
127.0.0.1 localhost
::1 localhost
127.0.0.1 arch.localdomain  arch
```

### 7. 设置root密码  
```zsh
passwd
```

### 8. 安装启动器  
```zsh
pacman -S grub efibootmgr networkmanager network-manager-applet dialog wireless_tools wpa_supplicant os-prober mtools dosfstools ntfs-3g base-devel linux-headers reflector git sudo
```
### 9. 安装微码文件
如果CPU牌子是Intel  
```zsh
pacman -S intel-ucode
```
如果CPU牌子是AMD  
```zsh
pacman -S amd-ucode
```

### 10. 配置grub  
```zsh
vim /etc/default/grub
```
添加下面这行  
```zsh
GRUB_DISABLE_OS_PROBER=false
```
完成之后输入  
```zsh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
#生成grub.cfg
grub-mkconfig -o /boot/grub/grub.cfg
```

## 7. 退出新系统并取消挂载
```zsh
exit
umount -a
reboot
```
在下次启动前拔掉U盘  

## 8.进入系统  
### 1. 联网  
```zsh
systemctl enable --now NetworkManager
nmtui
```
### 2. 新建用户  
比如新建一个叫mir的用户  
```zsh
useradd -m -G wheel mir
passwd mir
EDITOR=vim visudo
#输入/wheel 找到#wheel ALL=(ALL) ALL这一行,删去#
```
### 3. 显卡驱动  
AMD集显驱动  
```zsh
pacman -S xf86-video-amdgpu
```
NVIDIA独显驱动  
```zsh
pacman -S nvidia nvidia-utils
```
###  4. Display Server  
```zsh
pacman -S xorg
```
### 5. 安装[Display Manager](https://wiki.archlinux.org/title/display_manager)  
Gnome:
```zsh
pacman -S gdm
```
KDE:
```zsh
pacman -S sddm
```
Xfce||DDE:
```zsh
pacman -S lightdm lightdm-gtk-greeter
```
设置开机自动启动，以gdm为例：
```zsh
systemctl enable gdm
```
### 6. 安装[Desktop Environment](https://wiki.archlinux.org/title/Desktop_environment)  
Gnome:
```zsh
pacman -S gnome
#gnome全家桶(可选)
pacman -S gnome-extra
```
KDE:
```zsh
pacman -S plasma kde-applications packagekit-qt5
```
Xfce:
```zsh
pacman -S xfce4 xfce4-goodies
```
DDE:
```zsh
pacman -S deepin deepin-extra
```
### 7. 中文字体  
```zsh
pacman -S ttf-sarasa-gothic
```
再次重启后就可以见到登陆界面  

# 选择性折腾
## 1. 双显卡切换  
方案: [optimus-manager](https://github.com/Askannz/optimus-manager)
## 2. [AUR](https://wiki.archlinux.org/title/Arch_User_Repository)  
安装[yay](https://github.com/Jguer/yay)  
```zsh
pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
## 3. zsh
```zsh
pacman -S zsh
chsh -s /usr/bin/zsh
```
安装[ohmyzsh](https://github.com/ohmyzsh/ohmyzsh)  

## 4. 拼音输入法   
安装fcitx5  
```zsh
yay -S fcitx5-im
vim ~/.pam_environment
```
内容为:  
```zsh
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=\@im=fcitx
SDL_IM_MODULE DEFAULT=fcitx
```
```zsh
yay -S fcitx5-rime
yay -S rime-cloverpinyin
vim ~/.local/share/fcitx5/rime/default.custom.yaml
```
内容为:  
```yaml
patch:
  "menu/page_size": 5
  schema_list:
    - schema: clover
```
安装中文维基百科词库:  
```zsh
yay -S fcitx5-pinyin-zhwiki-rime
```
配置主题:  
```zsh
yay -S fcitx5-material-color
```
完成后注销,打开fcitx5-configtool把rime输入法按喜好配置一番  

------
# 可能遇到的问题  
* https://github.com/2432001677/2432001677.github.io/issues/9

## Reference  
* [Installation guide](https://wiki.archlinux.org/title/Installation_guide)  
* [2021 Archlinux双系统安装教程（超详细）](https://zhuanlan.zhihu.com/p/138951848)  
* [Manjaro-KDE安装配置全攻略](https://zhuanlan.zhihu.com/p/114296129)  
