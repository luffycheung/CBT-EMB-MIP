# 实验九. 基于IIC接口的ADXL345三轴加速度开发实验

----------
##  实验目的
- 了解IIC驱动;
- 掌握Android Studio下编写NDK代码，实现获取三轴加速度的数值;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写NDK程序，实现获取三轴加速度的数值。


##  实验原理

### IIC驱动体系

体系结构包括：IIC总线驱动；IIC核心；IIC从设备驱动。
1. I2C核心

IIC 核心提供了IIC总线驱动和设备驱动的注册、注销方法，IIC通信方法（即“algorithm”），与具体适配器无关的代码以及探测设备、检测设备地址等。i2c-core.c中的核心驱动程序可管理多个IIC总线适配器（控制器）和多个IIC从设备。每个IIC从设备驱动都能找到和它相连的IIC总线适配器。

2. IIC总线驱动

IIC总线驱动主要包括IIC适配器结构i2c_adapter和I2C适配器的algorithm数据结构。

通过IIC总线驱动的代码，可控制IIC适配器以主控方式产生开始位、停止位、读写周期，以及以从设备方式被读写、产生ACK等。

3. IIC设备驱动

IIC设备驱动是对IIC设备端的实现，设备一般挂接在受CPU控制的I2C适配器上，通过IIC适配器与CPU交换数据。IIC设备驱动主要包括数据结构i2c_driver和i2c_client。


### ADXL345驱动实现

IIC驱动体系中一般来说硬件上可含有一个或多个从设备，一个或多个适配器，这俩之间的关系：**一个适配器可以控制多个从设备；且我们既可以通过直接编写从设备驱动来操控他，也可以通过操作适配器来操控从设备**。

I2C总线的设备文件通常为/dev/i2c-n（n=0、1、2……），每个设备文件对应一组I2C总线。应用程序通过这些设备文件可以操作I2C总线上的任何从机器件。

本实例中ADXL345设备采用**IIC总线驱动**的方式。通过操作控制器来产生特定的IIC时序信号，来发送和接收数据，让适配器工作。

### IIC编程接口

由于采用总线驱动方式，需调用`i2c.h`及`i2c-dev.h`这两个头文件。使用到的系统方法有：
```c
/* This is the structure as used in the I2C_SMBUS ioctl call */
struct i2c_smbus_ioctl_data {
    __u8 read_write;
    __u8 command;
    __u32 size;
    union i2c_smbus_data __user *data;
};

/*
 * Data for SMBus Messages
 */
#define I2C_SMBUS_BLOCK_MAX 32  /* As specified in SMBus standard */
union i2c_smbus_data {
    __u8 byte;
    __u16 word;
    __u8 block[I2C_SMBUS_BLOCK_MAX + 2]; /* block[0] is used for length */
                   /* and one more for user-space compatibility */
};

```

1. 打开设备

在操作I2C总线时，先调用open()函数打开I2C设备获得文件描述符，代码如下所示。

打开I2C设备文件
```c
int fd;
fd = open("/dev/i2c-0", O_RDWR); 
if (fd < 0) {
perror("open i2c-1 n");
}
```
2. 关闭设备

当操作完成后，调用close()函数关闭设备：
```c
close(fd);
```

3. 配置设备

当应用程序操作I2C总线上的从机器件时，必须先调用ioctl()函数设置从机地址和从机地址的长度。

设置从机地址

设置从机地址是使用I2C_SLAVE命令，其定义为：
```c
#define I2C_SLAVE 0x0703
```

该命令的参数为从机地址右移一位。设置从机地址为0xA0的示例代码为：
```c
if (ioctl(GiFd, I2C_SLAVE, 0xA0>> 1)< 0) {
perror("set slave address failen");
}
```
注意：地址需要右移一位，是因为地址的Bit0是读写控制位，在驱动中会将从机地址命令参数左移一位，并补上读写控制位。

设置地址长度

设置从机地址的长度是使用I2C_TENBIT命令，其定义为：
```c
#define I2C_TENBIT 0x0704
```
该命令的参数可选择为：1表示设置从机地址长度为10位；0表示设置从机地址长度为8位。设置从机地址长度为10位的示例代码为：
```c
ioctl(fd, I2C_TENBIT, 1);
```
该命令是不会返回错误的。

