---
title: Windows Driver 理解
categories:
- Windows Driver
---

重新系统的学习一下Windows Driver涉及到的基本概念。
<!-- more -->
## 什么是driver
驱动的概念是很难直接给出一个非常精准的定义，我们来一点一点的了解。  
先以最根本的意义来讲，**驱动是一种用来让操作系统和硬件设备进行通信的软件程序**。假如一个应用程序想要从设备读取数据，应用程序要调用操作系统实现的函数，操作系统调用驱动实现的函数，驱动从硬件获取到数据后再返回给操作系统进而返回给应用程序。有些硬件是根据已有的硬件标准设计的，所以驱动程序可以是硬件厂商编写，也可以是操作系统厂商编写。如下图所示：
​​![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver01.png)  
而实际上并不是所有驱动程序都是直接与硬件设备通信的。每一个`device node`（先理解成一个设备后面再说明）都有一个`Driver stack`，一般`Driver stack`当中包含了多个驱动，应用程序的一次请求有多个驱动程序参与，其中一些只是对请求进行过滤或者将请求转化成另外一种请求，不直接与硬件进行通讯，而是将请求下发到更下一层的驱动程序。  
现在我们有了进一步的认识，驱动有两种，一种是`Function driver`用来与设备进行通讯的，一种是`Filter driver`用来辅助通信的，可能是进行请求的转换，也可能是仅仅用来观察和记录请求不主动参与到通信当中。如下图所示：  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver02.png)  

前面所描述的情景只适用于直接连接在PCI总线上的设备，它们可以直接使用硬件的地址映射来进行操作。还有一些情况设备并不直接连接在PCI总线上，比如，`USB toaster`（这个词不知道应该怎么翻译正确）连接在`host bus adapter`（主机总线控制器）上，确切的说是`USB host adapter`。而`USB host adapter`连接在PCI总线上。`USB toaster`和`USB host adapter`都有自己的`Function driver`。`USB totaster Function driver`需要通过`USB host adapter Function driver`和`USB host adapter`间接的与`USB totaster`进行通信。如下图所示：  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver04.png)  

现在重新对驱动定义：**驱动是一种观察或参与到系统与设备之间通信的软件。**
而设备一词并非一定是物理设备，有些驱动程序不与任何的硬件设备相关联。比如，想要编写一个访问内核数据结构的工具，因为虚拟地址和权限的原因，需要将工具拆分成两个组件，一个在用户态作为用户的操作界面，另一个在内核态用来访问内核数据。而这种运行在内核态的驱动程序我们叫它`Software driver`，它不与任何硬件相关联。像一些系统安全研究人员最常接触的就是这种驱动。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver03.png)  
还有一种叫`Bus driver`（总线驱动）的驱动，这种在下面介绍`Device nodes`和`device stacks`当中进行说明。  

## Device nodes 和 Plug and Play device tree
Windows使用了一个树状结构来组织设备之间的关系叫做`Plug and Play device tree`或者`device tree`。树中的节点叫做`device node`,它可以代表一个物理设备，或是一个复合设备的单独功能，或者是一个软件组件。树的根节点叫做`root device node`。如下图所示：  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/devicetree01.png)  
既然是树结构则节点间含有父子关系，`device tree`中的一些节点表示总线设备，比如上图中的`PCI bus`代表物理总线，而连接在物理总线的设备则成了`PCI bus`的子节点。并且子节点表示设备也可以是总线设备，上图中`PCI Express Port`连接在`PCI bus`上，而它本身也是一个总线设备连接着一个显示适配器`Display Adapter`，同样显示适配器也连接着一个显示器设备。  
## Device stacks
通常，一个I/O请求发送到设备是多个驱动程序来进行处理，这些驱动都有一个对应的对象`driver object`相关联，并且他们是按照一定顺序排列起来的，这个序列我们称作`device stack`。每一`device node`都有它自己的`device stack`。  
`device stack`在不同情境下可能有不同的定义方式，最正式的情况下`device stack`代表的是(`driver object`,`driver`)这一对的有序序列。而某些情况下把它看作`driver object`或`driver`单独的有序序列。  
既然是栈结构那么就有栈顶和栈底。`device stack`中第一个创建的`driver object`作为栈底，最后一个创建的作为栈顶。下图中Proseware Gizmo节点的`device stack`中有三对对象，PCI Bus节点含有两对对象。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/prosewaredevicenode01.png)  
在启动时，PnP管理器(Plug and Play manager)让每一个总线设备的驱动程序去枚举连接在总线上的子设备。以下图为例，PnP manager让PCI Bus的驱动程序Pci.sys枚举连接在其总线上的子设备，为了响应请求，Pci.sys为每一个连接在其上的子设备创建了一个设备对象physical device object(PDO)。所以PDO也将是该节点device stack的第一个对象出现在栈底。在创建后device tree看起来就会像图中这个样子。
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/prosewaredevicenode04.png)  
PnP manager将device node与新创建的PDO进行关联，并且去注册表查询确认哪些驱动需要成为这个节点的device stack的一部分。一个device stack当中必须包含一个且仅一个function driver，可选择包含一个或多个filter drivers。function driver是device stack中的主要驱动程序，负责对I/O请求的处理。而filter drivers仅仅是辅助作用。每个驱动程序被加载后都将创建一个driver object并放入device stack中。filter drivers可能在function driver上方也可能在下方，在上方的叫上层过滤驱动，在下方的叫下层过滤驱动。由function driver创建的对象叫做functional device object(FDO)，被filter driver创建的对象叫filter device object(Filter DO)。现在device tree变成像下图的样子了。
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/prosewaredevicenode02.png)  
**Note** 驱动安装时是由INF文件来确认哪个驱动是function driver哪个是filter driver驱动，这个文件是由驱动开发厂商来提供。安装后这些信息会被写到注册表当中，PnP manager通过查询注册表来获取这些信息。  

## Bus drivers
前面提到过一个概念叫Bus driver，再以上面的Pci.sys为例，它扮演着两个角色。其一，它在PCI Bus中创建了FDO，也就是说Pci.sys是PCI Bus的function driver。其二，它为与PCI Bus相连的子节点创建了PDO，那么它就是字节点的Bus driver。那么我们可以定义**为device node创建PDO的驱动程序成为该节点的bus driver**。  

## User-mode device stacks
目前为止所讨论的都是内核模式下的驱动程序，有些情况下一个设备除了有内核下的驱动还有用户模式的驱动。用户模式的驱动通常是基于Windows WDF框架提供的UMDF框架。在UMDF中驱动程序就是一个dll文件，它的device object就是一个实现了IWDFDevice接口的COM对象。这种设备对象在用户模式的堆栈中称作WDF device object(WDF DO)。下图表示了一个USB-FX-2设备的用户模式和内核模式的堆栈，他们都参与到对设备的请求处理当中。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/userandkerneldevicestacks01.png)  

