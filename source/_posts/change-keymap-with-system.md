---
title: Linux系统中修改键盘映射
categories: linux
abbrlink: 9f7c0abf
date: 2024-01-30 16:16:27
---

笔记本键盘的键位一般都是通用键位, 一些习惯特殊配列或特殊键位的同学在使用笔记本键盘时都会很苦恼, 通常会需要修改键位. 如果笔记本驱动能够使用最好, 可是支持 Linux 下可用的驱动程序很少, 我们还可以通过修改系统对键盘的映射来调整键位.  <!--more-->  

------

1. 找到内置键盘的厂商，型号，版本标识
使用 `ls /proc/bus/input/devices`, 通过 Name找到自己的键盘设备信息，比如：  
```	
I: Bus=0003 Vendor=1532 Product=0233 Version=0111
N: Name="Razer Razer Blade Keyboard"
P: Phys=usb-0000:00:14.0-8/input1
S: Sysfs=/devices/pci0000:00/0000:00:14.0/usb1/1-8/1-8:1.1/0003:1532:0233.0005/input/input11
U: Uniq=
H: Handlers=sysrq kbd event11
B: PROP=0
B: EV=10001f
B: KEY=33eff 0 0 483ffff17aff32d bfd4444600000000 1 130c730b17c007 ffbf7bfad941dfff febeffdfffefffff fffffffff
ffffffe
B: REL=1040
B: ABS=100000000
B: MSC=10
```
记住第一行`I: Bus=0003 Vendor=1532 Product=0233 Version=0111` 后面需要的信息.  
2. 找到要改的键的scancode  
可以使用`evtest`工具查看scancode, 安装evtest:    
```sh
sudo pacman -S evtest
```
执行 `sudo evtest` 根据提示选择键盘设备, 按下你要修改的按键, 得到的信息类似如下:  
```
Event: time 1691079393.218622, type 4 (EV_MSC), code 4 (MSC_SCAN), value 700e0
Event: time 1691079393.218622, type 1 (EV_KEY), code 29 (KEY_LEFTCTRL), value 1
Event: time 1691079393.218622, -------------- SYN_REPORT ------------
Event: time 1691079398.169693, type 4 (EV_MSC), code 4 (MSC_SCAN), value 70039
Event: time 1691079398.169693, type 1 (EV_KEY), code 58 (KEY_CAPSLOCK), value 1
Event: time 1691079398.169693, -------------- SYN_REPORT ------------
```
可见KEY_LEFTCTRL的scancode是700e0，keycode是29；KEY_CAPSLOCK的scancode是70039，keycode是58
3. 设置hwdb  
创建或打开`/etc/udev/hwdb.d/90-custom-keyboard.hwdb`写入以下内容：
```
evdev:input:b0003v1532p0233*
  KEYBOARD_KEY_70039=leftctrl
  KEYBOARD_KEY_700e0=capslock
```
b(bus)v(vendor)p(product)要小写, 这是我们在第一步得到的信息. 然后执行`sudo systemd-hwdb update`，并重启.  
