---
title: 搭建ArchLinux桌面环境 -- 桌面美化
categories: linux
abbrlink: 44a301f7
date: 2024-01-13 17:31:46
---
介绍一下Gnome桌面美化.  <!--more-->  
美化一般就是壁纸 + 主题 + 插件. 壁纸就不提了设置中可以很容易的设置, 主要介绍一下插件和主题.  

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
    进入 extensions.gnome.org 搜索想要安装的插件, 进入插件详情页面点击 **OFF/NO** 按钮即可控制插件开关, 如果该插件未安装则会出现安装提示. (安装 Dash to dock 尝试一下)  
4. 插件管理  
    可以使用插件网页, 也可以使用Gnome自带的应用程序Extensions.  

Gnome 桌面美化插件占了很大的一笔. 现在可以浏览插件网址安装你想要的插件了.  

#### Tweaks  
 tweaks  是一个帮助我们管理 Gnome 桌面环境的软件, 它可以方便的管理主题, 字体, 窗口样式等设置.  
```sh
sudo pacman -S gnome-tweaks --noconfirm
```

#### 安装主题  
1. 开启 user-themes 插件  
    这是一个系统插件, 打开 Extensions 程序可以看到所有已安装的插件, 在里面点开即可.
2. 下载主题并安装  
    [Gnome-look](www.gnome-look.org)是一个很好的寻找主题的地方. 下载主题压缩包后解压到 ** ~/.themes** 或者 **/usr/share/themes** 中.  
    但是!! 强烈建议进入主题的 github 页面使用作者提供的安装方式,方便省力.  
3. 更改主题  
    使用 user-themes 或者 Tweaks 去选择主题.  
    GNOME 43 之后部分程序使用了 Libadwaita (比如 Files), 这些程序目前不支持自定义主题, 如果想更改只能通过覆盖 gtk-4.0 的配置文件. 这种方式非常不灵活但也是目前唯一的方法. 这里没有深研究具体覆盖哪些文件，主题作者一般会在自己仓库的安装说明中介绍如何使用自身提供的安装脚本完成这一目的.  

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


