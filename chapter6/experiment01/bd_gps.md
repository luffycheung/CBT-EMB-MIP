# 实验一. 北斗/GPS模块实验

----------
##  实验目的
- 学习北斗GPS定位技术，掌握GPS协议标准;
- 掌握Android串口数据解析的方法;
- 掌握Handler的使用机制，实现子线程与UI线程的通信

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机，北斗GPS双核模块;
* 软件：Android Studio;

## 实验内容

* 分析北斗GPS输出数据协议帧各数据位含义;
* 编写Android程序解析GPS数据;

## 实验原理

### GPS简介

GPS(Global Positioning System), 即全球定位系统，它是一个由覆盖全球的24颗卫星组成的卫星系统。其目的是在全球范围内对地面和空中目标进行准确定位和监测。随着全球性空间定位信息应用的日益广泛，GPS提供的全时域、全天候、高精度定位服务将给空间技术、地球物理、大地测绘、遥感技术、交通调度、军事作战以及人们的日常生活带来巨大的变化和深远的影响。

GPS系统一般由地面控制站、导航卫星和用户接收机（GPS的移动用户端）三大部分组成。导航卫星至少24颗，均匀分布在6个极地轨道上，轨道的夹角为60度，距地平均高度为20200公里，每12恒星时绕地球一周。

### 北斗GPS标准NEMA数据格式协议介绍

![](/chapter6/experiment01/4.2.png)

北斗双核模块NEMA协议输出信息主要包括以下几个部分：


<table class="table table-bordered table-striped table-condensed">

 <tr>
  <th>模式</th>
  <th>语句名称</th>
  <th>功能</th>
   </tr>
 <tr>
<tr>
    <th rowspan="5">GPS模式</th>

</tr>
<tr>
   <td>GPGGA</td>
    <td>坐标位置数据</td>
</tr>
<tr>
   <td>GPRMC</td>
    <td>运输定位数据</td>
</tr>
<tr>
   <td>GPGSA</td>
    <td>DOP与有效卫星</td>
</tr>
<tr>
   <td>GPGSV</td>
    <td>GPS卫星状态信息</td>
</tr>

<tr>
    <th rowspan="5">BD模式</th>

</tr>
<tr>
   <td>BDGGA</td>
    <td>坐标位置数据</td>
</tr>
<tr>
   <td>BDRMC</td>
    <td>运输定位数据</td>
</tr>
<tr>
   <td>BDGSA</td>
    <td>DOP与有效卫星</td>
</tr>
<tr>
   <td>BDGSV</td>
    <td>BD卫星状态信息</td>
</tr>

<tr>
    <th rowspan="8">GPS&BD混合模式</th>

</tr>
<tr>
   <td>GNGGA</td>
    <td>卫星定位信息</td>
</tr>
<tr>
   <td>GNRMC</td>
    <td>运输定位数据</td>
</tr>
<tr>
   <td>GNGSA</td>
    <td>DOP与有效卫星</td>
</tr>
<tr>
   <td>GPGSV</td>
    <td>GPS卫星状态信息</td>
</tr>
<tr>
   <td>BDGSV</td>
    <td>BD卫星状态信息</td>
</tr>
<tr>
   <td>GNGLL</td>
    <td>含经、纬度的地理信息</td>
</tr>
<tr>
   <td>GNVTG</td>
    <td>对地方向及地面速度</td>
</tr>
</table>

本实验中所使用的北斗GPS模块工作在GPS&BD混合模式下，具体协议格式见光盘doc目录下的《BD-126输出数据协议文档.pdf》。

## 实验步骤

