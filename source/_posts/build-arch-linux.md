---
title: 搭建ArchLinux桌面环境
categories:
  - linux
abbrlink: '5e538680'
date: 2024-01-12 20:04:15
---
ArchLinux是一个以简洁, 高效, 用户完全控制为目标的Linux发行版, 非常适合喜欢自定义系统的朋友. 本文记录了日常所使用的ArchLinux环境的安装过程, 以及对搭建过程中遇到的问题进行记录方便以后查阅, 希望对你也能有所帮助. <!--more-->  

> 要特别感谢 [ArchTurorial](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/) 的教程, 是一份非常详细的 ArchLinux 安装教程, 对我在接触使用 Arch 的过程中提供了非常大的帮助.  

---  
本文所搭建的环境是 ArchLinux + Gnome + Wayland + Nvidia. 接下来的内容都是以这个为目标所使用的步骤.  
在开始安装前的一些准备工作.  

- 前往 [Arch 官网](https://archlinux.org/)下载最新的ISO文件
- 制作U盘启动盘 (如果是使用虚拟机安装则不需要)  
    - windows 下建议使用 rufus  
    - linux 下使用命令 `sudo dd bs=4M if=/path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync`
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

> Note: 如果你正在做双系统, 挑过这一步.  

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

> Note: 如果你在做双系统, 注意不要格式化 efi 分区.  

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

也可以借助 reflector 工具.  
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
编辑 /etc/locale.gen 去掉 **en_US.UTF-8** 和 **zh_CN.UTF-8** 的注释. 然后执行:  
```sh
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf # 设置成英文, 我们还没安装中文字体
```

#### 设置主机名与 
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
编辑 /etc/default/grub 文件, 删除 **GRUB_CMDLINE_LINUX_DEFAULT** 一行中的 **quite** 参数, 同时将 **loglevel** 改为5, 增加 **nowatchdog nvidia_drm.modeset=1**.
```sh
grub-mkconfig -o /boot/grub/grub.cfg
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
进入刚刚安装好的基本 Arch 我们来进行一些配置以及桌面的安装.  

#### 连接网络  
先开启 DHCP 服务.  
```sh
systemctl start dhcpcd
```
如果使用无线网再开启 iwd 服务, 并使用上面安装系统时的方式连接 wifi.  
```sh
systemctl start iwd # 无线网
iwctl
station wlan0 connect YOUR-WIRELESS-NAME
```

#### Pacman 颜色与多线程
编辑 /etc/pacman.conf 将 **Color** 与 **ParallelDownloads** 的注释去掉.  

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
pacman -S gnome wayland
systemctl enable gdm
```
安装过程比较长, 重启后就会进入登录界面了. 

> Wayland 
> - gtk3,4 默认支持 Wayland.  
> - Qt 想要支持 Wayland 安装 qt5-wayland / qt6-wayland. 想要显式设置的话设置环境变量 QT_QPA_PLATFORM=wayland.  
> - Electron (>= 28) 则需要设置环境变量 ELECTRON_OZONE_PLATFORM_HINT 为 auto 或 wayland. (环境变量的优先级低于参数)  
> 还可以使用给应用程序添加参数或写到配置文件 ~/.config/electron-flags.conf 中.   
    > ```
    > --enable-features=WaylandWindowDecorations # 解决缺少顶栏
    > --ozone-platform-hint=auto
    > ```

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

#### 显卡驱动
环境关键字: arch + Nvidia + Intel + Wayland

1. 安装显卡驱动  
    ```sh
    sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel # intel 显卡
    sudo pacman -S nvidia nvidia-utils nvidia-settings lib32-nvidia-utils # nvidia 显卡
    sudo pacman -S nvidia-prime # 动态切换, 在想要使用独显的程序前加 prime-run 前缀
    ```

2. 修改 gdm 配置文件  
    编辑 /etc/gdm/custom.conf 取消注释并修改为 `WaylandEnable=true`

3. 更新Mkinitcpin  
    编辑 /etc/mkinitcpin.conf 修改 `MODULES=()` 为:   
    ```sh
    MODULES=(nvidia nvidia_modset nvidia_uvm nvidia_drm)
    ```
    保存后执行  
    ```sh
    sudo mkinitcpin -P
    ```

4. 添加内核参数  
    nvidia_drm.modeset=1 这个我们在前面已经添加过了.  

5. 禁用 GDM udev 规则  
    ```sh
    sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules

> Wayland 下 optimus-manager 已经不好使用了, 按照上述的步骤最终只能达到 hybrid 的效果, Nv显卡通电, 想要使用需要在程序前加 prime-run. 想要看使用 Nv 运行了哪些程序可以使用 nvidia-smi  

#### 中文字体
```sh
sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```
安装完就可以去设置里将系统切换成中文的了.  

#### 浏览器
```sh
sudo pacman -S firefox
```

#### 安装网络管理工具
不想每次连网都这么麻烦可以提前安装管理工具.  
```sh
pacman -S networkmanager
systemctl disable iwd
systemctl stop iwd # 使用无线网的用户先禁用 iwd
systemctl enable NetworkManager
```

## 桌面美化
美化一般就是壁纸 + 主题 + 插件. 壁纸就不提了, 主要介绍一下插件和主题.  

#### 如何安装插件  
1. 安装 gnome-browser-connector  
    ```
    sudo pacman -S gnome-browser-connector
    ```
2. 安装浏览器插件  
    使用 firefox 打开 Gnome 插件地址 extensions.gnome.org.    
    首次打开页面中会提示 **Click here to install browser extension** 点击安装插件.  
    浏览器会弹窗提示是否允许安装插件, 选择 **Continue to Installation** 完成安装即可.  
    重启 firefox.  
3. 安装 Gnome 插件  
    进入 extensions.gnome.org 搜索想要安装的插件, 进入插件详情页面点击 **OFF/NO** 按钮即可控制插件开关, 如果该插件未安装则会出现安装提示. (安装 Dash to dock 练习一下)  

Gnome 桌面美化插件占了很大的一笔. 现在可以浏览插件网址安装你想要的插件了.  

#### 安装 Tweaks  
 tweaks  是一个帮助我们管理 Gnome 桌面环境的软件, 它可以方便的管理主题, 字体, 窗口样式等设置. 如果你安装了 gnome-extra 可能已经拥有了它.  
```sh
sudo pacman -S gnome-tweaks
```

#### 安装主题  
1. 开启 user-themes 插件  
    这是一个系统插件, 打开 Extensions 程序可以看到所有已安装的插件, 在里面点开即可.
2. 下载主题并安装  
    [Gnome-look](www.gnome-look.org)是一个很好的寻找主题的地方. 下载主题压缩包后解压到 ** ~/.themes** 或者 **/usr/share/themes** 中.  
    但是!! 强烈建议进入主题的 github 页面使用作者提供的安装方式,方便省力.  
3. 更改主题  
    使用 user-themes 或者 Tweaks 去配置自己的主题.  
    GNOME 43 之后部分程序使用了 Libadwaita (比如 Files), 这些程序目前不支持自定义主题, 如果想更改只能通过覆盖 gtk-4.0 的配置文件. 这种方式非常不灵活但也是目前唯一的方法.这里没有深研究具体覆盖哪些文件， 主题作者一般会在安装脚本中提供一个选项稳妥的帮助你完成这一目的.  

#### Orchis 
我使用的主题是 [Orchis-theme](https://github.com/vinceliuice/Orchis-theme).  
必要条件:  

- gnome-themes-extra  
- gtk-engine-murrine  
- sassc

```sh
sudo pacman -S gnome-themes-extra gtk-engine-murrine sassc
```

> Note: 这个主题与插件 Blur my shell 有冲突, 不要开启此插件.  

安装主题:  
```sh
git clone https://github.com/vinceliuice/Orchis-theme.git
cd Orchis-theme
./install.sh -t purple 
./install.sh -l -c dark -t purple 
```

#### 推荐安装插件  
- Input Method Pannel  
    输入法需要使用这个插件来显示候选面板
- Dash to dock  
    dock栏
- Coverflow Alt-tab  
    窗口切换动画
- Compiz alike magic lamp effect  
    窗口最小化动画效果
- Compiz windows effect  
    窗口移动动画效果
- AppIndicator and KStatusNotifierItem Support  
    程序托盘图标

## 遇到的问题
#### VmwareTools  
Vmware 想要共享剪切板和调整系统桌面大小需要安装 open-vm-tools, 并且记得将服务 vmtoolsd 和 vmware-vmblock-fuse 设置 enable, 并且需要安装 gtkmm3.    
```sh
sudo pacman -S open-vm-tools
sudo systemctl enable vmtoolsd vmware-vmblock-fuse
sudo pacman -S gtkmm3
```

#### 声音问题  
(VMware 环境中, 实体机器没发现有异常)安装 Gnome 时使用了 pipewire, 但是声音会爆裂断断续续的. 不会调试具体原因查阅资料也没弄明白. 所以先选择安装 pulseaudio 代替.  
```sh
sudo pacman -R pulse-native-provider # 与 pulseaudio 冲突的包有依赖, 先删除
sudo pacman -S alsa-utils pulseaudio pavucontol
systemctl --user enable pulseaudio.service pulseaudio.socket 
```

> 安装完成后声音是正常了, 但是重启后发生一种现象就是, 静音播放视频可以播放, 一旦开了声音视频就卡死了. 查阅资料找到一个方案. 修改配置文件 `/etc/pulse/default.pa`, 注释行 `load-module module-suspend-on-idle`.    

#### F区按键映射
F1 - F12 被映射成多媒体按键  
```sh
echo 0 | sudo tee /sys/module/hid_apple/parameters/fnmode
```
看看是否恢复正常, 如果正常了就将他写入配置文件.  
```sh
echo "options hid_apple fnmode=0" | sudo tee -a /etc/modprobe.d/hid_apple.conf
```

## 结尾
&emsp;&emsp;一个桌面环境就安装完了, 在你熟悉了的情况下依靠命令行也能很快的完成. 要特别记得Arch是滚动更新, 每天记得`pacman -Syyu`, 不然太久不更新系统滚挂的风险很高. 另外如果遇到系统进不去了的情况可以使用前面安装基本操作系统中挂载使用`arch-chroot`的方法进入系统进行调整.  
&emsp;&emsp;我这里只是罗列了一个很基本的环境, 一些像蓝牙,显卡,魔法等问题因为我还没有使用就暂时还没进行记录, [ArchTurorial](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/) 中对这些问题都有一个很好的支持, 需要的同学可以直接跳转过去进行查阅.  

## 常用软件  
这部分不重要, 所以放在最后给自己做个备忘.  

#### Nerd 字体
这个字体在配置 nvim zsh 等很多地方都能用到.  
```sh
sudo pacman -S ttf-jetbrains-mono-nerd
```
手动安装方法:  

1. 在 `usr/share/fonts` 或 `.locale/share/fonts` 下建立属于你字体的目录.  
2. 将字体文件全部解压到文件夹中.  
3. 修改所有 ttf 文件权限为 644.  
4. 刷新字体缓存 `sudo fc-cache -fv`

#### fcitx5  
中文输入法.  
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
输入法候选面板需要 Gnome 插件 Input Method Pannel 支持.  
打开  fcitx5-configuration, 点击  Run Fcitx5, 从右侧找到输入法添加到左侧确认即可.  

#### Dialect 
这是一款翻译软件.  
```sh
yay -S dialect
```

#### 猫咪魔法  
```sh
yay -S clash-verge-rev-bin
```

#### Openvpn  
```sh
sudo pacman -S openvpn
```
为了支持旧版本的加密算法, 需要在 .ovpn 配置文件中增加以下配置.  
```
data-ciphers BF-CBC
data-ciphers-fallback BF-CBC
providers legacy default
```
使用时执行 `sudo openvpn --config xxxx.ovpn`.  

#### Remmina
```sh 
sudo pacman -S remmina freerdp
```

#### vscode  
```sh
sudo pacman -S code 
yay -S code-oss-marketplace
```

#### zsh  
zsh 需要 noto 字体先装一下.  
```sh
sudo pacman -S zsh zsh-completions
```
执行 `zsh` 运行安装向导. 也可以手动执行, 在 zsh 下执行 (没什么用直接关了就行, 后面还有 oh-my-zsh)  
```sh
autoload -Uz zsh-newuser-install
zsh-newuser-install -f
```
设置 zsh 为默认 shell.  
```sh
chsh -l # list all shell 
chsh -s /usr/bin/zsh # change default shell to zsh
```
安装 oh-my-zsh.  
```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
设置插件, 编辑 ~/.zshrc 
```
plugins = (git z sudo web-search)
```
安装 powerlevel10k 主题. 这是 Arch 的安装方法, powerlevel10k 的仓库有详细的各种安装方式.    
```sh
yay -S zsh-theme-powerlevel10k-git
```
配置进配置文件
```sh
echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```
然后使用 `exec zsh` 来重启 zsh, 应该会自动执行 powerlevel10k 的配置向导, 也可以执行 `p10k configure` 来手动执行向导.  

#### Sunlogin
```sh
yay -S sunloginclient
sudo systemctl start runsunloginclient.service
sudo systemctl enable runsunloginclient.service # 如果不想让服务一直在后台运行每次使用前记得 start
```

#### Waydroid  
要使用 Waydroid 必须在 Wayland 下, 使用 `echo $XDG_SESSION_TYPE` 来检查. 并且需要使用 binder 模块.  
```sh
yay -S binder_linux-dkms
sudo modprobe binder_linux
su
echo "binder_linux" >> /etc/modules-load.d/binder.conf
```
安装waydroid  
```sh
yay -S waydroid
sudo waydroid init
sudo systemctl start waydroid-container
sudo systemctl enable waydroid-container
```
一些常用命令   
```sh
waydroid app install xxx.apk # 安装 apk

waydroid prop set persist.waydroid.multi_windows true # 设置多窗口模式，记得重启服务

sudo systemctl status waydroid-container # 排障可能用到的命令
sudo waydroid logcat
waydroid log
```
删除 Waydroid  
```sh
waydroid session stop
sudo systemctl stop waydroid-container
yay -Rsn waydroid
sudo rm -rf /var/lib/waydroid ~/.local/share/application/*aydroid* ~/.local/share/waydroid
```
为了精简，只涉及到够用的步骤，还有关于Arm转译，Google注册等内容可进一步查看文档 [Waydroid](https://wiki.archlinux.org/title/Waydroid).  
再推荐一篇博客 [WayDroid教学](https://ivonblog.com/posts/archlinux-waydroid)


