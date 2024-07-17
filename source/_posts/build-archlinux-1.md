---
title: 搭建ArchLinux桌面环境
categories:
  - linux
abbrlink: debdcd0c
date: 2024-01-12 20:04:15
---
ArchLinux是一个以简洁, 高效, 用户完全控制为目标的Linux发行版, 非常适合喜欢自定义系统的朋友. <!--more-->  
本篇文章仅记录一个自己安装ArchLinux过程的Command List以方便以后重装时参照. 详细的学习安装过程可参考ArchTurorial.  

> 特别感谢 [ArchTurorial](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/) 的教程, 是一份非常详细的 ArchLinux 安装教程, 对我在接触使用Arch的过程中提供了非常大的帮助.  

---  
## 基本系统安装
```sh
systemctl stop reflector.service
ls /sys/firmware/efi/efivars # 检查是否是UEFI启动
iwctl # 连接无线网
# device list
# station wlan0 scan
# station wlan0 get-networks 
# station wlan0 connect YOUR-WIRELESS-NAME
# exit
timedatectl set-ntp true # 同步系统时间
lsblk # 查找硬盘名称.例如: sda
parted /dev/sda # 转换分区格式
# mktable
# gpt
# quit
cfdisk /dev/sda # 磁盘分区
# /efi 800M "EFI system"
# / 100G "Linux filesystem"
# /home 剩余全部 "Linux filesystem"
fdisk -l # 检查分区情况
mkfs.vfat /dev/sda1 # /efi
mkfs.ext4 /dev/sda2 # /
mkfs.ext4 /dev/sda3 # /home
mount /dev/sda2 /mnt
mkdir /mnt/efi 
mount /dev/sda1 /mnt/efi
mkdir /mnt/home
mount /dev/sda3 /mnt/home
vim /etc/pacman.d/mirrorlist
# Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
# Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
pacstrap /mnt base base-devel linux linux-headers linux-firmware dhcpcd iwd bash-completion neovim git
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt 
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime # 设置时区
hwclock --systohc 
vim /etc/locale.gen # 去掉en_US.UTF-8和zh_CN.UTF-8的注释
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf # 设置成英文
vim /etc/hostname # 设置主机名
vim /etc/hosts
# 127.0.0.1   localhost  
# ::1         localhost  
# 127.0.1.1   you_hostname 
passwd root # 设置root密码
pacman -S intel-ucode # Intel 微码
pacman -S amd-ucode # AMD 微码
pacman -S grub efibootmgr # 安装引导程序
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
vim /etc/default/grub # 修改内核参数 GRUB_CMDLINE_LINUX_DEFAULT="loglevel=5 nowatchdog"
grub-mkconfig -o /boot/grub/grub.cfg
exit # 退出chroot的环境
umount -R /mnt
reboot
```

## 桌面环境
```sh
systemctl start dhcpcd # 先联网
systemctl start iwd
iwctl
vim /etc/pacman.conf # 去除Color,ParallelDownloads,[multilib] 注释
pacman -Syyu
useradd -m -G wheel -s /bin/bash username # 非root账户
passwd username 
EDITOR=vim visudo # 去掉 %wheel ALL=(ALL:ALL) ALL前的注释
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress #创建4G的交换空间 大小根据需要自定
chmod 600 /swapfile
mkswap /swapfile #格式化swap文件
swapon /swapfile #启用swap文件
nvim /etc/fstab # 文件末尾添加行 /swapfile none swap defaults 0 0
pacman -S gnome wayland # Gnome桌面
systemctl enable gdm
pacman -S networkmanager # 网络管理工具
systemctl disable iwd
systemctl stop iwd
systemctl enable NetworkManager
# Wayland设置
sudo pacman -S qt5-wayland/qt6-wayland
vim ~/.config/electron-flags.conf
# --enable-features=WaylandWindowDecorations
# --ozone-platform-hint=auto
```

## 常用软件

