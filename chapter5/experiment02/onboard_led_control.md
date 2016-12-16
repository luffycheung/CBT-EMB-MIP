#实验二. 板载LED灯控制实验

----------
##  实验目的
- 了解多核心平台底板设备电路结构;
- 掌握Android Studio下编写NDK代码，实现LED灯控制;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写NDK程序，实现对板载5个LED灯的控制。


##  实验原理
&emsp;&emsp;LED硬件原理及驱动移植请参见Linux实验文档LED驱动移植章节。    
&emsp;&emsp;LED字符驱动程序\(`linux-3.5/drivers/char/4412_led.c`\)已**动态编译**进内核`zImage`里。
在Android文件系统中加载内核生成的module\(`4412_led.ko`)会生成LED灯设备的文件\(`dev/leds`\)。   
&emsp;&emsp;在Linux 系统中，所有设备都是以文件的形式被打开并进行读/写操作的。本实验中通过编写JNI层代码调用**POSIX**的文件接口函数对底层设备进行操作，实现上层应用程序控制LED灯亮灭。

### [本实验JNI编程时需要用到以下几个文件操作函数](#本实验JNI编程时需要用到以下几个文件操作函数)  

####  OPEN - 打开设备   
   - 【函数原型】   
            
            #include <fcntl.h>
            int open(const char *pathname, int flags); 
            int open(const char *pathname, int flags, mode_t mode);
            
   - 【功能】打开名为path 的文件或设备，成功打开后返回文件句柄。
   - 【参数】`pathname`:文件路径或设备名；`flags`:打开方式，可选值是**表4.1**中的一个值或几个值的组合。
    
| 打开方式 |  含义 | 
|:-------------|:----------------|
| O_RDONLY | 只读方式打开 | 
| O_WRONLY | 只写方式打开 | 
| O_RDWR | 读写方式打开(等同于 O_RDONLY &vert;  O_WRONLY) | 
| ... | ... | 
| O_NONBLOCK | 采用非阻塞文件I/O方式 |    

**表4.1** 打开方式对照表

   - 【返回值】成功打开后返回文件句柄，失败返回-1。   

####   IOCTL - 控制高低电平
- 【函数原型】
        
            #include <unistd.h>
            int ioctl(int fildes, int request, /* arg */ ...);
            int ioctl(int fd, int ledState,int ledID);
    

- 【功能】控制I/O设备(指定LED灯的亮灭)   
    
- 【参数】`fildes`：文件或设备句柄，通常由open函数返回；   
    `ledState`:LED灯控制参数(0:灭/1:亮);`ledID`:5个led灯对应的ID。
- 【返回值】成功返回0，失败返回错误码。
    
####  CLOSE - 关闭设备
- 【函数原型】
        
            #include <unistd.h>
            int close(int fildes);
    

- 【功能】关闭之前被打开的文件或设备  
    
- 【参数】`fildes`：文件或设备句柄，通常由open函数返回
- 【返回值】成功返回0，失败返回-1。

### JNI代码具体实现   

在本实验中，需要使用1个设备文件：`/dev/leds`对多核心嵌入式平台的5个LED灯进行控制。   

1. 打开LED设备   

        #define DEVICE_NAME    "/dev/leds"
        ...
        fd = open(DEVICE_NAME, O_RDWR);//打开设备   

2. 调用IOCTL控制   

        #define LED_ON 1
        #define LED_OFF 0
        ...
        ioctl(fd, LED_ON, ledID);  //打开指定ID的LED灯
        ...
        ioctl(fd, LED_OFF, ledID); //关闭指定ID的LED灯  

3. 关闭LED设备   

        close(fd);


##  实验步骤
### 在Android Studio下创建支持C/C++的新项目   

创建步骤参见本章实验一，本实验例程中创建的项目如`图5.1`所示。
![LED工程](/chapter5/experiment02/ch05_02.png)   

**图5.1** LED项目工程   

### 更改库名及CMake构建脚本

- `MainActivity`更改加载库名称，注释掉`stringFromJNI()`相关调用。   

```
    static {
        //System.loadLibrary("native-lib");
        System.loadLibrary("leds-jni");
    }
    ...
    //TextView tv = (TextView) findViewById(R.id.sample_text);
    //tv.setText(stringFromJNI());
    ...
    //public native String stringFromJNI();
```
 

- 修改CMake构建配置文件   

```
add_library( # Sets the name of the library.
             leds-jni
                ...
             src/main/jni/leds-jni.c )
             ...
target_link_libraries( # Specifies the target library.
                       leds-jni

                       ${log-lib} )             
```

