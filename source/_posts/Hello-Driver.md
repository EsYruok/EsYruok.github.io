---
title: Hello Driver
categories:
  - Windows Driver
date: 2023-03-05 11:56:17
tags:
---

编写第一个软件驱动程序。并使用vs和windbg进行调试。  <!--more-->
## 环境安装
在windows中我们使用Visual Studio + WDK来编写驱动程序。具体配置方式可以查看[官方文档](https://learn.microsoft.com/zh-cn/windows-hardware/drivers/download-the-wdk)。
## 模型选择
打开Visual Studio后选择**新建项目**，并将项目类型选择为**Driver**。可以看到WDK提供了多种驱动模型供选择。对于Software driver可以使用两种模型：KMDF与WDM（传统的Windows NT模型）。使用这两种模型无需关心PnP即插即用与电源的管理，只专注于驱动程序的主要任务即可。  
## Visual Studio使用技巧
- 提示消失时使用`Ctrl+J`重新显示
- `Alt+F12`在当前页面查看定义位置，使用`ESC`关闭
- 自动选中提示：`工具->选项->文本编辑器->C/C++->高级->主动提交成员列表->TRUE`
## 项目配置
在编译驱动前需要对项目进行一些设置  
1. Driver Settings -> Target OS Version 修改为指定的平台（Windows 10/Windows 7）
2. Driver Settings -> Target Platfornm 修改为Desktop
3. Inf2Cat -> Run Inf2Cat 修改为No
4. Driver Signing -> Sign Mode 修改为Off
## Hello Driver
新建源代码文件时一定要注意新建的是`.c`文件而不是`.cpp`文件。  
```c
//main.c
#include <ntddk.h>

void DriverUnload(PDRIVER_OBJECT pDriverObject) {
    UNREFERENCED_PARAMETER(pDriverObject);
    DbgPrint("Goodbye Driver");
}

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject, PUNICODE_STRING pRegPath) {
    //未使用的参数
    UNREFERENCED_PARAMETER(pRegPath);
    pDriverObject->DriverUnload = DriverUnload;
    
    DbgPrint("Hello Driver");   //此功能debug，release中都有用
    KdPrint(("Hello Driver"));  //此功能只在debug中有用，release中无用

    return STATUS_SUCCESS;
}
```
## 禁用签名
对驱动程序的测试出现bug容易蓝屏，所以一般要放在虚拟机或者其他机器上进行。正常情况下windows加载驱动是要验证签名的，所以我们要将测试机**禁用驱动签名**。  
### win7禁用驱动签名（临时）
1. 系统启动时按F8进入高级选项
2. 选择**禁用驱动程序签名强制**
### win10禁用驱动签名（临时）
1. win10在系统启动时找不到F8选项，需要选择**更新和安全**->**恢复**->**高级启动**->**立即重新启动**
2. 在重启的界面中选择，**疑难解答**->**高级选项**->**启动设置**->**重启**
3. 进入了和win7相同的高级选项菜单，选择**禁用驱动程序签名强制**
## 测试运行
这里使用了两个工具**INSTDRV**与**DebugView**。在使用时要注意：
1. DebugView的**监视**菜单中**监视核心**与**监视事件**要确认勾选
2. INSTDRV要以管理员权限启动
使用INSTDRV对驱动依次进行**安装**，**启动**，**停止**，**卸载**，并且将在DebugView观察到我们进行的输出**Hello Driver**与**Goodbye Driver**。  
如果仍然无法看到输出，需要对注册表进行如下修改，并**重启**。  
```bash
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Debug Print Filter]
"DEFAULT"=dword:ffffffff
```
## 驱动调试
### 虚拟机调试准备
1. 虚拟机建立串口 (使用命名管道 \\.\pipe\com_1，该端服务器，另一端是应用程序)
2. 虚拟机使用管理员cmd执行bcdedit设置DEBUG选项
- `bcdedit /dbgsettings serial baudrate:115200 debugport:1`
- `bcdedit /copy {current} /d DebugEntry` **产生一段GUID**
- `bcdedit /displayorder {current} {0b45a6bd-59df-11ed-8e39-f78cd21324a8}` **使用上一步骤中产生的GUID**
- `bcdedit /debug {0b45a6bd-59df-11ed-8e39-f78cd21324a8} ON` **使用上一步骤中产生的GUID**
3.虚拟机中安装WDK Test Target Setup x64-x64_en-us.msi（C:\Program File (x86)\Windows Kits\10\Remote）
4.重启选择DebugEntry项，再F8选择禁用签名。
### 使用VS调试
1. 选择**扩展**->**Driver**->**Test**->**Configure Devices**
	- Display name :自己取任意名字
	- Network host name : 虚拟机计算机名 (计算机->属性->计算机名)
	- 选择 Manually configure debuggers and do not provision
	- **Next**
	- Connection Type:Serial
	- Pipe ✔
	- Reconnect ✔
	- Pipe name:\\.\pipe\com_1
	- Target Port:com1
	- **Next**
如果成功Status: Configured for driver testing（注意保持网络通畅，防火墙会有影响）
2. 选择**调试**->**附加到进程** 
	- 连接类型：Windows Kernel Mode Debugger
	- 连接目标：driver_debug(刚刚创建设备的Display name)
	- **附加**
	- 出现 Waiting to reconnect...
	- 等待出现Kernel base = 0xfffff等字样表示连接成功
	- g继续执行（实际使用的是vs内置的Windbg）
此时就已经可以在源文件下断点然后加载驱动来触发断点断下并使用调试功能调试。如果刚开始断点无效可尝试暂停一下再重新g运行一次。  
### 使用Windbg调试
1. 创建windbg.exe快捷方式
2. 修改目标参数**-k con:port=\\.\pipe\com_1,baud=115200,pipe**
3. 启动虚拟机，选择Debug选项，禁用签名
4. 双击快捷方式 出现Kernel base = 0xfffff800 表示连接成功
5. 加载自己驱动的符号文件，选择**File** -> **Symbol File Path**->**Brows** 选中自己驱动的符号路径->**reload**->**确定**
6. 打开源文件 **File** -> **Open source file** -> 选中源文件
7. 在源文件F9下断点进行调试

提示：
- 前面加载符号时默认提前设置好微软的符号路径，设置方法：**File** -> **Symbol File Path**输入**srv\*d:\symbols_xp\*http://<span><span>msdl.microsoft.com/download/symbols;** 。这里的本地路径建议根据目标系统的不同而单独建立文件夹。
- 加载好符号后建议进行检查自己的pdb是否加载成功使用`lm m [驱动名（支持正则）]`或`lm`，查看自己的驱动模块后是否跟着pdb路径 `hellodriver (private pdb symbols) d:\dev\drv\hello.....`
- 加载自己的符号前需要将驱动在系统中先加载
- 驱动与符号不匹配的情况下pdb也不会加载成功，可以使用`.reload /i`命令强制加载或重新编译拷贝驱动程序
- Windbg中系统可能会出现许多与当前调试无关的垃圾输出，使用`ed nt!Kd_SXS_Mask 0`和`ed nt!Kd_FUSION_Mask 0`命令进行禁用
