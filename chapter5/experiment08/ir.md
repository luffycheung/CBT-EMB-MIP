# 实验八. 红外接收实验

----------
##  实验目的
- 了解Android用户输入系统框架;
- 掌握上层获取输入事件的方法;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写上层程序，实现获取红外遥控器传入的事件信息。


##  实验原理

内核中已移植红外驱动，实现红外遥控解码，将遥控器上的按键解码出键值并上报给系统。属于input设备，工作原理类似于键盘。
驱动实现细节可参见红外驱动移植实验章节。

Android输入系统框图如**图4.1**所示：

![Android输入系统结构](/chapter5/experiment08/android_input_system.jpg) 

**图4.1** Android输入系统结构

自下而上，Android的用户输入系统分成几个部分：

 驱动程序：在/dev/input目录中，通常是Event类型的驱动程序

EventHub：本地框架层的EventHub是libui中的一部分，它实现了对驱动程序的控制，并从中获得信息

KeyLayout（按键布局）和KeyCharacterMap（按键字符映射）文件。同时，libui中有相应的代码对其操作。定义按键布局和按键字符映射需要运行时配置文件的支持，它们的后缀名分别为kl和kcm

Java框架层的处理：在Java框架层具有KeyInputDevice等类用于处理由EventHub传送上来的信息，通常信息由数据结构RawInputEvent和KeyEvent来表示。通常情况下，对于按键事件，则直接使用KeyEvent来传送给应用程序层，对于触摸屏和轨迹球等事件，则由RawInputEvent经过转换后，形成MotionEvent时间传送给应用程序层

在Android的应用程序层中，通过重新实现onTouchEvent和onTrackballEvent等函数来接收运动事件（MotionEvent），通过重新实现onKeyDown和onKeyUp等函数来接收按键事件（KeyEvent）。这些类包含在android.view包中。

本地框架层底层已实现，本实验主要为编写上层应用层获取红外事件信息。

## 实验步骤

- 新建普通Android工程
- 在`MainActivity`中重新系统方法`onKeyDown`，代码如下：
```
  @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        textView.setText("KeyEvent: " + event.toString());
        return true;
    }
```
- 编译运行
- 使用**内核中已经支持的遥控器**对着平台主板上的红外接收头按下任意按键，在界面中会显示该按键的相应信息。如**图5.1**所示：

![红外按键事件](chapter5/experiment08/ch05_08_ui.png)   





