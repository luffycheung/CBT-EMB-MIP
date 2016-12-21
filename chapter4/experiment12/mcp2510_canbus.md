# 实验十二. 基于MCP2510的CAN总线开发实验

 ---
##  实验目的

* 掌握CAN总线通讯原理;
* 学习CAN字符驱动接口，编写NDK程序实现控制;

##  实验环境

* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容

* 学习CAN字符驱动接口，使用Android Stuido创建JNI项目操作该设备来控制CAN总线实现通讯。

##  实验原理

CAN总线控制器`MCP2510`硬件原理及使用SPI子系统注册MCP2510 CAN总线驱动具体流程请参阅Linux光盘中CAN总线实验章节。

系统运行后会产生`dev/can0`设备，上层编写JNI程序对其进行操作实现读写等功能。

### 驱动接口函数

在`can.h`头文件中定义操作CAN总线的宏定义及结构体，具体如下：
```c
#ifndef CAN_H__
#define CAN_H__

#include <linux/types.h>

#define CAN_IOCTRL_SETBAND            0x1    //设置波特率
#define CAN_IOCTRL_SETID              0x5    //设置帧ID
#define CAN_IOCTRL_SETLPBK            0x3    //设置模式（回环/正常）
#define CAN_IOCTRL_SETFILTER          0x4    //为CAN设备设置过滤器
#define CAN_IOCTRL_PRINTRIGISTER    0x2    // 打印spi和portE的寄存器信息

#define CAN_EXCAN                    (1<<31)    //extern can flag
typedef enum {
    BandRate_125kbps = 1,
    BandRate_250kbps = 2,
    BandRate_500kbps = 3,
    BandRate_1Mbps = 4
} CanBandRate;

typedef struct {
    unsigned int id;          //CAN总线ID
    unsigned char data[8];    //CAN总线数据
    unsigned char dlc;        //数据长度
    int IsExt;    //是否是扩展总线
    int rxRTR;    //是否是远程帧
} CanData, *PCanData;

/*********************************************************************\
    CAN设备设置接收过滤器结构体
    参数: IdMask，Mask
            IdFilter，Filter
    是否接收数据按照如下规律:
    Mask    Filter  RevID   Receive
    0       x       x       yes
    1       0       0       yes
    1       0       1       no
    1       1       0       no
    1       1       1       yes
    
\*********************************************************************/
typedef struct {
    unsigned int Mask;
    unsigned int Filter;
    int IsExt;    //是否是扩展ID
} CanFilter, *PCanFilter;

#endif

```

- 打开设备
```c
 fd = open("dev/can0", O_RDWR);
```

- 关闭设备
```c
 close(fd);
```
- 初始化CAN设备
```c
  ioctl(fd, CAN_IOCTRL_PRINTRIGISTER, 1);
        ioctl(fd, CAN_IOCTRL_SETID, id);
        if (isLoop) { //是否回环标志位
            ioctl(fd, CAN_IOCTRL_SETLPBK, 1);
        }
```
- 读取数据
```c
 char temp[16]; //保存CAN数据帧中的数据位数据
    bzero(temp, 16);
    int k = 0;
    CanData data = {0};
    if (fd == -1) {
        LOGE("Can is not open!", "");
        data.id = 0;
        data.dlc = 0;
    } else {
        read(fd, &data, sizeof(CanData));
        for (k = 0; k < data.dlc; k++)
            temp[k] = data.data[k];
        temp[k] = 0;
        sleep(delaytime);
    }
```
- 写数据
```c
static void CanSendString(char *pstr) {
    CanData data;
    int len = strlen(pstr);
    memset(&data, 0, sizeof(CanData));
    data.id = 0x123;
    data.dlc = 8;
    for (; len > MAX_CANDATALEN; len -= MAX_CANDATALEN) {
        memcpy(data.data, pstr, 8);
        write(fd, &data, sizeof(data));
        pstr += 8;
    }
    memset(&data, 0, sizeof(CanData));
    data.id = 0x123;
    data.dlc = len;
    memcpy(data.data, pstr, len);
    write(fd, &data, sizeof(data));
}
```

##  实验步骤

### 导入工程源码

1.  打开Android Studio，从菜单栏选择 **File \> Open**。
2.  弹窗中浏览选择光盘src目录下的Gradle工程 **CH05_NDK** ,点击**OK**导入。
3.  等待工程构建完成后，在工具栏中的*Android App*列表中选择本实验例程**CH05_12_CAN**。

### 演示运行

- 平台主板通过miniUSB线连接电脑后，点击 **Run**
![从菜单栏运行应用](https://developer.android.com/studio/images/buttons/toolbar-run.png)
运行程序。

- 进入界面后若单个平台测试时需勾选[x]`回环显示`选项。
- 点击右侧的`打开`按钮打开CAN设备。弹窗提示请求获取权限，点击`授权`允许。
若打开成功下方的`清空`和`发送`按钮才可以使能。
- 在左侧文本框中输入相应英文字符，点击下方`发送`按钮。发送成功会在右侧接收区显示，如**图5.2.1**所示：

![CAN](/chapter4/experiment12/ch05_12_ui.png)

**图5.2.1** CAN测试程序
