---
title: 搭建ArchLinux桌面环境
tags:
  - arch
  - linux
abbrlink: '5e538680'
date: 2024-01-12 20:04:15
---
ArchLinux是一个以简洁, 高效, 用户完全控制为目标的Linux发行版, 非常适合喜欢自定义系统的朋友. 本文记录了日常所使用的ArchLinux环境的安装过程, 以及对搭建过程中遇到的问题进行记录方便以后查阅, 希望对你也能有所帮助. <!--more-->  

> 要特别感谢 [ArchTurorial](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/) 的教程, 是一份非常详细的 ArchLinux 使用教程, 对我在接触使用 Arch 的过程中提供了非常大的帮助.  

---  
我对操作系统的期望是 ArchLinux + Gnome + Wayland. 接下来的内容都是以这个为目标所使用的步骤.  
在开始安装前需要做一些准备工作.  

- 前往 [Arch 官网](https://archlinux.org/)下载最新的ISO文件
- 制作U盘启动盘 (如果是使用虚拟机安装则不需要)
- 调整主板启动模式为UEFI
- 关闭主板的安全启动选项 

## 基本系统安装
Arch 对新手不友好的原因之一是它没有类似其他发行版那样的GUI界面, 它的安装界面是一个黑黝黝的控制台, 实际上就是一个内置了一些帮助我们安装系统工具的 Archlinux 系统. 我们需要使用命令一步一步的将系统构造起来, 这也是 Arch 完全控制的魅力之一.  

> 虽然 Arch 也提供了一个安装脚本 **archinstall** 来帮助用户快速安装, 不过建议在熟悉 Arch 之后再使用这种方式.  

#### 禁用 reflector  
```sh
systemctl stop reflector.service
```

> reflector  是一个镜像源选择工具, 它可以根据需求自动获取最新的镜像源写入你的 mirrorlist 配置中.  

#### 检查是否是UEFI启动  
```sh
ls /sys/firmware/efi/efivars
```
如果是则会输出一堆文件名. 如果不是则找不到这个文件夹.  

#### 连接网络  
如果有有线网卡直接插入网线, 虚拟机调整成桥接即可使用网络. 如果使用无线网卡则使用下面的步骤连接你的 wifi.  
```sh
iwctl
device list # 列出设备名, 例如: wlan0
station wlan0 scan # 使用设备 wlan0 扫描网络
station wlan0 get-networks # 列出扫描到的网络
station wlan0 connect YOUR-WIRELESS-NAME # 连接网络, 过程中会提示输入密码
exit
```
等待几秒用`ping`测试一下网络.  

#### 同步系统时间
```
timedatectl set-ntp true # 使用网络同步时间
```

#### 转换磁盘分区格式
UEFI 要使用 GPT 格式的磁盘分区.  
```sh
lsblk # 显示所有block设备信息, 找到你想要安装的硬盘名称 例如: sda
parted /dev/sda # 使用 parted + mktable 将磁盘类型转换成gpt
(parted)mktable # 进入 parted 命令后会显示 (parted) 输入 mktable
New disk label type? gpt # 输入 gpt
quit # 退出 parted
```

#### 磁盘分区  
磁盘分区根据个人需求进行自定义, 要注意磁盘分区会清除磁盘中的全部内容.  

> 一个比较通用的方案  
> EFI分区: **/efi** 800M  
> 根目录: **/** 100G  
> home目录: **/home** 剩余全部

使用 cfdisk 进行分区, 分区时建议将 EFI 分区作为第一个分区. 其中 EFI 分区要使用 **EFI system** 类型, 其余使用 **Linux filesystem** 类型.  
```sh
cfdisk /dev/sda
fdisk -l # 检查分区情况
```

#### 格式化分区  
将ELF分区格式化为 vfat 文件系统, 其他格式化为 ext4.  
```sh
mkfs.vfat /dev/sda1 # efi
mkfs.ext4 /dev/sda2 # root
mkfs.ext4 /dev/sda3 # home
```

#### 挂载分区  
我们将分区依次以操作系统的布局挂载到 /mnt 下面, 就好比在 /mnt 下面挂载了另一个操作系统.  
```sh
mount /dev/sda2 /mnt # 这是我们新系统的根目录
mkdir /mnt/efi  # 根目录中创建 efi 目录
mount /dev/sda1 /mnt/efi # 将EFI分区挂载到 efi 目录上
mkdir /mnt/home # 根目录中创建home目录
mount /dev/sda3 /mnt/home # 将 home 分区挂载到 home 目录上
```

#### 选择镜像源  
编辑 /etc/pacman.d/mirrorlist 文件, 将下面的源添加到最前面.    
```
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```

> mirrorlist 文件是一个镜像源的列表, 其中的内容可以从 [Pacman Mirrorlist Generator](https://archlinux.org/mirrorlist/) 获得. 当你使用 pacman 安装软件时, 会从上到下依次尝试每一个源, 直到选择一个可用的源, 所以应当将速度快的服务器放在顶端.  
> 如果你还不知道用哪一个, 暂时先使用上面中科大或者清华的源放在文件的最前面以提高速度.   

#### 安装系统
```sh
pacstrap /mnt base base-devel linux linux-headers linux-firmware dhcpcd iwd bash-completion neovim git
```

> base : 基础系统工具和库  
> base-devel : 一些用来构建和编译软件使用的工具和库  
> linux : Linux内核  
> linux-headers : 内核头文件, 帮助编译内核模块和驱动  
> linux-firmware : 一些常见的硬件固件  
> dhcpcd : DHCP客户端  
> iwd : 无线网管理工具(不使用无线网可以不用)  
> bash-completion : 命令补全工具  
> neovim : 文本编辑器  
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
hwclock --systohc # 同步到硬件
```

#### 设置 Locale 信息  
locale 信息影响系统中的字符编码, 货币日期时间等信息的显示方式. 
```sh
# 设置Local信息
nvim /etc/locale.gen # 去掉 en_US.UTF-8 和 zh_CN.UTF-8 的注释
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf # 设置成英文, 我们还没安装中文字体
```

#### 设置主机名与 
```sh
nvim /etc/hostname # 直接写入自定义的名称就行 例如: archLinux
```

#### 设置 hosts 文件
```sh
nvim /etc/hosts
# 127.0.0.1   localhost  
# ::1         localhost  
# 127.0.1.1   archLinux  
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
编辑 /etc/default/grub 文件, 删除 **GRUB_CMDLINE_LINUX_DEFAULT** 一行中的 **quite** 参数, 同时将 **loglevel** 改为5, 增加 **nowatchdog**.
```sh
nvim /etc/default/grub 
grub-mkconfg -o /boot/grub/grub.cfg
```

> `GRUB_CMDLINE_LINUX_DEFAULT` 一行代表传递给Linux内核的参数. 具体想要怎么设置可以按照个人决定.  
> - quite : 抑制大多数的启动信息, 仅显示错误信息  
> - loglevel : 内核日志的详细程度, 3代表错误信息, 5代表普通但重要的信息  
> - nowatchdog : 禁用watchdog机制, watchdog 可以在系统挂起或失去响应的情况下系统重启, 但不利于排查错误.   
> - nvidia_drm.modeset=1 : 如果使用N卡, 在最后加入参数, 开启 DRM.  

想要了解一下还有什么参数可以查阅 [The kernel’s command-line parameters](https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html), 学无止境啊. 

#### 完成安装
```sh
exit # 退出chroot的环境
umount -R /mnt
reboot
```
重启后就会进入我们刚刚安装好的系统. 至此一个基础的 ArchLinux 安装完成.  

## 安装桌面环境
#### 连接网络  
又是连接网络, 这次记得先开启 DHCP 服务.  
```sh
systemctl start dhcpcd
```
如果使用无线网再开启 iwd 服务, 并使用上面安装系统时的方式连接 wifi.  
```sh
systemctl start iwd # 无线网
# iwd ...
```

#### 安装网络管理工具
不想每次连网都这么麻烦可以提前安装管理工具.  
```sh
pacman -S networkmanager
# systemctl disable iwd
# systemctl stop iwd # 使用无线网的用户先禁用 iwd
systemctl enable NetworkManager
```

#### 准备一个非 root 用户
```sh
useradd -m -G wheel -s /bin/bash username  
passwd username 
```
修改 sudoers 配置.  
```sh
EDITOR=nvim visudo # 去掉 %wheel ALL=(ALL:ALL) ALL前的注释
```
意思是wheel组的用户可以在任意主机上以任意用户用 sudo 执行任意命令

#### 安装 Gnome 
```sh
pacman -S gnome wayland # gnome-extra
systemctl enable gdm
reboot
```
安装过程比较长, 重启后就会进入登录界面了. 

#### Wayland 
gtk3,4 默认支持 Wayland.  
Qt 想要支持 Wayland 安装 qt5-wayland / qt6-wayland. 想要显式设置的话设置环境变量 QT_QPA_PLATFORM=wayland.  
Electron (>= 28) 则需要设置环境变量 ELECTRON_OZONE_PLATFORM_HINT 为 auto 或 wayland. (环境变量的优先级低于参数)  
还可以使用给应用程序添加参数或写到配置文件 ~/.config/electron-flags.conf 中.   
```
--enable-features=WaylandWindowDecorations # 解决缺少顶栏
--ozone-platform-hint=auto
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

#### 设置DNS(可选)  
编辑 /etc/resolv.conf 文件. 可选是因为有些人的网络环境访问这几个 DNS 比较慢.    
```
nameserver 8.8.8.8
nameserver 2001:4860:4860::8888
nameserver 8.8.4.4
nameserver 2001:4860:4860::8844
```
要注意,  resolv.conf  文件开机时会被  NetworkManager  服务修改, 修改文件属性 `sudo chattr +i /etc/resolv.conf` 避免被修改.  

#### 声音固件  
```sh
pacman -S sof-firmware alsa-firmware alsa-ucm-conf
```

#### 显卡驱动
```sh
sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel # intel 显卡
sudo pacman -S nvidia nvidia-settings lib32-nvidia-utils # nvidia 显卡
yay -S optimus-manager optimus-manager-qt # 切换显卡
sudo systemctl enable optimus-manager
sudo pacman -S nvidia-prime # 动态切换, 在想要使用独显的程序前加 prime-run 前缀
```

#### 中文字体
```sh
sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```

#### 浏览器
```sh
sudo pacman -S firefox
```

#### yay 
Arch User repository Arch特色仓库. 使用方法跟 pacman 一样.   
```sh
pacman -Sy --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
也可以使用 paur
```sh
sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```

#### 中文输入法
```sh 
sudo pacman -S fcitx5 fcitx5-im 
sudo pacman -S fcitx5-chinese-addons
# sudo pacman -S fcitx5-rime # 如果想使用 Rime
yay -S fcitx5-input-support
```
添加环境变量  
```
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```
安装 Gnome 插件  Input Method Pannel  支持. 这个稍后在后面安装主题时有介绍安装插件的方法.  
打开  fcitx5-configuration, 点击  Run Fcitx5, 从右侧找到输入法添加到左侧确认即可.  

## 桌面美化
准备了两套主题, [WhiteSur-gtk-theme](https://github.com/vinceliuice/WhiteSur-gtk-theme) 和 [Orchis-theme](https://github.com/vinceliuice/Orchis-theme).

#### 如何安装插件  
1. 安装 gnome-browser-connector  
    ```
    sudo pacman -S gnome-browser-connector
    ```
2. 使用 firefox 打开 Gnome 插件地址 extensions.gnome.org 首次页面中会提示安装浏览器插件, 点击安装.  
3. 搜索想要安装的插件, 进入插件详情页面点击 `OFF/NO` 按钮即可控制开关, 如果未安装过则会出现安装提示.    

#### 安装 Tweaks  
 tweaks  是一个帮助我们管理 Gnome 桌面环境的软件, 它可以方便的管理主题, 字体, 窗口样式等设置. 如果你安装了 gnome-extra 可能已经拥有了它.  
```sh
sudo pacman -S gnome-tweaks
```

#### WhiteSur  
需要三个插件支持.  
- user-themes  
- dash-to-dock  
- blur-my-shell   

三个 Git 仓库, 根据 Readme 的说明使用安装脚本安装即可.  
- WhiteSur-gtk-theme : https://github.com/vinceliuice/WhiteSur-gtk-theme
- WhiteSur-icon-theme : https://github.com/vinceliuice/WhiteSur-icon-theme
- WhiteSur-wallpapers : https://github.com/vinceliuice/WhiteSur-wallpapers  

以下记录的是我使用的安装过程, 建议每位同学都去阅读 Readme 使用一个合适自己的配置.  
```sh
sudo pacman -S nano # 安装firefox主题时需要使用这个编辑器.  
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme?tab=readme-ov-file
cd WhiteSur-gtk-theme
./install.sh -l # 默认是 dark 明亮使用 -l -c Light
./install.sh -t all 
./install.sh -N glassy 
./tweaks.sh -f alt # firefox
./tweaks.sh -e # 看不懂内容直接保存
# ./tweaks.sh -f -r 删除 FireFox 主题
sudo ./tweaks.sh -g  # gdm

# icon
git clone https://github.com/vinceliuice/WhiteSur-icon-theme
cd WhiteSur-icon-theme
./install.sh

# wallpapers
git clone https://github.com/vinceliuice/WhiteSur-wallpapers
cd WhiteSur-wallpapers
sudo ./install-gnome-backgrounds.sh
```

#### Orchis
必要条件:  

- gnome-themes-extra  
- gtk-engine-murrine  
- sassc

```sh
sudo pacman -S gnome-themes-extra gtk-engine-murrine sassc
```
安装主题:  
```sh
git clone https://github.com/vinceliuice/Orchis-theme.git
cd Orchis-theme
./install.sh
./install.sh -t all 
./install.sh -l # 默认 light 黑暗主题 -c dark -l 例如暗紫色 -c dark -t purple -l
```
Flatpak  

- gtk3.0 的用 stylepak
    ```sh
    sudo pacman -S ostree appstream-glib
    git clone https://github.com/refi64/stylepak.git
    cd stylepak
    ./stylepak install-system # ./stylepak install-user
    ```
- gtk4.0
    ```sh
    flatpak override --filesystem=xdg-config/gtk-4.0
    # flatpak override --user --filesystem=xdg-config/gtk-4.0
    ```

#### 主题设置  
打开 tweaks  ->  
选择 **Apperance**, 将 Icons, Shell, Legacy Applications  选择为你想要的主题.  
选择 **Windows**, 在 Titlebar Buttons 中将 Maximize, Minimize  选中,  Placement  调整为 Left.  

#### 推荐插件  
- Input Method Pannel  
    输入法需要使用这个插件来显示候选面板
- Dash to dock  
    dock栏
- Blur my shell  
    毛玻璃效果插件
- Coverflow Alt-tab  
    窗口切换动画
- Clipboard Indicator  
    剪切板记录  
- Compiz alike magic lamp effect  
    窗口最小化动画效果
- Compiz windows effect  
    窗口移动动画效果
- AppIndicator and KStatusNotifierItem Support  
    程序托盘图标
- Removable Drive Menu  
    移动磁盘的托盘图标
- Extension List  
    一个插件管理的小工具  
- Apps Menu  
    左上角显示一个Apps菜单
- Place Status Indicator  
    左上角一个Place菜单

## 遇到的问题
#### VmwareTools  
Vmware 想要共享剪切板和调整系统桌面大小需要安装 `open-vm-tools`, 并且记得将服务 `vmtoolsd` 和 `vmware-vmblock-fuse` 设置 enable, 还需要安装 `gtkmm3`.    
```
sudo pacman -S open-vm-tools
sudo systemctl enable vmtoolsd vmware-vmblock-fuse
sudo pacman -S gtkmm3
```

#### 声音问题  
(VMware 环境中)安装 Gnome 时使用了 pipewire, 但是声音会爆裂断断续续的. 不会调试具体原因查阅资料也没弄明白. 所以先选择安装 pulseaudio 代替.  
```
sudo pacman -R pulse-native-provider # 与 pulseaudio 冲突的包有依赖, 先删除
sudo pacman -S alsa-utils pulseaudio pavucontol
systemctl --user enable pulseaudio.service pulseaudio.socket 
```

> 安装完成后声音是正常了, 但是重启后发生一种现象就是, 静音播放视频可以播放, 一旦开了声音视频就卡死了. 查阅资料找到一个方案. 修改配置文件 `/etc/pulse/default.pa`, 注释行 `load-module module-suspend-on-idle`.    

## 结尾
&emsp;&emsp;一个桌面环境就安装完了, 在你熟悉了的情况下依靠命令行也能很快的完成. 要特别记得Arch是滚动更新, 每天记得`pacman -Syyu`, 不然太久不更新系统滚挂的风险很高. 另外如果遇到系统进不去了的情况可以使用前面安装基本操作系统中挂载使用`arch-chroot`的方法进入系统进行调整.  
&emsp;&emsp;我这里只是罗列了一个很基本的环境, 一些像蓝牙,显卡,魔法等问题因为我还没有使用就暂时还没进行记录, [ArchTurorial](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/) 中对这些问题都有一个很好的支持, 需要的同学可以直接跳转过去进行查阅.  
