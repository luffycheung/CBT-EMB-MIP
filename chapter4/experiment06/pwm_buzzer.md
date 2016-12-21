# 实验六. 基于PWM的无源蜂鸣器控制实验

----------
##  实验目的
- 了解PWM原理;
- 掌握Android Studio下编写NDK代码，实现蜂鸣器的控制;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写NDK程序，实现对蜂鸣器的控制。


##  实验原理

- 硬件原理及驱动移植参见Linux光盘中的PWM实验。
- 驱动底层预留接口如下：
```c
int open(const char *pathname, int flags); 
int close(int fildes);
int ioctl(int fd, int step,int delaytime); 
int ioctl(int fd, int PWM_IOCTL_SET_FREQ, int frequency);
int ioctl(int fd, int PWM_IOCTL_STOP);
```
通过调节PWM的频率实现声音大小的控制；通过ioctl传递stop参数实现播放/暂停功能。

## 实验步骤

###JNI中间层程序编写   

从[实验原理](#实验原理)相关接口函数介绍，编写`buzzer-jni.c`代码参考如下：
```c
#include <jni.h>
#include <fcntl.h>  //文件操作相关函数
#include "android/log.h"

static const char *TAG = "libs";
#define LOGI(fmt, args...) __android_log_print(ANDROID_LOG_INFO,  TAG, fmt, ##args)
#define LOGD(fmt, args...) __android_log_print(ANDROID_LOG_DEBUG, TAG, fmt, ##args)
#define LOGE(fmt, args...) __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, ##args)

#define DEVICE_NAME    "/dev/pwm"
#define PWM_IOCTL_SET_FREQ       1
#define PWM_IOCTL_STOP            0
int fd;

JNIEXPORT jboolean JNICALL
Java_cbt_edu_iot_cbt4412_1buzzer_MainActivity_buzzerInit(JNIEnv *env, jclass type) {

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
Java_cbt_edu_iot_cbt4412_1buzzer_MainActivity_buzzerSet(JNIEnv *env, jclass type, jint frequency) {

    int ret = ioctl(fd, PWM_IOCTL_SET_FREQ, frequency);
    if (ret < 0) {
        LOGI("set the frequency of the buzzer", "");
    }

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_cbt4412_1buzzer_MainActivity_buzzerStop(JNIEnv *env, jclass type) {

    int ret = ioctl(fd, PWM_IOCTL_STOP);
    if (ret < 0) {
        LOGI("stop the %s success \n", DEVICE_NAME);
    }
    if (ioctl(fd, 2) < 0) {
        LOGI("ioctl 2:", "");
    }

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_cbt4412_1buzzer_MainActivity_buzzerExit(JNIEnv *env, jclass type) {

    if (fd >= 0) {
        ioctl(fd, PWM_IOCTL_STOP);
        if (ioctl(fd, 2) < 0) {
            LOGI("ioctl 2:", "");
        }
        close(fd);
        fd = -1;
    }

}
```

### 导入工程源码

1.  打开Android Studio，从菜单栏选择 **File \> Open**。
2.  弹窗中浏览选择光盘src目录下的Gradle工程 **CH05_NDK** ,点击**OK**导入。
3.  等待工程构建完成后，在工具栏中的*Android App*列表中选择本实验例程**CH05_06_Buzzer**。

### 演示运行

- 运行前需将蜂鸣器旁边的跳线帽跳到`BUZZER`位置。

- 平台主板通过miniUSB线连接电脑后，点击 **Run**
![从菜单栏运行应用](https://developer.android.com/studio/images/buttons/toolbar-run.png)
运行程序。

- 点击`加载驱动`按钮，弹窗请求权限点击`授权`。之后会在系统中生成字符驱动`/dev/pwm`。 

- 点击右侧`打开`按钮后即可操作该驱动。点击下方按钮控制其音量加减及播放暂停。如**图5.3.1**所示：

![ui](/chapter4/experiment06/ch05_06_ui.png) 