> Note: pacman用法:  
> ```sh
> sudo pacman -S package_name     # 安装软件包
> sudo pacman -Syyu               # 升级系统 yy标记强制刷新 u标记升级动作
> sudo pacman -R package_name     # 删除软件包
> sudo pacman -Rs package_name    # 删除软件包，及其所有没有被其他已安装软件包使用的依赖包
> sudo pacman -Qdt                # 找出孤立包 Q为查询本地软件包数据库 d标记依赖包 t标记不需要的包 dt合并标记孤立包
> sudo pacman -Rs $(pacman -Qtdq) # 删除孤立软件包
> pacman -Ss package_name         # 正则查询软件包
> ```

### 基本软件
```sh
sudo pacman -S git neovim
git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si # yay
sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra # 中文字体
sudo pacman -S firefox
sudo pacman -S ttf-jetbrains-mono-nerd # nerd字体
yay -S dialect # 翻译软件
yay -S clash-verge-rev-bin
sudo pacman -S remmian freerdp # 远程桌面工具
sudo pacman -S code && yay -S code-oss-marketplace # vscode
```

### 显卡驱动
```sh
sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel # intel驱动
sudo pacman -S nvidia nvidia-utils nvidia-settings lib32-nvidia-utils # nv驱动
sudo pacman -S nvidia-prime # 动态切换工具, 在使用独显的程序命令前加prime-run
vim /etc/gdm/custom.conf # 修改 WaylandEnable=true
vim /etc/mkinitcpin.conf # 编辑 MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
sudo mkinitcpin -P
vim /etc/default/grub # 添加内核参数 nvidia_drm.modeset=1
sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules # 禁用GDM udev规则
```

> `nvidia-smi`命令可以查看nv显卡使用情况.  

### VmwareTools
```sh
sudo pacman -S open-vm-tools gtkmm3
sudo systemctl enable vmtoolsd vmware-vmblock-fuse
```

### 中文输入法
```sh
sudo pacman -S fcitx5 fcitx5-im fcitx5-chinese-addons
yay -S fcitx5-input-support
vim /etc/environment
# QT_IM_MODULE=fcitx
# XMODIFIERS=@im=fcitx
fcitx5-configuration # 打开配置工具配置
```

> Gnome需要安装`Input Method Pannel`插件支持候选面板显示.

### Openvpn Client
```sh
sudo pacman -S openvpn
# 为支持旧版本加密算法,需要在.ovpn配置文件中增加配置
# data-ciphers BF-CBC
# data-ciphers-fallback BF-CBC
# providers legacy default
sudo openvpn --config xxxx.ovpn
```

### Zsh
```sh
sudo pacman -S zsh zsh-completions
chsh -l # 查看所有shell
chsh -s /usr/bin/zsh # 改变默认shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
vim ~/.zshrc # plugins = (git z sudo web-search)
yay -S zsh-theme-powerlevel10k-git
echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >> ~/.zshrc
exec zsh # 重启zsh执行向导 或 p10k configure
```

### Sunlogin
```sh
yay -S sunloginclient
sudo systemctl start runsunloginclient.service # 使用前先启动服务
```

### Waydroid
```sh
yay -S binder_linux_dkms
sudo modprobe binder_linux
su
echo "binder_linux" >> /etc/modules-load.d/binder.conf
yay -S waydroid
sudo waydroid init
sudo systemctl start waydroid-container
sudo systemctl enable waydroid-container
waydroid session start # 手动执行一下waydroid
waydroid app install xxx.apk # 安装apk
waydroid prop set persist.waydroid.multi_windows true # 设置多窗口
# 卸载方法
waydroid session stop
sudo systemctl stop waydroid-container
yay -Rsn waydroid
sudo rm -rf /var/lib/waydroid ~/.local/share/application/*aydroid* ~/.local/share/waydroid
```

## 常见问题记录

### F区按键映射
```sh
echo 0 | sudo tee /sys/module/hid_apple/parameters/fnmode
# 如果恢复正常写入配置
echo "options hid_apple fnmode=0" | sudo tee -a /etc/modprobe.d/hid_apple.conf
```
