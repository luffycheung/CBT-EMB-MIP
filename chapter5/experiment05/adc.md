# 实验五. 基于ADC接口的旋转编码器器及光敏电阻开发实验

----------
##  实验目的
- 熟悉ADC原理;
- 掌握Android Studio下编写NDK代码，获取ADC设备相应数据;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写NDK程序，实现旋转编码器器及光敏电阻数据的获取。


##  实验原理

### 硬件原理 

>模数转换器即A/D转换器，或简称ADC，通常是指一个将模拟信号转变为数字信号的电子元件。通常的模数转换器是把经过与标准量比较处理后的模拟量转换成以二进制数值表示的离散信号的转换器。故任何一个模数转换器都需要一个参考模拟量作为转换的标准，比较常见的参考标准为最大的可转换信号大小。而输出的数字量则表示输入信号相对于参考信号的大小。  

### 驱动实现
ADC驱动已**静态编译**进内核，系统启动后生成的设备名为`/dev/adc`。
ADC通道0接收旋转编码器数据，通道1接收光敏电阻数据。


### 接口函数

```c
int open(const char *pathname, int flags); 
int close(int fildes);
int ioctl(int fd, int ADC_SET_CHANNEL,int channel); //设置adc通道0~1
ssize_t read(int fd, void *buf, size_t count);//读取adc数组
```

## 实验步骤

###JNI中间层程序编写   

从[实验原理](#实验原理)相关接口函数介绍，编写`adc-jni.c`代码参考如下：
```c
#include <jni.h>
#include <fcntl.h>  
#include <stdio.h>
#include "android/log.h"

static const char *TAG = "libs";
#define LOGI(fmt, args...) __android_log_print(ANDROID_LOG_INFO,  TAG, fmt, ##args)
#define LOGD(fmt, args...) __android_log_print(ANDROID_LOG_DEBUG, TAG, fmt, ##args)
#define LOGE(fmt, args...) __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, ##args)

#define DEVICE_NAME    "/dev/adc"
#define ADC_SET_CHANNEL     0xc000fa01
int fd;
float valt = 0;
JNIEXPORT jint JNICALL
Java_cbt_edu_iot_cbt4412_1adc_MainActivity_adcInit(JNIEnv *env, jclass type) {

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

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_cbt4412_1adc_MainActivity_adcSetChannel(JNIEnv *env, jclass type, jint channel) {

   int ret =  ioctl(fd, ADC_SET_CHANNEL, channel);
    if(ret < 0){
        LOGI("% set channel failed",DEVICE_NAME);
    }

}


JNIEXPORT jint JNICALL
Java_cbt_edu_iot_cbt4412_1adc_MainActivity_adcExit(JNIEnv *env, jclass type) {
    if (fd >= 0) {
        close(fd);
        fd = -1;
    }
}

JNIEXPORT jfloat JNICALL
Java_cbt_edu_iot_cbt4412_1adc_MainActivity_adcRead(JNIEnv *env, jclass type) {

    char buffer[30];
    int len = read(fd, buffer, sizeof buffer -1);
    if (len > 0) {
        buffer[len] = '\0';
        int value = -1;
        sscanf(buffer, "%d", &value);
        valt = value;
        return  valt;
        usleep(1* 1000);
    } else {
        LOGE("read ADC device:","failed");
        return 1;
    }
}
```

### 导入工程源码

1.  打开Android Studio，从菜单栏选择 **File \> Open**。
2.  弹窗中浏览选择光盘src目录下的Gradle工程 **CH05_NDK** ,点击**OK**导入。
3.  等待工程构建完成后，在工具栏中的*Android App*列表中选择本实验例程**CH05_05_ADC**。

### 演示运行

- 平台主板通过miniUSB线连接电脑后，点击 **Run**
![从菜单栏运行应用](https://developer.android.com/studio/images/buttons/toolbar-run.png)
运行程序。界面中会显示旋转编码器及光敏电阻的数据，如**图5.3.1**所示：

![ui](/chapter5/experiment05/ch05_05_ui.png) 

**图5.3.1** ADC

拨动编码器旋钮/用手遮住光敏电阻操作可观察到界面中数值的变化。


