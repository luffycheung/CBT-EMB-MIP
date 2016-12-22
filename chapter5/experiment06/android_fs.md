# 实验六. Android 文件系统编译实验

----------
##  实验目的
- 掌握Android 文件系统的搭建过程;
- 熟悉Android 文件系统的初始化脚步及启动过程;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机;
* 软件：VMware Workstation + Ubuntu 12.04 + JDK编译环境;

## 实验内容

* 安装编译环境：JDK及需要的开发包;
* 编译Android文件系统源代码;
* 制作根文件系统的镜像

## 实验步骤

### 安装编译环境

将Android光盘\src\chapter05\experiment06下的“ubuntu”文件夹拷贝到“/tmp”目录下

#### 安装JDK
启动Ubuntu系统后，按下<kbd>ctrl</kbd>+<kbd>alt</kbd>+<kbd>t</kbd>打开一个终端。
输入命令

```
sudo apt-get install openjdk-7-jdk
```
输入密码后安装。

- 配置环境变量

执行```sudo gedit /etc/profile``` 文件末尾添加：
```
JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export JAVA_HOME
export PATH
```

执行如下命令刷新环境变量：
```
sudo source /etc/profile
```

验证java是否配置完成，可以在终端输入```java --version```，如果能够正确的打印出java版本就表示成功。

#### 安装编译源码所需要的工具集

Android文件系统的编译需要安装一些开发包，确保虚拟机已经联网，在终端执行install-devel-packages.sh脚本：
```
# cd /tmp/ubuntu/
# chmod 755 install-devel-packages.sh
# ./install-devel-packages.sh
```

中途会询问是否下载软件包，输入Y回车即可。

### 编译Android 文件系统源码

注：要求最低内存1.5G左右，硬盘40G以上。

- 将Android光盘\src\chapter05\experiment06下android文件系统源码包“android-4.2.2_r1.tar.gz”复制到Ubuntu 12.04 的目标目录下（例如本次实验虚拟机目标目录为/home/cbt/Android），然后解压：
```
$ suod -s 
//输入密码后切换成root权限
# tar jxvf android-4.2.2_r1.tar.gz
//然后进到目录android-4.2.2_r1下
# cd android-4.2.2_r1
```
- 设置相关编译环境变量，执行：
```
#  .  setenv ;“.”后面有一空格
```
- 开始编译
```
#make -j3
```
注：若出现如下错误提示：
```
build/core/config.mk:268: *** Error: could not find jdk tools.jar, please install JDK6, which you can download from java.sun.com。 停止。
```
可执行如下命令，将JDK导入到环境变量中：
```
# source /etc/environment
```

编译源代码需要等待很长的时间，请耐心等待。编译完成后系统会自动生成相关的文件系统镜像：.
`out/target/product/tiny4412/`可在这里找到各个部分，如**图4.3.1**所示：

![](/chapter5/experiment06/4.3.1.png)

**图4.3.1**

### 制作文件系统镜像

- 运行gen-img.sh脚本，从编译完成的Android文件系统源码中提取我们需要的目标文件系统，最后生成img镜像,如**图4.4.1**所示：
```
#. gen-img.sh   // . 后面有空格，提取并生成目标文件系统镜像
```

![](/chapter5/experiment06/4.4.1.png)

**图4.4.1**


- 将本章实验二中生成的zImage和本实验生成的各个img镜像烧写到Android平台上（Android系统如何安装详见Android光盘\IMG\Cortex-A9系统烧写说明.pdf），开发板上电，如**图4.4.2**所示

![](/chapter5/experiment06/4.4.2.png)

**图4.4.2**

