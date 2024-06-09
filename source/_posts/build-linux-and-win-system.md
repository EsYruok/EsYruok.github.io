---
title: 搭建Linux + Win11 双系统
categories: linux
abbrlink: a9f101ac
date: 2024-01-17 08:08:57
---
虽然当下 Linux 已经非常强大了, 但是仍然存在某些情况让你不得不使用 Windows, 使用双系统就成为了大多数人的选择. 下面介绍以下如何安装一个 Linux + Win11 的双系统环境.  <!--more-->  
我使用的是 ArchLinux + Win11 的组合. 先安装 Windows 再安装 Linux.  

## 安装 Win11  
具体步骤就不详细罗列了, 但是要注意以下两点:  

1. 使用 Win11 进行分区时会自动创建一个 EFI 分区. (记住这个分区)  
2. 在给 Win11 划分磁盘空间时不要全部划分出去, 给 Linux 预留出空间.  

## 安装 Linux 
安装步骤可以参考 {% post_link build-arch-linux %}. 安装过程中要注意以下几点:  

1. 磁盘分区时不需要再创建一个 EFI 分区, 使用 Windows 的 EFI 分区即可.  
2. 给磁盘格式化时注意不要格式化到 Windows 使用的分区.  
3. 安装操作系统过程中, 直接将 Windows 的 EFI 分区挂载到 efi/ 即可.  
4. 安装结束后 Linux 会将 Windows 的启动记录覆盖, 重启后进入 Linux 重建.  

## 重建 grub 启动项
1. 安装 os-prober
```sh
sudo pacman -S os-prober
```
2. 修改 /etc/default/grub  
取消 **GRUB_DISABLE_OS_PROBER=false** 的注释, 没有就添加.  
3. 更新 grub  
```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
在输出信息中应该可以看到找到了 Windwos 的启动项.  

重启即可在 grub 启动目录上看到 Windows.  
