---
title: Windows Driver 理解
categories:
  - Windows Driver
date: 2023-02-28 14:22:17
tags:
---
全面了解一下windows driver当中涉及到的基本术语和概念。  <!-- more -->
## 什么是driver
很难准确的给驱动一个全面的定义。先从最根本的意义来讲它是一个软件或者说是一个组件，是用来**让操作系统和硬件设备进行通信的软件程序**。假如一个应用程序想要从硬件设备读取数据，应用程序要调用操作系统的函数，而操作系统调用驱动的函数，驱动按照一定的规则与硬件设备进行通信获取数据再返回给操作系统进而返回给应用程序。这种驱动程序可以是硬件设备厂商来提供，也可以是操作系统厂商提供。如下图所示：
​​![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver01.png)  
## Software driver
实际上有一部分开发人员编写驱动程序并不是为了支持某一硬件设备，而是为了在内核态运行代码（如果不理解用户模式与内核模式可以看一下[这里](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)）。比如，想要编写一个访问内核数据结构的工具，因为虚拟地址的原因，需要将工具拆分成两个组件，一个在用户态作为用户的操作界面，另一个在内核态用来访问内核数据。这种驱动程序我们叫`Software driver`，它不与任何硬件设备相关联。提前提醒一下，每个驱动都会有一个设备对象`driver obect`的概念，这个概念中的设备二字并非一定是与硬件设备关联。  
## Device nodes 和 Plug and Play device tree
Windows使用了一个树状结构来组织设备之间的关系叫做`Plug and Play device tree`或者简称`device tree`。树中的节点叫做`device node`,它可以用来代表一个物理设备，或是一个复合设备的单独功能，或者是一个软件组件。树的根节点叫做`root device node`。如下图所示：  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/devicetree01.png)  
既然是树结构则节点间含有父子关系。如上图中一些节点表示总线设备（Bus），比如图中的PCI bus代表物理总线，连接在这个物理总线的设备则成了PCI bus的子节点。子节点也可以是总线设备，图中PCI Express Port作为子节点连接在PCI bus上，并且它本身也是一个总线设备连接着一个显示适配器Display Adapter，显示适配器也连接着一个显示器设备。  
## Device stacks
通常情况下一个发送到设备的I/O请求是由多个驱动程序来处理的，这些驱动都有一个对应的对象driver object（就是一个`DEVICE_OBJECT`的实例）相关联，并且它们是按照一定顺序排列起来，我们称这个有序序列为`device stack`。每一个device node都有它自己的device stack。device stack在最正式的情况下代表的是(driver object,driver)对的有序序列。而某些情景下会把它看作driver object或driver单独的有序序列。所以在其它地方看到这个名词不要对其代表的意义过于僵硬。既然是stack那么就有栈顶和栈底。device stack中将第一个创建的driver object作为栈底，最后一个创建的作为栈顶。如下图中所示，Proseware Gizmo节点的device stack中有三对对象，PCI Bus节点含有两对对象。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/prosewaredevicenode01.png)  
## I/O request packets
大多数发送到设备的I/O请求都是打包成IRP包（I/O request packets）。操作系统或驱动使用`IoCallDriver`来发送一个IRP包，IoCallDriver含有两个参数，一个指向DEVICE_OBJECT的指针和一个指向IRP结构的指针。当一个程序使用IoCallDriver时我们称程序将IRP发送到设备对象或发送到设备对象关联的驱动。而有时我们会使用传递（passes the IRP）或者转发（forwards the IRP）来代替发送（sends the IRP）这个说法。一个IRP先被device stack栈顶的驱动接收，处理完毕后继续在栈中向下传递，我们称驱动将IRP向下传递到堆栈（passes the IRP down the device stack）。  
## Function driver 与 Filter driver
我们现在知道一个硬件设备会有一个device node相关联，而device node含有一个device stack，device stack中可能含有多个驱动程序。而并不是所有驱动都是用来与设备进行通信，其中一些驱动不直接与设备进行通信，仅仅是对请求进行转换，记录，过滤等，然后将请求下发到更下一层的驱动程序。我们将负责对设备进行控制和通信的驱动叫做`Function driver`,而另一种不主动参与到通信当中起辅助作用的叫做`Filter driver`。filter drivers可能在function driver上方也可能在下方，在上方的叫上层过滤驱动，在下方的叫下层过滤驱动。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver02.png)  
## Bus drivers
在系统启动时，PnP管理器(Plug and Play manager)让每一个总线设备的驱动程序去枚举连接在总线上的子设备。以下图为例，PnP manager让PCI Bus的驱动程序Pci.sys枚举连接在其总线上的子设备，Pci.sys 为每一个连接在其上的子设备创建一个设备对象 `physical device object(PDO)`。所以PDO也将是该节点device stack的第一个对象，放在栈底。在创建PDO后device tree看起来就会像图中这个样子。这里要注意Pci.sys即和PCI Bus的FDO关联，也和子设备的PDO关联。从device node的视角来看可以说是Pci.sys即属于PCI Bus所代表的节点也属于其子节点。它扮演着两个角色。其一，它在PCI Bus中创建了FDO，也就是说Pci.sys是PCI Bus的function driver。其二，它为与PCI Bus相连的子节点创建了PDO，那么它就是子节点的`Bus driver`。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/prosewaredevicenode04.png)  
## PDO 与 FDO 与 Filter DO
PnP manager将device node与新创建的PDO进行关联，并且去注册表查询确认哪些驱动需要成为这个节点的device stack的一部分（这部分信息由安装驱动时厂家所提供的INF文件所提供）。一个device stack当中必须包含一个且仅一个function driver，可选择包含一个或多个filter drivers。每个驱动程序被加载后都将创建一个driver object并放入device stack中。由function driver创建的对象叫做functional device object(FDO)，由filter driver创建的对象叫filter device object(Filter DO)。
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/prosewaredevicenode02.png)  
## User-mode device stacks
目前为止所讨论的都是内核模式下的驱动程序，有些情况下一个设备除了有内核下的驱动还有用户模式的驱动。用户模式的驱动通常是基于Windows提供的UMDF框架。在UMDF中驱动程序就是一个dll文件，它的device object就是一个实现了IWDFDevice接口的COM对象。这种设备对象在用户模式的堆栈中称作WDF device object(WDF DO)。下图表示了一个USB-FX-2设备的用户模式和内核模式的堆栈，他们都参与到对设备的请求处理当中。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/userandkerneldevicestacks01.png)  
## Minidrivers, Miniport drivers和driver pairs
minidriver是微软所提供的一种开发模型，将原本一个驱动程序的任务划分为两部分形成一个dirver pairs，一部分处理一类设备集合中所共有的一般任务，另一部分处理特定与某一个设备的特殊任务。这样使开发人员专注在特有部分，免去对公共部分的重复操作。处理特定任务部分的驱动程序有多种名称包括miniport driver，miniclass driver，minidriver。通常微软来提供共有部分的驱动程序，硬件厂商提供特定部分的驱动程序。  
## Driver stacks
我们前面说明过，一个IRP可能会经过一个device stack当中的多个驱动。与此类似的，一个I/O请求也可能经过多个device stack。看一个例子：  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/chain01.png)  
IRP由Disk.sys所创建，它是My USB Storage Device的FDO。IRP由它向下传递给Usbstor.sys。  
Usbstor.sys即是My USB Storage的PDO也是USB Mass Storage的FDO。  
同理Usbstor.sys处理完继续将IRP向下发送给Usbhub.sys。  
当Usbhub.sys完成处理，继续向下发送给(Usbuhci.sys, Usbport.sys)驱动对。它们进行了与主机控制器的实际通信，继而主机控制器又继续与物理USB存储进行通信。我们将参与I/O请求所经过的驱动序列叫做I/O请求的`drive stack`。  
上面所描述的IRP按照一定顺序经过了四个驱动，抛开device node转而关注驱动程序会得到另外一个视角。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/driverstack01.png)  
还可以继续使用特定技术或者操作系统的特定组件进行划分。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/driverstack02.png)  
第一部分属于Volume Manager，第二部分属于操作系统的存储组件，第三部分属于操作系统的核心USB部分（core USB portion）。我们对这种框图称为技术驱动栈`technology driver stack`。  
## 参考资料
本文中提到的所有内容都是通过学习微软的官方文档自行理解所得，更详细的内容可以去原版查询。  
https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/  
