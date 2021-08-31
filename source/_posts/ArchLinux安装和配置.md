---
title: "Arch Linux安装&配置"
tags: ["中文","入门","配置","未完结","Linux","Arch Linux"]
date: 2021-08-31T10:56:29+08:00
draft: false  
---

# 开始折腾  
## 0. [下载官方ISO](https://archlinux.org/download/)  

## 1. 制作启动盘  
推荐使用[Rufus](https://rufus.ie/), 插上U盘,写入方式为DD, MBR改成GPT后写入  

## 2. 硬盘分区  
开始菜单右键->磁盘管理, 为Linux分出一块空闲的区域  
![分区图](./images/windows_partion.png)  
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
安装基础的包
```zsh
pacstrap /mnt base linux linux-firmware vim
```
生产fstab
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
### 3. 设置[时区](https://wiki.archlinux.org/title/Locale) 
```zsh
timedatectl set-timezone Asia/Shanghai
#同步硬件时钟
hwclock --systohc
```
设置locale  
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

### 4. hostname
```zsh
vim /etc/hostname
```
自己取个主机名

## Reference  
* [Installation guide](https://wiki.archlinux.org/title/Installation_guide)  
* [2021 Archlinux双系统安装教程（超详细）](https://zhuanlan.zhihu.com/p/138951848)  
* [Manjaro-KDE安装配置全攻略](https://zhuanlan.zhihu.com/p/114296129)  
* [fstab](https://wiki.archlinux.org/title/fstab)
