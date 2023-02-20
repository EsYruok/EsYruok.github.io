---
title: Windows Driver 理解
categories:
- Windows Driver
---

重新系统的学习一下Windows Driver的基本概念。
<!-- more -->
## 什么是driver
以最根本的意义来讲，**驱动是一种用来让操作系统和硬件设备进行通信的软件程序**。假如一个应用程序想要从设备读取数据，应用程序要调用操作系统实现的函数，而操作系统则要调用驱动实现的函数，驱动从硬件获取到数据后再返回给操作系统进而返回给应用程序。如下图所示：
​​![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver01.png)  
因为有些硬件是根据已有的硬件标准设计的，所以驱动程序可以是硬件厂商编写，也可以是操作系统厂商编写。而且并不是所有驱动程序都是直接与硬件设备通信的。抽象来看每一个设备都有一个`Driver stack`（这里不知翻译成驱动栈是否恰当所以先用原本的名称）。一般`Driver stack`当中包含了多个驱动，一般我们把在应用程序的一次请求当中，最先参与的驱动程序放在栈顶，最后参与的放在栈底。其中一些驱动程序只是对请求进行过滤，或者将请求转化成另外一种请求，它们不直接与硬件进行通讯，只是将请求下发到更下一层的驱动程序。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver02.png)  
可以先扩展出两个概念：  
- 功能驱动（Function driver）：直接与设备进行通讯的驱动。（这不完全准确）
- 过滤驱动（Filter driver）：进行辅助执行的驱动。一些过滤驱动只是用来观察或记录I/O请求，并不主动的参与到通信当中。

对`Function driver`的定义只适用于直接连接在PCI总线上的设备，它们可以直接使用硬件的地址映射来进行操作。还有一些情况设备并不直接连接在PCI总线上，比如，`USB toaster`（这个词不知道应该怎么翻译正确）连接在`host bus adapter`（主机总线控制器）上，也就是`USB host adapter`。而`USB host adapter`连接在PCI总线上。`USB toaster`和`USB host adapter`都有自己的`Function driver`。`USB totaster Function driver`需要通过`USB host adapter Function driver`和`USB host adapter`间接的与`USB totaster`进行通信。如下图所示：  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver04.png)  

现在重新修正一下驱动的概念：**驱动是一种观察或参与到系统与设备之间通信的软件。**  
可以注意到一点，前面使用的词语当中有的使用了硬件设备有的使用了设备，而这里想表达的是，其实有些驱动程序不与任何的硬件设备相关联。比如，您要编写一个访问内核数据结构的工具，因为虚拟地址的原因，需要将工具拆分成两个组件，一个在用户态作为用户的操作界面，一个在内核态用来访问内核数据。而这种运行在内核态的驱动程序我们叫它`Software driver`（软件驱动），它不与任何硬件驱动相关联。像一些系统安全研究人员最常接触的就是这种驱动。  
![](https://learn.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/images/whatisadriver03.png)  
还没有结束！还有一种叫`Bus driver`（总线驱动）的驱动，介绍的同时一同来了解一下`Device Nodes`和`Device Stacks`  

