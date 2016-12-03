# 实验二. 基于IIC总线的EEPROM驱动移植实验

---

## 1. 实验目的

* 熟悉平台主板外围设备EEPROM芯片AT24CXX的电路结构与工作原理;
* 掌握IIC总线设备驱动移植流程;

## 2. 实验环境

* 硬件：CBT-EMB-MIP 实验平台,PC机;
* 软件：VMware,Linux OS;

## 3. 实验内容

* 移植IIC总线EEPROM设备驱动;

## 4. 实验原理

* AT24CXX芯片简介

> AT24CXX是美国ATMEL公司的低功耗CMOS串行EEPROM，典型的型号有AT24C01A/02/04/08/16等5种，它们的存储容量分别是1024/2048/4096/8192/16384位；也就是128/256/512/1024/2048字节；使用电压级别有5V，2.7V,2.5V,1.8V。

* 平台主板外设原理图

![](/chapter4/experiment02/AT24.png)

AT24CXX的1、2、3脚是三条地址线，用于确定芯片的硬件地址（因为只有一块器件平台主板中直接接地）;第8脚和第4脚分别为正、负电源。第5脚SDA为串行数据输入/输出，数据通过这条双向I2C总线串行传送，SDA和SCL都需要和正电源间各接一个5.1K的电阻上拉。第7脚为WP写保护端，接地时允许芯片执行一般的读写操作。接电源端时不允许对器件写。

AT24CXX中带有片内地址寄存器。每写入或读出一个数据字节后，该地址寄存器自动加1，以实现对下一个存储单元的读写。所有字节均以单一操作方式读取。为降低总的写入时间，一次操作可写入多达8个字节的数据。

[AT24CXX用户手册](/pdf/AT24C08.pdf)

## 5. 实验步骤

1.配置内核

* 打开I2C支持

Location:

> │     -&gt; Device Drivers                                                                             │
>
> │       -&gt; I2C support \(I2C \[=y\]\)

* 打开杂项设备，该选项打开后，EEPROM也就打开了。

Location:

> │     -&gt; Device Drivers                                                                             │
>
> │       -&gt; Misc devices                                                                             │
>
> │         -&gt; EEPROM support

行内代码块 `code`

2.修改代码,在内核中增加AT24CXX设备配置

修改文件: arch/arm/mach-exynos/mach-tiny.c

增加如下代码:

\#ifdef CONFIG\_EEPROM\_AT24

\#include &lt;linux/i2c/at24.h&gt;

static struct at24\_platform\_data at24c02 = {

.byte\_len = SZ\_2K / 8,

.page\_size = 8,

.flags = 0,

};

\#endif

static struct i2c\_board\_info smdk4x12\_i2c\_devs0\[\] \_\_initdata = {

\#ifdef CONFIG\_EEPROM\_AT24

{

I2C\_BOARD\_INFO\("24c02", 0x50\),

.platform\_data = &at24c02,

},

\#endif

};

代码说明：AT24CXX使用8位地址，内存大小2K比特位，也就是256K字节，页大小为8字节。

3.步骤3

巴拉巴拉

4.步骤4

巴拉巴拉

