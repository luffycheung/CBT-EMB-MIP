# 实验一. Android SDK Windows系统环境搭建

---

## 实验目的
* 掌握在Windows下，Android集成开发环境的搭建;
* 熟悉集成开发环境;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机;
* 软件： Android Studio ,JDK;


## 实验内容
- 搭建 Android 集成开发环境;
- 创建 Android 模拟器AVD;


**Android国内官方网址：**[https://developer.android.google.cn](https://developer.android.google.cn)

## 实验步骤

**本地安装包路径：**`光盘\tools\Android Windows环境搭建\`。
```
└─Android Windows环境搭建
        android-studio-bundle-145.3537739-windows.exe //包含 Android SDK
        jdk-8u102-windows-i586.exe  //32位
        jdk-8u102-windows-x64.exe  //64位
        studio-install-windows.mp4 //安装视频
        usb_driver.rar //adb驱动
```
### Java开发环境安装
- 启动`jdk-*.exe`，根据安装向导指示安装。
- 配置环境变量。选择**“Start”菜单 \> 电脑 \> 属性 \> 高级系统选项**。然后打开**“高级”选项卡 \> 环境变量**进行配置。
    + 在“系统变量”新建一个变量名为`JAVA_HOME`的变量，变量值为你本地java的安装目录，这里为：`C:\Program Files\Java\jdk1.8.0_31`，设置这个的目的是作为下面两个环境变量的一个引用
    + 在“系统变量”选项区域中查看PATH变量，如果不存在，则新建变量PATH，否则选中该变量，单击“编辑”按钮，在“变量值”文本框的起始位置添加`“%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;”` 
    + 在“系统变量”选项区域中查看`CLASSPATH`变量，如果不存在，则新建变量`CLASSPATH`，否则选中该变量，单击“编辑”按钮，在“变量值”文本框的起始位置添加`“.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;”`。<br>
    判断是否安装成功：<br>
    在cmd命令行中输入命令：
    ```
    java -version
    ```
    出现如下打印信息说明配置成功：
```
    java version "1.8.0_31"
    Java(TM) SE Runtime Environment (build 1.8.0_31-b13)
    Java HotSpot(TM) 64-Bit Server VM (build 25.31-b07, mixed mode)
```

### 安装Android Studio

- 启动`android-studio-bundle-*-windows.exe` 文件。
- 根据安装向导的指示安装Android Studio和所有所需的SDK工具。
    + **建议：**安装向导选择SDK安装路径时，建议选择安装在剩余空间较多的磁盘，且选择一个较短且易找到的路径。
<video id="video" style="margin:20px 0" onclick="this.play()" controls="" preload="none" width='100%' height='100%' poster="https://developer.android.google.cn/images/develop/hero_image_studio5_2x.png">
      <source id="mp4" src="https://storage.googleapis.com/androiddevelopers/videos/studio-install-windows.mp4" type="video/mp4">
</video>

- 如果想在命令提示符中使用SDK命令，那么还需要把SDK下的`tools`目录和`platform-tools`目录添加到环境变量中。
- 解决Android Studio启动报错问题
    + 进入刚安装的 Android Studio 目录下的bin目录。找到 `idea.properties` 文件，用文本编辑器打开。
    + 在 `idea.properties` 文件末尾添加一行：`disable.android.first.run=true`，然后保存文件。
    + 关闭 Android Studio 后重新启动，便可进入界面。

如果有新的工具和其他 API 可供使用，Android Studio 将弹出提示，或者您也可以点击 Help > Check for Update 检查更新。

### 创建和管理虚拟设备(AVD)

#### 依赖项和先决条件
--------
-   Android Studio 2.0 或更高版本
-   SDK Tools 25.0.10 或更高版本
-   通过 **Tools** \> **Android** \> **Enable ADB Integration** 启用 ADB
    集成

利用 Android Virtual Device (AVD) 定义，您可以定义您想要在 [Android
Emulator](https://developer.android.google.cn/tools/devices/emulator.html)
中模拟的 Android 电话、平板电脑、Android Wear 或 Android TV
设备的特征。AVD Manager 可以帮助您轻松地创建和管理 AVD。

要高效地测试应用，您需要创建 AVD，AVD
能够模拟应用设计支持的每个设备类型。例如，我们建议您针对等于或高于为
[`minSdkVersion`](https://developer.android.google.cn/guide/topics/manifest/uses-sdk-element.html)
所指定值的每个 API 级别创建 AVD。通过使用高于应用所需的 API
级别进行测试，您可以确保应用在用户下载系统更新后的向前兼容性。

#### 关于 AVD 
--------

AVD 包括硬件配置文件、系统映像、存储区域、皮肤和其他属性。

硬件配置文件定义了设备出厂时的特征。AVD Manager
会预加载特定的硬件配置文件，如 Nexus
电话设备，您可以根据需要定义和导入硬件配置文件。如果需要，您可以替换 AVD
中的一些设置。

AVD Manager 会提供一些推荐，帮助您为 AVD 选择系统映像。利用 AVD
Manager，您也可以下载系统映像，一些映像具有应用需要的插件库，如 Google
API。请注意，x86 系统映像在模拟器中运行得最快。

AVD在您的开发机器上有专门的存储区域，可以存储设备用户数据，如已安装的应用和设置，以及模拟SD 卡。

模拟器皮肤指定了设备的外观。AVD Manager会提供一些预定义的皮肤。您也可以自己定义皮肤，或者使用第三方提供的皮肤。

就像在真实设备上一样，对于要使用在 AVD
中定义的某些功能（例如相机）的应用，应用清单中必须具有相应的
[`<uses-feature>`](https://developer.android.google.cn/guide/topics/manifest/uses-feature-element.html)
设置。

观看下面的视频，快速地概括了解模拟器的部分功能。
<video id="video" style="margin:20px 0" onclick="this.play()" controls="" preload="none" width='100%' height='100%' poster="https://developer.android.google.cn/images/tools/e-emulator.png">
      <source id="mp4" src="https://storage.googleapis.com/androiddevelopers/videos/studio-emulator-overview.mp4" type="video/mp4">
</video>

#### 查看和管理您的 AVD
------------------

AVD Manager 让您可以在一个位置管理 AVD。

要运行 AVD Manager，请执行以下操作：

-   在 Android Studio 中，选择 **Tools** \> **Android** \> **AVD
    Manager**。
-   点击工具栏中的 AVD Manager ![AVD Manager
    图标](https://developer.android.google.cn/studio/images/buttons/toolbar-avd-manager.png)。

将显示 AVD Manager。

![AVD Manager 主窗口AVD Manager
主窗口](https://developer.android.google.cn/images/tools/avd-main.png)

AVD Manager 会显示您已定义的任何 AVD。首次安装 Android Studio
时，会创建一个 AVD。如果已为 Android Emulator 24.0.*x* 或更低版本定义了
AVD，您需要重新创建它们。

从此页面中，您可以进行以下操作：

-   定义新的 [AVD](#createavd) 或[硬件配置文件](#createhp)。
-   编辑现有 [AVD](#workingavd) 或[硬件配置文件](#workinghp)。
-   删除 [AVD](#workingavd) 或[硬件配置文件](#workinghp)。
-   [导入或导出](#importexporthp)硬件配置文件定义。
-   [运行](#emulator) AVD 以启动模拟器。
-   [停止](#emulator)模拟器。
-   [清除](#emulator)数据，然后从您首次运行模拟器的状态重新开始。
-   在磁盘上[显示](#workingavd)关联的 AVD `.ini` 和 `.img` 文件。
-   [查看](#workingavd) AVD 配置详细信息，您可以在提交给 Android Studio
    团队的任何错误报告中包含这些信息。

#### 创建 AVD
--------

您可以从头创建新的 AVD，也可以[复制 AVD](#copyavd)，然后更改一些属性。

要创建新的 AVD，请执行下列操作：

1.  在 AVD Manager 的 [*Your Virtual Devices*](#viewing) 页面中，点击
    **Create Virtual Device**。
2.  选择硬件配置文件，然后点击 **Next**。
3.  选择针对特定 API 级别的系统映像，然后点击 **Next**。
4.  根据需要更改 [AVD 属性](#avdproperties)，然后点击 **Finish**。

    点击 **Show Advanced Settings** 显示更多设置，如皮肤。

要从副本开始创建 AVD，请执行以下操作：

1.  从 AVD Manager 的 [*Your Virtual Devices*](#viewing)
    页面中，右键点击 AVD，然后选择 **Duplicate**。
2.  如果您需要在 [*System Image*](#systemimagepage) 和 [*Select
    Hardware*](#selecthardwarepage) 页面上进行更改，请点击 **Change** 或
    **Previous**。
3.  进行更改，然后点击 **Finish**。

#### 创建硬件配置文件
----------------

AVD Manager
会为常见设备提供预定义的硬件配置文件，这样您就可以轻松地将它们添加至您的
AVD
定义中。如果您需要定义不同的设备，则可以创建新的硬件配置文件。您可以从头定义新的硬件配置文件，也可以[复制硬件配置文件](#copyavd)。预加载的硬件配置文件无法编辑。

要从头创建新的硬件配置文件，请执行以下操作：

1.  在 [*Select Hardware*](#selecthardwarepage) 页面中，点击 **New
    Hardware Profile**。
2.  在 *Configure Hardware Profile*
    页面中，根据需要更改[硬件配置文件属性](#hpproperties)。
3.  点击 **Finish**。

要从副本开始创建硬件配置文件，请执行以下操作：

1.  在 [*Select Hardware*](#selecthardwarepage)
    页面中，选择硬件配置文件，然后点击 **Clone Device**。
2.  在 *Configure Hardware Profile*
    页面中，根据需要更改[硬件配置文件属性](#hpproperties)。
3.  点击 **Finish**。

#### 使用现有的 AVD 
--------------

从 [*Your Virtual Devices*](#viewing) 页面，您可以对现有的 AVD
执行以下操作：

-   要编辑 AVD，请点击 **Edit**
    ![](https://developer.android.google.cn/images/tools/studio-advmgr-actions-edit-icon.png)，然后[进行更改](#copyavd)。
-   要删除 AVD，请右键点击 AVD，然后选择 **Delete**。或者点击 **Menu**
    ![](https://developer.android.google.cn/images/tools/studio-advmgr-actions-dropdown-icon.png)，然后选择
    **Delete**。
-   要在磁盘上显示关联的 AVD `.ini` 和 `.img` 文件，请右键点击
    AVD，然后选择 **Show on Disk**。或者点击 Menu
    ![](https://developer.android.google.cn/images/tools/studio-advmgr-actions-dropdown-icon.png)，然后选择
    **Show on Disk**。
-   要查看 AVD 配置详细信息（您可以将其包含在提交给 Android Studio
    团队的任何错误报告中），请右键点击 AVD，然后选择 **View
    Details**。或者点击 Menu
    ![](https://developer.android.google.cn/images/tools/studio-advmgr-actions-dropdown-icon.png)，然后选择
    **View Details**。

#### 在 Android Emulator 中运行应用 
------------------------------

您可以从 Android Studio
项目中运行应用。或者，您也可以运行已经安装到模拟器上的应用，就像在设备上运行任何应用一样。

要在您的项目中启动模拟器并运行应用，请执行以下操作：

打开一个 Android Studio 项目并点击 **Run**
![](https://developer.android.google.cn/studio/images/buttons/toolbar-run.png)。

将显示 *Select Deployment Target* 对话框。

![“Select Deployment
Target”对话框](https://developer.android.google.cn/images/tools/e-selectdeploymenttarget.png)

如果您在对话框的顶部看到错误或警告消息，请点击链接，纠正问题或者了解更多信息。

**No USB devices or running emulators detected**
警告表示您当前未运行任何模拟器，或者检测到有硬件设备连接到您的计算机。如果您未将硬件设备连接到计算机或者已经运行模拟器，可以忽略此警告。

不过，您必须修正某些错误才能继续，例如某些 Hardware Accelerated
Execution Manager (Intel® HAXM) 错误。

在 *Select Deployment Target*
对话框中，选择一个现有的模拟器定义，然后点击 **OK**。

如果您未看到想要使用的定义，请点击 **Create New Emulator** 以启动 AVD
Manager。定义新的 AVD 后，在 *Select Deployment Target* 对话框中点击
**OK**。

如果您想要将此模拟器定义用作项目的默认设置，请选择 **Use same selection
for future launches**。

模拟器将启动并显示您的应用。

在模拟器中测试您的应用。


要关闭模拟器，请点击 Close
![“Close”图标](https://developer.android.google.cn/images/tools/e-iclose.png)。

模拟器设备会存储已安装的应用，因此，您可以根据需要再次运行。要移除应用，您需要将其卸载。如果您在相同的模拟器上重新运行项目，系统会使用新版本替换应用。

#### 在不运行应用的情况下启动 Android Emulator
-----------------------------------------

要启动模拟器，请执行以下操作：

1.  [打开 AVD
    Manager](https://developer.android.google.cn/tools/devices/managing-avds.html)。
2.  双击 AVD，或者点击 **Run**
    ![](https://developer.android.google.cn/studio/images/buttons/toolbar-run.png)。


关于Android Studio更详细使用请查阅官方文档：[https://developer.android.google.cn/studio/intro/index.html](https://developer.android.google.cn/studio/intro/index.html)

