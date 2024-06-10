---
title: 搭建ArchLinux桌面环境(一) -- 基本系统安装
categories:
  - linux
abbrlink: 2a77f85f
date: 2024-01-12 20:04:15
---
ArchLinux是一个以简洁, 高效, 用户完全控制为目标的Linux发行版, 非常适合喜欢自定义系统的朋友. 文章分成了三个部分, 系统安装 + 软件安装 + 桌面美化. 本篇记录了ArchLinux基本系统以及桌面环境的安装步骤. <!--more-->   
另外两篇:  
{% post_link build-archlinux-desktop-env-2 %}  
{% post_link build-archlinux-desktop-env-3 %}  

> 要特别感谢 [ArchTurorial](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/) 的教程, 是一份非常详细的 ArchLinux 安装教程, 对我在接触使用Arch的过程中提供了非常大的帮助.  

---  
在开始安装前的准备工作:  

- 前往 [Arch 官网](https://archlinux.org/)下载最新的ISO文件
- 制作U盘启动盘 (虚拟机直接挂在ISO即可)  
    - Win 下建议使用 rufus  
    - Linux 下使用命令  
    `sudo dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync`
- 调整主板设置,启动模式为UEFI,关闭安全启动  

## 基本系统安装
Arch没有提供一个GUI的安装界面,安装过程完全使用命令一步一步进行构造.  

> 虽然Arch也提供了一个安装脚本 **archinstall** 快速安装,不过建议在熟悉 Arch 之后再使用这种方式.  

#### 检查是否是UEFI启动  
```sh
ls /sys/firmware/efi/efivars
```
如果是则会输出一堆文件名,如果不是则找不到这个文件夹.  

#### 连接网络  
插入网线或虚拟机调整成桥接即可使用网络, 如果使用无线网则使用下面的步骤连接你的wifi.  
```sh
iwctl
device list # 列出设备名, 例如: wlan0
station wlan0 scan # 使用设备 wlan0 扫描网络
station wlan0 get-networks # 列出扫描到的网络
station wlan0 connect YOUR-WIRELESS-NAME # 连接网络, 过程中会提示输入密码
exit # 退出iwctl
```
等待几秒用`ping`测试一下网络.  

> 如果熟悉了设备信息, 可以进入iwctl后直接`station wlan0 connect YOUR-WIRELESS-NAME`连接.

#### 同步系统时间
```sh
timedatectl set-ntp true
```

#### 转换磁盘分区格式
UEFI 要使用 GPT 格式的磁盘分区.

> 会擦除磁盘全部数据, 如果你正在做双系统, 跳过过这一步.  

```sh
lsblk                       # 显示所有block设备信息, 找到你想要安装的硬盘名称 例如: sda
parted /dev/sda             # 使用 parted + mktable 将磁盘类型转换成gpt
(parted)mktable             # 进入 parted 命令后会显示 (parted) 输入 mktable
New disk label type? gpt    # 输入 gpt
quit                        # 退出 parted
```

#### 磁盘分区  
磁盘分区根据个人需求进行自定义, 要注意磁盘分区会清除磁盘中的全部内容.  

> 一个比较通用的方案  
> EFI分区:800M `/efi`  
> 根目录: 100G `/`  
> home目录: 剩余全部 `/home`

使用cfdisk进行分区, 分区时建议将EFI分区作为第一个分区. 其中EFI分区要使用**EFI system**类型, 其余使用**Linux filesystem**类型.  

> 如果你正在做双系统, 不需要创建EFI分区, 你应该可以看到Win已经创建好的EFI分区.  

```sh
cfdisk /dev/sda
```
完成后使用`fdisk -l`检查以下分区情况是否正确.  

#### 格式化分区  
EFI分区格式化为vfat文件系统, 其他格式化为ext4.  

> 如果你在做双系统, 注意不要格式化efi分区与Win使用的分区.  

```sh
mkfs.vfat /dev/sda1 # efi
mkfs.ext4 /dev/sda2 # root
mkfs.ext4 /dev/sda3 # home
```

#### 挂载分区  
我们将分区依次以操作系统的布局挂载到/mnt下面, 就好比挂载了另一个操作系统.  
```sh
mount /dev/sda2 /mnt # 这是我们新系统的根分区
mkdir /mnt/efi  # 创建efi目录
mount /dev/sda1 /mnt/efi # 将EFI分区挂载到efi目录上
mkdir /mnt/home # 创建home目录
mount /dev/sda3 /mnt/home # 将home分区挂载到home目录上
```

#### 选择镜像源  
```sh
nvim /etc/pacman.d/mirrorlist
```
编辑mirrorlist文件, 将下面的源添加到最前面(如果你了解你的网络按照合适你自己的编辑).  
```
# 中科大和清华的源
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

> Linux使用镜像源时会在mirrorlist中从上到下依次尝试每一个源, 直到选择一个可用的源, 所以应当将速度快的服务器放在顶端.  

也可以借助 reflector 工具:  
```sh
pacman -S reflector # 安装reflector 
reflector --verbose -l 200 -p https --sort rate --save /etc/pacman.d/mirrorlist # 选择最快的镜像源
reflector --verbose --country 'China' -l 200 -p https --sort rate --save /etc/pacman.d/mirrorlist # 选择在中国的最快镜像源
```

#### 安装系统
```sh
pacstrap /mnt base linux... # 看下面
```

> base : 基础系统工具和库  
> base-devel : 一些用来构建和编译软件使用的工具和库  
> linux : Linux内核  
> linux-headers : 内核头文件, 帮助编译内核模块和驱动  
> linux-firmware : 一些常见的硬件固件  
> dhcpcd : DHCP客户端  
> iwd : 无线网管理工具(不使用无线网可以不用)  
> bash-completion : 命令补全工具  
> neovim : 终端文本编辑器  
> git : git 工具, 后面很多软件安装要使用到它

#### 生成 fstab 文件  
fstab 文件用于定义系统启动要自动挂在的磁盘分区与文件系统.  
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

#### Change root  
chroot 能够改变当前环境的根目录, 就好比切换到了另一个系统环境中.  
```sh
arch-chroot /mnt 
```
切换进新安装的系统环境中做一些配置. 就像我们使用 GUI 界面安装系统时设置一些内容一样.    

#### 设置时区  
```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc 
```

#### 设置 Locale 信息  
locale 信息影响系统中的字符编码, 货币日期时间等信息的显示方式.  
```sh
nvim /etc/locale.gen
```
编辑locale.gen去掉**en_US.UTF-8**和**zh_CN.UTF-8**的注释, 然后执行:  
```sh
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf # 设置成英文, 我们还没安装中文字体
```

#### 设置主机名 
```sh
nvim /etc/hostname # 直接写入自定义的名称就行 例如: you_hostname
```

#### 设置 hosts 文件
```sh
nvim /etc/hosts
# 127.0.0.1   localhost  
# ::1         localhost  
# 127.0.1.1   you_hostname  
```

#### 设置root密码  
```sh
passwd root
```

#### 安装处理器微码
```sh
pacman -S intel-ucode # Intel 
pacman -S amd-ucode # AMD
```

#### 安装引导程序
```sh
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
```
修改grub配置文件设置内核启动参数
```sh
nvim /et/cdefault/grub
```

编辑grub 文件, 修改**GRUB_CMDLINE_LINUX_DEFAULT="loglevel=5 nowatchdog nvidia_drm.modeset=1**, 保存并执行:  
```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

> - quite : 抑制大多数的启动信息, 仅显示错误信息  
> - loglevel : 内核日志的详细程度, 3代表错误信息, 5代表普通但重要的信息  
> - nowatchdog : 禁用watchdog机制, watchdog 可以在系统挂起或失去响应的情况下系统重启, 但不利于排查错误.   
> - nvidia_drm.modeset=1 : 如果使用N卡, 在最后加入参数, 开启 DRM.  

更多参数可以查阅 [The kernel’s command-line parameters](https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html).  

#### 完成安装
```sh
exit # 退出chroot的环境
umount -R /mnt
reboot
```
重启后就会进入我们刚刚安装好的系统. 一个无桌面的控制台ArchLinux安装完成.  

## 安装桌面环境
对刚装好的Arch进行一些基本的配置后进行桌面的安装.  

#### 连接网络  
先开启 DHCP 服务.  
```sh
systemctl start dhcpcd
```
如果使用无线网再开启 iwd 服务.  
```sh
systemctl start iwd
```

#### Pacman 颜色与多线程
```sh
nvim /etc/pacman.conf
```

编辑pacman.conf将**Color**与**ParallelDownloads**的注释去掉.  

#### 准备一个非 root 用户
```sh
useradd -m -G wheel -s /bin/bash username  
passwd username 
```
修改sudoers配置, 让wheel组用户能够使用sudo执行任何命令.  
```sh
EDITOR=nvim visudo # 去掉 %wheel ALL=(ALL:ALL) ALL前的注释
```

#### 安装网络管理工具
安装NetworkManager后就可以使用桌面点击右上角图标连接wifi了.  
```sh
pacman -S networkmanager
systemctl disable iwd
systemctl stop iwd
systemctl enable NetworkManager
```

#### 设置交换文件(可选)  
```sh
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress #创建4G的交换空间 大小根据需要自定
chmod 600 /swapfile
mkswap /swapfile #格式化swap文件
swapon /swapfile #启用swap文件
nvim /etc/fstab # 文件末尾添加行 /swapfile none swap defaults 0 0
```

#### 开启32位库支持(可选)  
可以安装 multilib 库中的32位程序.  
```sh
nvim /etc/pacman.conf  # 去掉 [multilib] 一节中的两行注释
pacman -Syyu
```

#### 安装 Gnome 
```sh
pacman -S gnome wayland
systemctl enable gdm
```
安装过程比较长, 重启后就会进入登录界面了.  

## 结尾
&emsp;&emsp;一个桌面环境就安装完了, 在你熟悉了的情况下依靠命令行也能很快的完成. 要特别记得Arch是滚动更新, 每天记得`pacman -Syyu`, 不然太久不更新系统滚挂的风险很高. 另外如果遇到系统进不去了的情况可以使用前面安装基本操作系统中挂载使用`arch-chroot`的方法进入系统进行调整.  
