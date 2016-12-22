# 实验二. Android 内核移植与编译实验

----------
##  实验目的
- 掌握Android 内核裁减与定制的基本方法;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机;
* 软件：VMware Workstation + Ubuntu 12.04 + ARM-LINUX 交叉编译开发环境;

## 实验内容

* 学习Android 内核裁减定制的基本配置方法，利用Android 设备配套Android 内核进行自定义功能（如 helloword 显示）的添加。并重新编译内核源码，生成内核压缩文件uImage ，下载到Android 设备中测试;

## 实验步骤

### 复制内核源码至实验目录

* 将Android光盘\src\chapter04\experiment01下的内核源码包“linux-3.5.tar.gz”复制到Ubuntu 12.04 相应目录下并解压
```
$ sudo -s        //输入普通用户密码切换到超级用户
#tar xvf linux-3.5.tar.gz
#cd linux-3.5/  
```

### 编译内核

在宿主机端为Android 平台设备 linux-3.5内核编写简单的测试驱动（内核）程序`helloworld.c`，并修改内核目录中相关文件，添加对测试驱动程序的支持。

* 使用vim 编译器手动编写实验代码`helloworld.c`（Android光盘\src\chapter05\experiment02已经存在，可以参考）
```
# vim helloworld.c
```
`helloworld.c` 内容如下：
```
#include<linux/init.h>
#include<linux/module.h>
// 驱动程序的入口函数
static int hello_init(void)
{
printk(KERN_ALERT”#########Hello, workd !  Cyb-Bot ##############\n”);
return 0;
}
//驱动程序的出口函数
static void hello_exit(void)
{
printk(KERN_ALERT”#############Goodbye , world ########\n”);
}
module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE(“Dual  BSD/GPL”);

```

有关驱动程序的编写规范，请参考后续驱动实验内容，本实验只在编写简单的驱动（内核）程序并加入到Linux 内核目录树中，使用户熟悉编译内核的过程。该驱动程序是向终端输出相关程序信息。
编写好`helloworld.c` 后将其拷贝到内核源码的`drivers/char/`目录下。
```
#cp  helloworld.c  linux-3.5/drivers/char/
```

* 进入实验内核源码目录修改drivers/char目录下的Kconfig 文件，按照Kconfig 语法添加helloworld程序的菜单支持
```
#cd  linux-3.5/drivers/char/
#vim  Kconfig
```

在Kconfig文件中添加如下代码：
```
config  HELLO_WORLD     //驱动名称
      bool “Hello  World  Test” //内核编译是bool类型
      depends  on  ARCH_EXYNOS4  //HELLO_WORLD只有ARCH_EXYNOS4板子上有效
      default  y        //y表示默认运行
      help          //提示说明，无实际意义  
          This is a demo to test kernel experiment On  Android .

```

注意`config HELLO_MODULE` 段要与前后段有空格隔开，且`bool`、`tristate`等变量要与行开头有TAB符号位隔开。注意`Konfig` 的格式（用户可以拷贝原有内容进行修改）。

* 进入实验内核源码目录修改`driver/char/`目录下的`Makefile` 文件，按照内核中`Makefile`语法添加`helloworld`程序的编译支持
```
#vim  Makefile
```

在Makefile 中添加下面一行：
```
obj-$(CONFIG_HELLO_WORLD)     += helloworld.o
```

* 在内核源码目录运行```make menuconfig``` 配置内核对`helloword`程序的支持
* 执行```make menuconfig```（注：执行之前确保开发环境当中有`ncurses`库，执行```sudo apt-get insatll ncurses-dev```下载）
* 配置`helloword`,静态编译。

> │     -&gt; Device Drivers                                                                             │
> │       -&gt; Character devices 
> │             -&gt; Hello World Test

* 退出保存内核配置。
* 在内核源码的顶层目录下编译内核
```
#   make        
  HOSTLD  scripts/kconfig/conf
scripts/kconfig/conf -s arch/arm/Kconfig
*
* Restart config...
*
*
* Character devices
*
Virtual terminal (VT) [Y/n/?] y
  Enable character translations in console (CONSOLE_TRANSLATIONS) [Y/n/?] y
  Support for console on virtual terminal (VT_CONSOLE) [Y/n/?] y
  Support for binding and unbinding console drivers (VT_HW_CONSOLE_BINDING) [N/y/?] n
Memory device driver (DEVMEM) [Y/n/?] y
/dev/kmem virtual device support (DEVKMEM) [Y/n/?] y
Hello World Test (HELLO_WORLD) [Y/n/?] (NEW)     //输入“y”
```

初次编译内核源码，由于内核源码代码庞大，所需较长时间。编译成功后会在内核源码目录的`arch/arm/boot/`目录下生成内核压缩文件`zImage`。

* 按照Android光盘配套烧写文档将新生成的内核镜像文件zImage 烧写到Android平台设备中（Android系统如何安装详见Android光盘\IMG\Cortex-A9系统烧写说明.pdf）

新内核烧写成功后启动Android 实验平台，可以在串口终端中查看到Linux内核在加载过程中打印出来的信息，如**图5.2.1**所示（内核加载只需要2-3秒，为了方便观察现象，可以在内核加载2-3秒后关闭电源来查看打印出来的信息）

```
#########Hello , world ! Cyb-Bot !###########
```