如果不设置地址长度，则默认为8位地址。

4. 发送/接收数据

通过调用系统方法ioctl传递不同参数`args`实现数据读写。
```c
ioctl(file, I2C_SMBUS, &args);
```
封装函数如下：
```c
static inline __s32 i2c_smbus_access(int file, char read_write, __u8 command,
                                     int size, union i2c_smbus_data *data) {
    struct i2c_smbus_ioctl_data args;

    args.read_write = read_write;
    args.command = command;
    args.size = size;
    args.data = data;
    return ioctl(file, I2C_SMBUS, &args);
}
```

##  实验步骤

###JNI中间层编程范例  

从[实验原理](#实验原理)相关接口函数介绍，编写`adxl345-jni.c`代码参考如下：
```c
#include <jni.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
//#include <linux/i2c-dev.h>
//#include <linux/i2c.h>

#include "android/log.h"
#include "i2c-dev.h"
#include "i2c.h"

static const char *TAG = "libs";
#define LOGI(fmt, args...) __android_log_print(ANDROID_LOG_INFO,  TAG, fmt, ##args)
#define LOGD(fmt, args...) __android_log_print(ANDROID_LOG_DEBUG, TAG, fmt, ##args)
#define LOGE(fmt, args...) __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, ##args)

#define DEVICE_NAME    "/dev/i2c-0"
#define I2C_SLAVE       0x0703  /* Use this slave address */
#define I2C_TENBIT      0x0704
#ifndef uchar
#define uchar unsigned char
#endif
int fd, n, res;
unsigned char id;
unsigned char dataxh = 0;
unsigned char dataxl = 0;
unsigned char datayh = 0;
unsigned char datayl = 0;
unsigned char datazh = 0;
unsigned char datazl = 0;
unsigned int datax = 0;
unsigned int datay = 0;
unsigned int dataz = 0;
float x, y, z;
uchar set1_buffer[7] = {0x0B, 0x0B, 0x08, 0x80, 0x00, 0x00, 0x00};  ////adxl345设置
//参数
uchar reg1_buffer[7] = {0x31, 0x2c, 0x2d, 0x2e, 0x1e, 0x1f, 0x20};

static inline __s32 i2c_smbus_access(int file, char read_write, __u8 command,
                                     int size, union i2c_smbus_data *data) {
    struct i2c_smbus_ioctl_data args;

    args.read_write = read_write;
    args.command = command;
    args.size = size;
    args.data = data;
    return ioctl(file, I2C_SMBUS, &args);
}

static inline __s32 i2c_smbus_read_byte_data(int file, __u8 command) {
    union i2c_smbus_data data;
    if (i2c_smbus_access(file, I2C_SMBUS_READ, command,
                         I2C_SMBUS_BYTE_DATA, &data)) {
        printf("no data\n");
        return -1;
    }
    else {
        //printf("datatmp:%02x\n",data.byte);
        return 0x0FF & data.byte;
    }
}

static inline __s32 i2c_smbus_write_byte_data(int file, __u8 command,
                                              __u8 value) {
    union i2c_smbus_data data;
    data.byte = value;
    return i2c_smbus_access(file, I2C_SMBUS_WRITE, command,
                            I2C_SMBUS_BYTE_DATA, &data);
}

static int read_adxl345(int fd, __u8 buff[], int addr, int count) {
    int res;

    buff[0] = i2c_smbus_read_byte_data(fd, addr);

    //res=read(fd,buff,count);
    return res;
}

static int write_adxl345(int fd, __u8 data, int addr/*, int count*/) {
    int i = i2c_smbus_write_byte_data(fd, addr, data);
    if (i < 0)
        printf("write error\n");
}

int toPrimary(unsigned int data) {
    int temp = 0;
    if (data & 0x8000) {
        temp = data ^ 0xffff;
        temp -= 1;
        return temp * (-1);
    }
    else
        temp = data;
//   printf("temp:%02x\n",temp);
    return temp;
}


JNIEXPORT jint JNICALL
Java_cbt_edu_iot_adxl345_MainActivity_adxl345Init(JNIEnv *env, jclass type) {
    res = ioctl(fd, I2C_TENBIT, 0); //not 10bit
    res = ioctl(fd, I2C_SLAVE, 0x53); //0x53为 ADXL345 IIC设备地址
    write_adxl345(fd, *(set1_buffer + 0),
                  reg1_buffer[0]);//数据格式控制,禁用自测力>，中断高电平有效，全分辨率模式（正负16g，13 bit），右对齐并带符号扩展
    write_adxl345(fd, *(set1_buffer + 1),
                  reg1_buffer[1]);//非低功耗模式，器件带宽为12.5Hz，输出数据速率为25Hz；（默认带宽50，速率100 ）
    write_adxl345(fd, *(set1_buffer + 2), reg1_buffer[2]);//静止功能和活动功能同时打开，禁止自动切换至休眠模式，测量模式，不休眠
    write_adxl345(fd, *(set1_buffer + 3), reg1_buffer[3]);
    write_adxl345(fd, *(set1_buffer + 4), reg1_buffer[4]);//X 偏移量
    write_adxl345(fd, *(set1_buffer + 5), reg1_buffer[5]);//Y 偏移量
    write_adxl345(fd, *(set1_buffer + 6), reg1_buffer[6]);//Z 偏移量
    write_adxl345(fd, 0x4f, 0x38);
    write_adxl345(fd, 0x00, 0x2e); //所有中断均关闭
    write_adxl345(fd, 0x00, 0x2f); //INT1

    write_adxl345(fd, 0x00, 0x28); //不使用FIFO
    read_adxl345(fd, &id, 0x00, 1);
    LOGI("adxl345 chip ID = %x \n", id);

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_adxl345_MainActivity_adxl345_1open(JNIEnv *env, jclass type) {

    fd = open(DEVICE_NAME, O_RDWR);//打开设备
    if (fd == -1) {
        LOGI("open device %s error \n", DEVICE_NAME);
        return 0;
    }
    else {
        LOGI("open device %s ok! \n", DEVICE_NAME);
        return 1;
    }

}

JNIEXPORT jfloat JNICALL
Java_cbt_edu_iot_adxl345_MainActivity_adxl345_1getAccel_1x(JNIEnv *env, jclass type) {

    read_adxl345(fd, &dataxl, 0x32, 1);
    read_adxl345(fd, &dataxh, 0x33, 1);
    datax=dataxh*256+dataxl;
    x=toPrimary(datax)*0.0039;
    return x;
}

JNIEXPORT jfloat JNICALL
Java_cbt_edu_iot_adxl345_MainActivity_adxl345_1getAccel_1y(JNIEnv *env, jclass type) {

    read_adxl345(fd, &datayl, 0x34, 1);
    read_adxl345(fd, &datayh, 0x35, 1);
    datay=datayh*256+datayl;
    y=toPrimary(datay)*0.0039;
    return y;
}

JNIEXPORT jfloat JNICALL
Java_cbt_edu_iot_adxl345_MainActivity_adxl345_1getAccel_1z(JNIEnv *env, jclass type) {

    read_adxl345(fd, &datazl, 0x36, 1);
    read_adxl345(fd, &datazh, 0x37, 1);
    dataz=datazh*256+datazl;
    z=toPrimary(dataz)*0.0039;
    return z;
}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_adxl345_MainActivity_adxl345Exit(JNIEnv *env, jclass type) {

    if (fd >= 0) {
        close(fd);
        fd = -1;
    }

}
```

### 导入工程源码

1.  打开Android Studio，从菜单栏选择 **File \> Open**。
2.  弹窗中浏览选择光盘src目录下的Gradle工程 **CH05_NDK** ,点击**OK**导入。
3.  等待工程构建完成后，在工具栏中的*Android App*列表中选择本实验例程**CH05_09_ADXL345**。

### 演示运行

- 平台主板通过miniUSB线连接电脑后，点击 **Run**
![从菜单栏运行应用](https://developer.android.com/studio/images/buttons/toolbar-run.png)
运行程序。
- 界面会显示当前多核心平台设备的三轴加速度实时数值，如**图5.3.1**所示：

![adxl345界面](/chapter4/experiment09/ch05_09_ui.png)

**图5.3.1** 主界面

- ADXL345用户手册中输出响应与相对于重力的方向的关系如**图5.3.2**所示：

![方向](/chapter4/experiment09/ch05_09_adxl345.png)

**图5.3.2** 输出响应与相对于重力的方向的关系