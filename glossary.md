## IIC
I2C总线（I2C bus，Inter-IC bus）是一个双向的两线连续总线，提供集成电路（ICs）之间的通信线路。

## CAN
CAN 全称为 ControllerArea Network，即控制器局域网，是国际上应用最广泛的现场 总线之一。最
初 CAN 总线被设计作为汽车环境中的微控制器通讯，在车载各电子控制装置 ECU 之间交换信息，形
成汽车电子控制网络。比如，发动机管理系统、变速箱控制器、仪表装备、电子主干系统中均嵌入 CAN
控制装置。

## EEPROM
EEPROM，或写作E2PROM，全称电子抹除式可复写只读内存 （英语：Electrically-Erasable Programmable Read-Only Memory），是一种可以通过电子方式多次复写的半导体存储设备。相比EPROM，EEPROM不需要用紫外线照射，也不需取下，就可以用特定的电压，来抹除芯片上的信息，以便写入新的数据。

## 总线驱动
总线驱动：在linux驱动架构中，几乎不需要驱动开发人员再添加bus，因为linux内核几乎集成所有总线bus，如usb、pci、i2c等等，并且总线bus中的（与特定硬件相关的代码）已由芯片提供商编写完成。例如TI davinci平台i2c总线bus与硬件相关的代码在
内核目录/drivers/i2c/buses下的i2c-davinci.c源文件中；而三星的s3c-2440平台i2c总线bus为/drivers/i2c/buses/i2c-s3c2410.c。
 

## 设备驱动
设备驱动：与特定设备相关的就需要驱动工程师来实现了。
与总线驱动关系：如拿i2c来说，linux内核和芯片提供商已经为我们的驱动程序提供了i2c驱动的框架，以及框架底层与硬件相关的代码的实现。一般情况下，对驱动开发人员来讲，剩下的就是针对挂载在i2c两线上的i2c设备，编写具体的设备驱动了，这里的设备是指soc外部挂载的设备，并不包含集成在soc内部的设备，如i2c控制器，反而集成在soc内部设备的驱动可以理解为总线驱动。

## JNI与NDK的区别
JNI是Java调用Native机制，是Java语言自己的特性全称为 Java Native Interface，类似的还有微软.Net Framework上的p/invoke，可以让C#或Visual Basic.NET调用C/C++的API，所以说JNI和Android没有关系，在PC上开发Java的应用，如果运行在Windows平台使用 JNI是是经常的，比如说读写Windows的注册表。
NDK是Google公司推出的帮助Android开发者通过C/C++本地语言编写应用的开发包，包含了C/C++的头文件、库文件、说明文档和示例代码，我们可以理解为Windows Platform SDK一样，是纯C/C++编写的，但是Android并不支持纯C/C++编写的应用，同时NDK提供的库和函数功能很有限，仅仅处理些算法效率敏感的问题。
NDK其实多了一个把.so和.apk打包的工具，这个是很重要的。
而JNI开发并没有打包，只是把.so文件放到文件系统的特定位置。

##POSIX
POSIX是Portable Operating System Interface foIX的首字母缩写词，是一套 IEEE 和ISO标准。这个标准定义了应用程序和操作系统之间的一个口。只要保证他们的程序设计的符合 POSIX 标准，开发人员就能确信他们的程序可以和支持SIX 的操作系统互联。这样的操作系统包括大部分版本的 UNIX。POSIX 标准现在由 IEEE 的一分支机构Portable Applications Standards Committee(PASC)维护。