- 点击工具栏中的 **Sync Project**
![](https://developer.android.com/studio/images/buttons/toolbar-sync-gradle.png)应用更改。   

### 在`MainActivity`中添加LED操作的本地方法
```
//    public native String stringFromJNI();
    public native int ledsOpen();
    public native int ledsClose();
    public native int ledsControl(int ledID, int ledState);
```

在方法名中按下<kbd>Alt</kbd>+<kbd>Enter</kbd>，选择`Create Function ...`  > `Fix all 'Missing JNI function' problems in file`选项。项目`src\main`目录下会生成`jni`文件夹及`leds-jni.c`原生源文件。

- 删除文件夹`cpp`及`native-lib.cpp`     

- 点击工具栏中的 **Sync Project**
![](https://developer.android.com/studio/images/buttons/toolbar-sync-gradle.png)应用更改。   

### 编写原生源文件`leds-jni.c`

按照[实验原理](#本实验JNI编程时需要用到以下几个文件操作函数)中的接口方法编写实现对应的打开、控制及关闭功能。   

```c
#include <jni.h>
#include <fcntl.h>
#include "android/log.h"

static const char *TAG = "libs";
#define LOGI(fmt, args...) __android_log_print(ANDROID_LOG_INFO,  TAG, fmt, ##args)
#define LOGD(fmt, args...) __android_log_print(ANDROID_LOG_DEBUG, TAG, fmt, ##args)
#define LOGE(fmt, args...) __android_log_print(ANDROID_LOG_ERROR, TAG, fmt, ##args)

#define DEVICE_NAME    "/dev/leds"
#define LED_ON 1
#define LED_OFF 0
int fd;

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_leds_MainActivity_ledsControl(JNIEnv *env, jobject instance, jint ledID,
                                               jint ledState) {

    int i = ledState;
    switch (i) {
        case LED_ON:
            ioctl(fd, LED_ON, ledID);
            break;

        case LED_OFF:
            ioctl(fd, LED_OFF, ledID);
            break;

        default:
            break;
    }
    return 1;

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_leds_MainActivity_ledsClose(JNIEnv *env, jobject instance) {

    if (fd >= 0) {
        close(fd);
        fd = -1;
    }

}

JNIEXPORT jint JNICALL
Java_cbt_edu_iot_leds_MainActivity_ledsOpen(JNIEnv *env, jobject instance) {

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

### 编写UI界面及代码

- 将光盘目录`src\chapter5\experiment02\res`目录下的**drawable**、**layout**和**values**三个文件夹拷贝至本项目工程的**res**目录下。

- 打开`activity_main.xml`界面如图5.1所示：
![主界面](/chapter5/experiment02/activity_main.png)   

**图5.1** 主界面   

- 编写`MainActivity`主程序,_参考代码_如下：   

```java
    public static final int LED_ON = 1;
    public static final int LED_OFF = 0;
    public static final int LED1 = 0;
    public static final int LED2 = 1;
    public static final int LED3 = 2;
    public static final int LED4 = 3;
    public static final int LED5 = 4;
    ImageView ivLed1, ivLed2, ivLed3, ivLed4, ivLed5;
    Switch swLed1, swLed2, swLed3, swLed4, swLed5;
    ...
  private void LayoutInit() {
        ivLed1 = (ImageView) findViewById(R.id.iv_led1);
        ...
        ivLed5 = (ImageView) findViewById(R.id.iv_led5);
        swLed1 = (Switch) findViewById(R.id.sw_led1);
        ...
        swLed5 = (Switch) findViewById(R.id.sw_led5);
        swLed1.setOnCheckedChangeListener(this);
        ...
        swLed5.setOnCheckedChangeListener(this);
    }

    @Override
    public void onCheckedChanged(CompoundButton compoundButton, boolean b) {
        switch (compoundButton.getId()) {
            case R.id.sw_led1:
                if (b) {
                    ledsControl(LED1, LED_ON);
                    ivLed1.setImageResource(R.drawable.ic_led_on);
                } else {
                    ledsControl(LED1, LED_OFF);
                    ivLed1.setImageResource(R.drawable.ic_led_off);
                }
                break;
           ...
            case R.id.sw_led5:
                if (b) {
                    ledsControl(LED5, LED_ON);
                    ivLed5.setImageResource(R.drawable.ic_led_on);
                } else {
                    ledsControl(LED5, LED_OFF);
                    ivLed5.setImageResource(R.drawable.ic_led_off);
                }
                break;
        }

    }
```

具体实现细节可参考光盘目录下相应实验源码。

- 构建和运行示例程序

操作应用程序之前需将多核心平台底板5路LED灯左侧的跳线帽跳到LED处，如**图5.2**所示：   

![LED跳线帽](/chapter5/experiment02/led_jumper_cap.png)   

**图5.2** LED跳线帽   

### 演示运行

运行程序后，首先点击`加载驱动(modules)`按键，会弹出如**图5.3**对话框，请求获取超级用户权限，点击`授权`。   

![加载驱动](/chapter5/experiment02/load_modules.png)   

**图5.3** 加载驱动   

点击右侧的`打开`按钮会调用`ledsOpen()`方法打开LED设备。
Android Studio界面下方的**Android Monitor**里的logcat会打印如下信息：
```
01-01 13:41:15.655 29662-29662/cbt.edu.iot.leds I/libs: open device /dev/leds ok! 
```

之后便可以点击下方的**Switch**按钮，对5个LED灯进行控制，如**图5.4**所示：   

![LED灯控制](/chapter5/experiment02/led1_control.png)

**图5.4** LED灯控制




