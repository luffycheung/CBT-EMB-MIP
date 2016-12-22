# 实验二. Android ADB调试实验

## **实验目的**

-   掌握在Windows下，Android ADB 环境的搭建
-   了解Android SDK的开发环境
-   熟悉Android 应用程序的安装, 文件的传输

## **实验设备**

* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio;

## **实验内容**

-   搭建Android ADB 环境，安装MiniUSB驱动，通过adb 访问android平台上的文件系统
-   通过adb安装Android 应用程序
-   通过adb传输文件

## **实验步骤**


### 驱动安装

-   打开Android平台设备，插入MiniUSB线与PC相连。
-   右击“我的电脑”-\>“属性”-\> “硬件” ，打开“设备管理器”，如**图4.1.1**所示

![](/chapter2/experiment02/device_manager.png)

**图4.1.1** 设备管理器

-   右键“S5P
    OTG-USB”，选择安装驱动，选择“从列表或指定位置安装（高级）”，如**图4.1.2**所示，点击“下一步”

![](/chapter2/experiment02/driver_wizard.png)

**图4.1.2** 安装ADB驱动

-   点击“浏览”，在Android SDK
    安装路径中选择USB驱动程序的路径，这里选择`“D:\\Android\\android-sdk-windows\\extras\\google\\usb\_driver”`(**以用户实际SDK安装解压路径为准**),选择路径后点击“下一步”进行安装，如**图4.1.3**所示

![](/chapter2/experiment02/chose_path.png)

**图4.1.3** 安装ADB驱动

-   点击“完成”，如**图4.1.4**所示

![](/chapter2/experiment02/setup_complete.png)

**图4.1.4** 安装ADB驱动

-   再次打开设备管理器，如**图4.1.5**所示，“Android Composite ADB
    Interface”驱动安装成功

![](/chapter2/experiment02/adb_interface.png)

**图4.1.5** 安装ADB驱动

### ADB环境配置及测试

-   将ADB命令所在的路径添加到Path环境变量中
    + 右击“我的电脑”-\>“属性” ，再选择“高级系统设置”选项。
    +  点击“环境变量”选项。 
    +  在“系统变量”中，找到Path环境变量，双击它，在变量值前面追加一下内容：“D:\\Android\\android-sdk-windows\\platform-tools；”(以用户实际SDK安装解压路径为准)，注意后面有一个分号，如**图4.2.1**所示
      
![](/chapter2/experiment02/adb_env.png)

**图4.2.1** 配置ADB环境变量

- 
    + 点击“确定”完成环境变量设置。
    + 在cmd.exe上按回车来启动DOS窗口，输入adb按回车，显示如**图4.2.2**所示则表明环境变量设置OK。

![](/chapter2/experiment02/adb_cmd.png)

**图4.2.2** 命令行

-   输入以下命令，查看设备连接状态（开发板已经用miniUSB和PC连接）
```
$ adb devices
```

若显示如下信息所示则表示成功连接到Android平台设备。

```
List of devices attached
* daemon not running. starting it now on port 5037 *
* daemon started successfully *
SuperIOT4Android        device

```

### ADB安装软件

-   在Windows目录下新建一个文件夹（D:\\hello），把Android光盘\\src\\chapter02\\experiment02\\hello.apk复制到D：\\hello目录下

-   以安装D:\\hello\\hello.apk的程序为例，在DOS窗口输入以下命令进行安装
```
> adb install D: \\hello\\hello.apk
```
-   DOS窗口如**图4.3.1**所示

![](/chapter2/experiment02/adb_install.png)

**图4.3.1** 命令行安装

-   查看Android 平台，找到“hello”应用程序，如**图4.3.2**所示

![](/chapter2/experiment02/adb_hello.png)

**图4.3.2** hello程序


### ADB传输文件

-   确保A9网关已启动且用miniUSB线与PC端连接

-   打开光盘`tools`目录下的串口终端`putty`，进入Android系统根目录，输入“su”命令获取root权限。

-   将根目录挂载成读写权限，输入如下命令：
```
# mount -o remount rw, /
```
-   新建文件夹source
```
# mkdir source
```

-   以上传D:\\hello\\hello.apk的程序为例，在DOS窗口输入以下命令进行传输
```
\> adb push D: \\hello\\hello.apk source
```
-   文件上传成功，即可以在Android系统文件夹目录下看到下载的文件DOS窗口如**图4.4.1**所示

![](/chapter2/experiment02/adb_push_file.png)

**图4.4.1** ADB传输文件


关于ADB更详细使用请查阅官方文档：[https://developer.android.google.cn/studio/command-line/adb.html](https://developer.android.google.cn/studio/command-line/adb.html)