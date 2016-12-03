# 实验二. 基于IIC总线的EEPROM驱动移植实验

---

## 1. 实验目的

* 熟悉平台主板外围设备EEPROM芯片AT24C08的电路结构与工作原理;
* 掌握IIC总线设备驱动移植流程;

## 2. 实验环境

* 硬件：CBT-EMB-MIP 实验平台,PC机;
* 软件：VMware,Linux OS;

## 3. 实验内容

* 移植IIC总线EEPROM设备驱动;

## 4. 实验原理

* EEPROM简介

> EEPROM \(Electrically Erasable Programmable Read-Only Memory\)， 电可擦可编程只读存储器--一种掉
>
> 电后数据不丢失的存储芯片。 EEPROM 可以在电脑上或专用设备上擦除已有信息，重新编程。

* 平台主板外设原理图

![](/chapter4/experiment02/AT24.png)

CBT-EMB-MIP多核心嵌入式创新平台EEPROM使用的是AT24C08 芯片，该芯片24WC08最多可连接2个器件 且仅使用地址管脚 A2 A0 A1 管脚未用可以连接到 Vss 或悬空如果只有一个 24WC08 被总线寻址 A2 管脚可悬空或连接Vss。

## 5. 实验步骤

1.步骤1

巴拉巴拉  
行内代码块 `code`

2.步骤2

巴拉巴拉

3.步骤3

巴拉巴拉

4.步骤4

巴拉巴拉

