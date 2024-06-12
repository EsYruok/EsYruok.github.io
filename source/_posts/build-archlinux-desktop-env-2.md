---
title: 搭建ArchLinux桌面环境(二) -- 常用软件安装
abbrlink: 2ab5f660
date: 2024-01-13 17:31:43
categories: linux
---
上篇文章我们安装了一个最基本的ArchLinux以及Gnome桌面, 本篇介绍一些常用软件的安装方法.  <!--more-->  
另外两篇:  
{% post_link build-archlinux-desktop-env-1 %}  
{% post_link build-archlinux-desktop-env-3 %}  

#### Pacman
在介绍其他软件之前, 先介绍以下Arch中最基本的包管理器pacman的使用方法.  
```sh
sudo pacman -S package_name     # 安装软件包
sudo pacman -Syyu               # 升级系统 yy标记强制刷新 u标记升级动作
sudo pacman -R package_name     # 删除软件包
sudo pacman -Rs package_name    # 删除软件包，及其所有没有被其他已安装软件包使用的依赖包
sudo pacman -Qdt                # 找出孤立包 Q为查询本地软件包数据库 d标记依赖包 t标记不需要的包 dt合并标记孤立包
sudo pacman -Rs $(pacman -Qtdq) # 删除孤立软件包
pacman -Ss package_name         # 正则查询软件包
```

#### Yay/paur
Aur即Arch User repository是Arch特色仓库. yay/paur则是包管理工具, 它可以同时使用Aur和标准仓库的资源, 使用方法与pacman 一样.   
```sh
pacman -Sy --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
paur:  
```sh
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```

#### 中文字体
```sh
sudo pacman -S noto-fonts-cjk noto-fonts-emoji noto-fonts-extra
```
安装完就可以去设置里将系统切换成中文的了.  

#### 浏览器
选择firefox就是选择最少的折腾. :>  
```sh
sudo pacman -S firefox
```

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
    MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
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

#### VmwareTools  
```sh
sudo pacman -S open-vm-tools gtkmm3 --noconfirm
sudo systemctl enable vmtoolsd vmware-vmblock-fuse
```

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

#### 输入法  
我选择了fcitx5.  
```sh 
sudo pacman -S fcitx5 fcitx5-im fcitx5-chinese-addons 
yay -S fcitx5-input-support
```
添加环境变量  
```
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```
输入法候选面板需要 Gnome 插件Input Method Pannel支持. 怎么安装插件看一下 {% post_link build-archlinux-desktop-env-3 %}    
打开fcitx5-configuration, 点击Run Fcitx5, 从右侧找到输入法添加到左侧确认即可.  

#### Dialect 
一款翻译软件.  
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

#### Zsh  

> Zsh需要使用noto字体  

```sh
sudo pacman -S zsh zsh-completions
```
执行 `zsh` 运行安装向导(没什么用直接关了就行, 后面还有oh-my-zsh), 也可以手动执行, 在 zsh 下执行  
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
设置插件, 编辑~/.zshrc修改plugins:  
```
plugins = (git z sudo web-search)
```
安装powerlevel10k主题. 这是Arch的安装方法, powerlevel10k 的仓库有详细的各种安装方式.    
```sh
yay -S zsh-theme-powerlevel10k-git
```
配置进配置文件
```sh
echo 'source /usr/share/zsh-theme-powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```
然后使用`exec zsh`来重启zsh, 应该会自动执行powerlevel10k的配置向导, 也可以执行`p10k configure`来手动执行向导, 跟随向导来配置你想要的款式.    

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

#### Wayland  
Wayland这东西毕竟比较新, 各个开发者适配的进度不一, 下面是我查阅到的一些资料留作备用, 在遇到程序对wayland兼容有问题时可以试一试.  
- gtk3/4默认支持Wayland.  
- Qt想要支持Wayland安装qt5-wayland/qt6-wayland. 想要显式设置的话设置环境变量 QT_QPA_PLATFORM=wayland.  
- Electron (>= 28) 则需要设置环境变量 ELECTRON_OZONE_PLATFORM_HINT 为 auto 或 wayland. (环境变量的优先级低于参数)
    还可以使用给应用程序添加参数或写到配置文件 ~/.config/electron-flags.conf 中.   
    ```
    --enable-features=WaylandWindowDecorations # 解决缺少顶栏
    --ozone-platform-hint=auto
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
