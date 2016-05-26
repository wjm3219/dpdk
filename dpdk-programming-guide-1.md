## Dpdk proggramming guide 1

- date = "2015-05-13T14:12:41+08:00"


- title = "DPDK Programmer's Guide(1)"

- author = "sqh.me" - qhsong

  #### 介绍

* Roadmap  
  本文主要讲述
  	+ 软件体系结构和如何使用它，尤其是在linux环境下 
  	+ DPDK的内容、编译系统（包括可以在root下使用Makefile去编译开发工具和程序）、移植程序的准则
  	+ 软件中的优化和在一个新的开发中需要重新考虑的问题   
* 相关的出版物  
  	下列的文档中使用DPDK开发应用程序：  
  + Intel® 64 and IA-32 Architectures Software Developer’s Manual Volume 3A: System Programming Guide

1. 概述

   这部分对DPDK给了一个全局的概述.  
   DPDK主要的目标是去提供一个简单、完整的框架在数据平面的程序中快速的进行数据包处理。用户可能使用代码去理解一些已经应用的技术去部署上层的原型系统或者构建他们自己的协议栈。这些在使用DPDK的程序中的生态系统中都有可替代的产品。  
   这个框架通过环境抽象层（EAL）生成了一系列的库来对于一些特定的环境进行屏蔽，这可能是Intel 架构（32位或者64位）、linux下编译器或者一些特定的平台。这些环境通过使用makefile和configuration file来进行配置。一旦EAL被生成用户就可以使用这个库文件来创建他们自己的应用程序。在EAL之外的库，例如hash，LPM(longest prefix match)和rings库也在DPDK中提供。样例程序也对用户提供帮助，并向他们展示如何使用多种多样的DPDK的特点。  
   DPDK实现运行一个完整的包处理模型，所有的资源在调用前必须被提前分配，程序运行就像执行单元一样在逻辑CPU核心上运行。这个模型不支持任务调度并且所有的设备需要通过轮询的方式进行访问。不使用中断的主要原因是中断处理的性能开销大了。

   * 开发环境  

     DPDK项目安装需要Linux和相关的工具链，如一个或者更多的编译器、汇编器、make工具、编辑器和多种库文件来生成DPDK的组件和库。  
     指定的环境和构架的库文件生成好后，就可以用来创建用户数据层的应用程序。  
     在Linux用户空间中生成一个程序，glibc库是需要使用的。对于DPDK应用程序，编译之前需要有两个环境变量（RTE_SDK 和RTE_TARGET）被配置。下面是如何配置这个环境变量的例子：  
     ```bash
     export RTE_SDK=/home/user/DPDK
     export RTE_TARGET=x86_64-native-linuxapp-gcc
     ```
     具体参考_DPDK Getting Started Guide_ 来如何建立开发环境
     	+ EAL（环境抽象层）
     	EAL提供一个普通的应用程序接口来隐藏不同环境的差异，EAL提供的服务如下：
     	+ DPDK的加载和登陆
     	+ 支持多核和多线程执行
     	+ CPU内核指定
     	+ 系统内存分配释放
     	+ 原子锁操作
     	+ 时间引用
     	+ PCI总线访问
     	+ 跟踪和调试函数
     	+ CPU特点识别
     	+ 中断处理
     	+ 闹钟？操作
     EAL更详细的描述参见第三章

* 主要部件

    主要的部件采用一系列的库来提供高性能的包处理应用程序。
    ![主要部件](http://dpdk.org/doc/guides/_images/architecture-overview.svg)

  + 内存分配(librte_malloc)

  `librte_malloc` 库提供了一个API让系统在大页(hugepage)中分配内存，而不是从堆中分配。当在Linux用户空间环境下分配比较大量的内存时，使用系统默认的4K大小的堆内存页将会使得TLB失效的更快。

  + 环空间管理(librte_tring)

    环的结构在一个有限大小的表中提供了一个无锁的多生产者、多消费者的FIFO的API。使用无锁的队列有很多优势，如更容易去使用、适用于大量的操作并且速度快。一个环状空间被_Memory pool Manager_(librte_mempool)使用，并且也可以作为一个通用的交流机制，用于在内核之间或者在一个逻辑核上一些执行单元的；连接。

  + 内存池管理(librte_mempool)

    内存池管理负责对内存中的对象分配池进行管理。每一个池被由名字来进行识别，并且使用换空间来存储释放的对象。这提供了一些额外的服务，如每一个核对象的缓存和内存对齐（保证使得每一个对象都在所有的内存通道上都填充为相等的大小）

+ 网络包缓存管理(librte_mbuf)

    mbuf库提供了一个便利去创建和摧毁可能被DPDK应用程序用来存储消息的缓冲区。这些缓冲区使用DPDK内存池库在程序开始时被创建，并在内存池中存储。  
    这一个库提供了一个API去分配和释放mbuf，操作控制普通消息缓冲区，和携带网络包的包缓冲区。

+ 时间管理(librte_timer)

    这个库提供了对于DPDK执行单元提供了一个时间服务，也提供了函数能够异步执行的能力。他可以周期性的进行函数调用，也可以只是一次调用。它使用EAL提供的时间接口来得到准确的时间并且能作为每一个核的必要的基础被初始化。

* 以太网Poll模式驱动框架

    DPDK包含的Poll模式驱动（Poll Mode Drivers，PMDs）包括1Gb,10Gb,40Gb,以及被设计来不需要进行异步工作、基于中断信号机制的虚拟网络控制器。

* 包转发算法支持

    DPDK包含hash算法(librte_hash)和最长前缀匹配算法(LPM,librte_lpm)库来支持对应的包转发算法

* librte_net

  `librte_net`库是一个IP协议定义、一些便利的宏的集合。它的代码基于FreeBSD的IP协议栈并且包含很多协议(对于使用IP包头)、IP相关的宏、IPv4/IPv6头结构和TCP、UDP、SCTP头的结构。