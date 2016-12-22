# 实验三. 基于IIC总线的EEPROM驱动移植实验

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

![](/chapter5/experiment03/AT24.png)

AT24CXX的1、2、3脚是三条地址线，用于确定芯片的硬件地址（因为只有一块器件平台主板中直接接地）;第8脚和第4脚分别为正、负电源。第5脚SDA为串行数据输入/输出，数据通过这条双向I2C总线串行传送，SDA和SCL都需要和正电源间各接一个5.1K的电阻上拉。第7脚为WP写保护端，接地时允许芯片执行一般的读写操作。接电源端时不允许对器件写。

AT24CXX中带有片内地址寄存器。每写入或读出一个数据字节后，该地址寄存器自动加1，以实现对下一个存储单元的读写。所有字节均以单一操作方式读取。为降低总的写入时间，一次操作可写入多达8个字节的数据。

[AT24CXX用户手册](/pdf/AT24C08.pdf)

## 5. 实验步骤

**1.配置内核**

* _打开I2C支持_

Location:

> │     -&gt; Device Drivers                                                                             │
>
> │       -&gt; I2C support \(I2C \[=y\]\)

* _打开杂项设备，该选项打开后，EEPROM也就打开了。_

Location:

> │     -&gt; Device Drivers                                                                             │
>
> │       -&gt; Misc devices                                                                             │
>
> │         -&gt; EEPROM support

行内代码块 `code`

**2.修改代码,在内核板级支持包中增加AT24CXX设备配置**

linux-3.5内核中已经包含at24cxx芯片的驱动源码，具体位置为：

> _include/linux/i2c/at24.h _
>
> _/drivers/misc/eeprom/at24.c    
> _

修改板级支持包文件: arch/arm/mach-exynos/mach-tiny.c

增加如下代码:

```
#ifdef CONFIG_EEPROM_AT24
#include <linux/i2c/at24.h>
static struct at24_platform_data at24c02 = {
    .byte_len = SZ_2K / 8,
    .page_size = 8,
    .flags = 0,
};
#endif
static struct i2c_board_info smdk4x12_i2c_devs0[] __initdata = {
#ifdef CONFIG_EEPROM_AT24
    {
        I2C_BOARD_INFO("24c02", 0x50),
        .platform_data = &at24c02,
    },
#endif
};
```

**代码说明**：AT24CXX使用8位地址，内存大小2K比特位，也就是256K字节，页大小为8字节。

手册中AT24CXX的设备地址是

| 1 | 0 | 1 | 0 | A2 | A1 | A0 | R/W |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |


其最低位是读写标志位，但是在Linux中，I2C设备地址的最高位为0，而低七位地址就是手册中去掉R/W的剩余7位。在_**4.实验原理**_中可知A2 A1 A0引脚接GND，因此，地址为0b 01010000（0x50）。

**3.编译并烧写内核**

**4.在终端中查看eeprom设备**

