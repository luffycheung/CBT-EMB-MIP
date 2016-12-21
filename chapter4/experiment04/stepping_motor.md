# 实验四. 步进电机控制实验

----------
##  实验目的
- 了解多核心平台底板设备电路结构;
- 掌握Android Studio下编写NDK代码，实现步进电机的控制;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写NDK程序，实现对步进电机的控制。


##  实验原理

- 硬件原理及驱动移植参见Linux光盘中的步进电机实验。
- 驱动底层预留接口如下：
```c
int open(const char *pathname, int flags); 
int close(int fildes);
int ioctl(int fd, int step,int delaytime); //delaytime：延迟时间，控制转速
```

- 底层驱动采用四拍驱动正反转。   
__示例__:
```c
static int forward[] = {1, 20, 4, 8}; //正传
static int rollback[] = {8, 4, 20, 1};//反转

 for (int i = 0; i < 4; i++) {  //正传
        ioctl(fd, forward[i], delay);
    }
```

## 实验步骤

###JNI中间层程序编写   

从[实验原理](#实验原理)相关接口函数介绍，编写`steppermoter-jni.c`代码参考如下：
```c
#include <jni.h>
#include <fcntl.h>  
#include <unistd.h>
#include "android/log.h"

static const char *TAG = "libs";
#define LOGI(fmt, args...) __android_log_print(ANDROID_LOG_INFO,  TAG, fmt, ##args)
#define LOGD(fmt, args...) __android_log_print(ANDROID_LOG_DEBUG, TAG, fmt, ##args)
#define LOGE(fmt, args...) __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, ##args)

#define DEVICE_NAME    "/dev/StepMoto"

int fd;
static int forward[] = {1, 20, 4, 8};

static int rollback[] = {8, 4, 20, 1};

static

void Motor_Forward(long delay) {

    for (int i = 0; i < 4; i++) {
        ioctl(fd, forward[i], delay);
    }
}

void Motor_Rollback(long delay) {

    for (int i = 0; i < 4; i++) {
        ioctl(fd, rollback[i], delay);
    }
}


JNIEXPORT jint JNICALL
Java_cbt_edu_iot_steppermoter_MainActivity_moterInit(JNIEnv *env, jobject instance) {

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
Java_cbt_edu_iot_steppermoter_MainActivity_moterStepForeward(JNIEnv *env, jobject instance,
                                                             jint count, jlong delay
) {

    for (int i = 0; i < count; i++) {
        Motor_Forward(delay);
    }

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_steppermoter_MainActivity_moterStepRollback(JNIEnv *env, jobject instance,
                                                             jint count, jlong delay
) {

    for (int i = 0; i < count; i++) {
        Motor_Rollback(delay);
    }

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_steppermoter_MainActivity_moterExit(JNIEnv *env, jobject instance) {

    if (fd >= 0) {
        close(fd);
        fd = -1;
    }
}
```
   
### 导入工程源码

1.  打开Android Studio，从菜单栏选择 **File \> Open**。
2.  弹窗中浏览选择光盘src目录下的Gradle工程 **CH05_NDK** ,点击**OK**导入。
3.  等待工程构建完成后，在工具栏中的*Android App*列表中选择本实验例程**CH05_04_StepperMoter**。

### 演示运行

- 运行前需将平台主板MOTOR的拨码开关拨到`ON`上供电。
- 平台主板通过miniUSB线连接电脑后，点击 **Run**
![从菜单栏运行应用](https://developer.android.com/studio/images/buttons/toolbar-run.png)
运行程序。
- 点击`加载驱动`按钮，弹窗请求权限点击`授权`。之后会在系统中生成字符驱动`/dev/StepMoto`。 

- 点击右侧`打开`按钮后即可操作该驱动。点击下方的两个按钮控制其正反转。如**图5.3.1**所示：

![ui](/chapter4/experiment04/ch05_04_ui.png) 