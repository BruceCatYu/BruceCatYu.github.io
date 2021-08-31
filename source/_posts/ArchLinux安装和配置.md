---
title: "Arch Linux安装&配置"
tags: ["中文","入门","配置","Linux","Arch Linux"]
date: 2021-08-31T10:56:29+08:00
draft: false  
---

## 开始折腾  
### 0. [下载官方ISO](https://archlinux.org/download/)  

### 1. 制作启动盘  
推荐使用[Rufus](https://rufus.ie/), 插上U盘,写入方式为DD, MBR改成GPT后写入  

### 2. 硬盘分区  
开始菜单右键->磁盘管理, 为Linux分出一块空闲的区域  
![分区图](./images/windows_partion.png)  
注意EFI分区大小要在300MB左右  

### 3. 设置BIOS  
进入BIOS禁用secure boot, 将U盘启动顺序移到第一位, 保存设置后重启, 进入Arch ISO  

### 4. 联网  
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

## Reference  
* [Installation guide](https://wiki.archlinux.org/title/Installation_guide)  
* [2021 Archlinux双系统安装教程（超详细）](https://zhuanlan.zhihu.com/p/138951848)  
* [Manjaro-KDE安装配置全攻略](https://zhuanlan.zhihu.com/p/114296129)  
