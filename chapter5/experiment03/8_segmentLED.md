# 实验三. 八段数码管控制实验

----------
##  实验目的
- 了解多核心平台底板设备电路结构;
- 掌握Android Studio下编写NDK代码，实现数码管的控制;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写NDK程序，实现对板载2路八段数码管的控制。


##  实验原理

### 主板电路原理图

![数码管](/chapter5/experiment03/Nixie.png)   

**图4.1** 原理图   

&emsp;&emsp;首先为了在单独的一位数字上显示出我们想要的数字，要根据数码管每一位所对应的引脚输入正确的值。一个数码管数字一共由7个长直型的发光二极管组成，分别被以”A”-“G”的英文字母编号，右下角的小数点用“DP”编号。点亮一个数码管的电路跟点亮一个普通发光二极管没什么两样，就是在高低电压之间让数码管和一个分压电阻串联，因此想要点亮带小数点的一个十进制数，需要由8个并联的发光二极管电路来完成。

&emsp;&emsp;那么如果想同时点亮两个数字，是不是需要8X2=16个并联电路呢？
&emsp;&emsp;实际上一般的数码管都不使用如此复杂的电路，而是利用人眼的“视觉暂留”效应，通过快速切换数码管的显示，达到多位数字的显示目的。这时，只需要每个数字的阳极或阴极列作为公共控制引脚，就可以控制每个数字的显示。   

**图4.1**中的内部电路是共阴极数码管的电路，每8组数码管有一个公共阴极，同一时刻，只允许一个数字发亮，当两个数字中快速切换，便可形成多位数字同时显示的效果。


### 驱动实现

&emsp;&emsp;每个数字当我们想让他亮的时候，将对应的阴极置为**低电平**即可。可得出下列二维数组：
```
unsigned char table[10][8] =
{
    {1, 1,  0,  0,  0,  0,  0,  0},         //0 --> 0xc0
    {1, 1,  1,  1,  1,  0,  0,  1},         //1 --> 0xf9
    {1, 0,  1,  0,  0,  1,  0,  0},         //2 --> 0xa4
    {1, 0,  1,  1,  0,  0,  0,  0},         //3 --> 0xb0
    {1, 0,  0,  1,  1,  0,  0,  1},         //4 --> 0x99
    {1, 0,  0,  1,  0,  0,  1,  0},         //5 --> 0x92
    {1, 0,  0,  0,  0,  0,  1,  0},         //6 --> 0x82
    {1, 1,  1,  1,  1,  0,  0,  0},         //7 --> 0xf8
    {1, 0,  0,  0,  0,  0,  0,  0},         //8 --> 0x80
    {1, 0,  0,  1,  0,  0,  0,  0}          //9 --> 0x90
};
```
   
&emsp;&emsp;以上的代码分别对应了0-9的数字在数码管上的显示IO管脚电平，按照顺时针方向，从8字顶端开始为ABCDEFG，DP，而代码中从高到低对应DP,GFEDCBA。

&emsp;&emsp;数码管底层驱动为字符驱动，采用动态编译(`4412_digitron.ko`)方式,加载后设备名为`/dev/digitron`。   

### 接口函数

```c
int open(const char *pathname, int flags); //同实验二LED
int close(int fildes);//同实验二LED
int ioctl(int fd, int number); //number为上述二维数组中对应的十六进制数值
```

这样数码管设备`/dev/digitron`通过`ioctl`赋值既可显示相应数字。   
***示例***：
```
//显示`0`(0xc0)
ioctl(fd, 0xc0);
```
<<<<<<< 7b647ecd48083c061990cc45d99435d8d21646a7
=======

## 实验步骤

###JNI中间层程序编写

从[实验原理](#实验原理)相关接口函数介绍，编写`digitron-jni.c`代码参考如下：
```c
#include <jni.h>
#include <fcntl.h>  
#include "android/log.h"

static const char *TAG = "libs";
#define LOGI(fmt, args...) __android_log_print(ANDROID_LOG_INFO,  TAG, fmt, ##args)
#define LOGD(fmt, args...) __android_log_print(ANDROID_LOG_DEBUG, TAG, fmt, ##args)
#define LOGE(fmt, args...) __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, ##args)

#define DEVICE_NAME    "/dev/digitron"
#define ZERO  0xc0
#define ONE  0xf9
#define TWO  0xa4
#define THREE  0xb0
#define FOURE  0x99
#define FIVE  0x92
#define SIX  0x82
#define SEVEN  0xf8
#define EIGHT  0x80
#define NINE  0x90
int fd;

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_nixie_MainActivity_closeDigitron(JNIEnv *env, jclass type) {
    if (fd >= 0) {
        close(fd);
        fd = -1;
    }
}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_nixie_MainActivity_setDigitronValue(JNIEnv *env, jclass type, jint digitValue) {

    int i = digitValue;
    switch (i) {
        case 0:
            ioctl(fd, ZERO);
            break;
      ...
        case 9:
            ioctl(fd, NINE);
            break;
    }
    return 1;

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_nixie_MainActivity_openDigitronDriver(JNIEnv *env, jclass type) {

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
```

### 导入工程源码

1.  打开Android Studio，从菜单栏选择 **File \> Open**。
2.  弹窗中浏览选择光盘src目录下的Gradle工程 **CH05_NDK** ,点击**OK**导入。
3.  等待工程构建完成后，在工具栏中的*Android App*列表中选择本实验例程**CH05_03_8_SegmentLED**,如**图5.2.1**所示：

![数码管工程](/chapter5/experiment03/ch05_03.png)  

**图5.2.1** 8段数码管工程

### 演示运行


- 运行前需将平台主板**JP18**处的跳线帽跳至_数码管_一侧。

![数码管跳线帽](/chapter5/experiment03/nixie_jumper_cap.png)   

**图5.3.1** 数码管跳线帽   

- 平台主板通过miniUSB线连接电脑后，点击 **Run**
![从菜单栏运行应用](https://developer.android.com/studio/images/buttons/toolbar-run.png)
运行程序，界面如**图5.3.2**所示：

![ui01](/chapter5/experiment03/ch05_03_ui_01.png)   

**图5.3.2** 主界面  

- 点击`加载驱动`按钮，弹窗请求权限点击`授权`。之后会在系统中生成字符驱动`/dev/digitron`。 

- 点击右侧`打开`按钮后即可操作该驱动。点击下方的两个按键图标即可同步控制两个8段数码管中的数值。如**图5.3.2**所示：

![ui03](/chapter5/experiment03/ch05_03_ui_03.png)   

