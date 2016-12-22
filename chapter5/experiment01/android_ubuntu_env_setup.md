# 实验一. Android Ubuntu编译环境搭建

----------
##  实验目的
- 掌握Ubuntu 12.04 操作系统的安装;
- 掌握交叉编译器的安装;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机;
* 软件：VMware;

## 实验内容
* 虚拟机上安装Ubuntu 12.04操作系统;
* 建立Android系统开发环境，安装Android配套的交叉编译器;

## 实验步骤

### 系统镜像下载

Ubuntu12.04 LTS官网下载地址：[http://www.ubuntu.org.cn/download/desktop](http://www.ubuntu.org.cn/download/desktop)  

选择64位版本下载（Android光盘\src\chapter05\experiment01目录已经提供，建议用户使用的软硬件环境与本书保持一致，实际硬件差异需要具体灵活设置和下载相应的资源包）。

### 在VMware中安装Ubuntu系统

* 打开VMware，点击File菜单选择New Virtual Machine。在弹出的窗口选择Typical 典型安装，点击下一步，如**图4.2.1**所示

![](/chapter5/experiment01/4.2.1.png)

**图4.2.1** 

* 选择ISO镜像的目录，点击下一步，如**图4.2.2**所示

![](/chapter5/experiment01/4.2.2.png)

**图4.2.2**

* 输入Full name,user name,password ,点击下一步，如**图4.2.3**所示

![](/chapter5/experiment01/4.2.3.png)

**图4.2.3**

* 点击Browse选择安装路径，点击下一步，如**图4.2.4**所示

![](/chapter5/experiment01/4.2.4.png)

**图4.2.4**

* Ubuntu需要硬盘空间大约5G，Android源码编译后占硬盘空间大约29G , linux内核编译后约1.9G，因此建议提前为Ubuntu12.04 的安装预留大约40G的空间，点击下一步，如**图4.2.5**所示

![](/chapter5/experiment01/4.2.5.png)

**图4.2.5**

* 点击`Customize Hardware` , 做一些启动基本配置，如图4.2.6和图4.2.7所示

![](/chapter5/experiment01/4.2.6.png)

**图4.2.6**

（注意： 推荐内存大于1G，否则因为内存大小限制将无法编译Android文件系统）

![](/chapter5/experiment01/4.2.7.png)

**图4.2.7**

* 点击`finish`开始安装，如**图4.2.8**所示

![](/chapter5/experiment01/4.2.8.png)

**图4.2.8**

* Ubuntu12.04使用Easy Install工具自动连接互联网安装部分资源包和软件，需一定安装时间。可点击skip跳过，进入系统之后再另外更新安装。如**图4.2.9**所示

![](/chapter5/experiment01/4.2.9.png)

**图4.2.9**

* 安装完成之后虚拟机会自动重启，进入`VMware Easy Install`命令行登录界面，如**图4.2.10**所示

![](/chapter5/experiment01/4.2.10.png)

**图4.2.10**

* 点击VMware中的Power Off按键强制关闭AndroidDev虚拟机，如**图4.2.11**所示

![](/chapter5/experiment01/4.2.11.png)

**图4.2.11**

* 点击`Edit virtual machine settings`，在弹出来的设置窗口中remove掉CD/DVD（IDE）设备。把Floppy中的Connection方式选择为Use physical drive：-> Auto detect。之后点击Power on启动虚拟机。如图4.2.12所示

![](/chapter5/experiment01/4.2.12.png)

**图4.2.12**

* 在[ubuntu login:]处输入用户名cbt，回车后输入之前步骤中设置的密码进行登录。在cbt@ubuntu:~$后输入startx命令进入系统图像界面，如**图4.2.13**所示

![](/chapter5/experiment01/4.2.13.png)

**图4.2.13**

* 进入系统后，点击VMware菜单中的VM -> Install VMware Tools选项。弹出对话框选择yes，如**图4.2.14**所示

![](/chapter5/experiment01/4.2.14.png)

**图4.2.14**

* 点击右上角的图标选择Log out 退出系统，如**图4.2.15**所示

![](/chapter5/experiment01/4.2.15.png)

**图4.2.15**


* 执行如下命令:
```
$ sudo mv /etc/issue.backup /etc/issue
$ sudo mv /etc/rc.local.backup /etc/rc.local
$ sudo mv /opt/vmware-tools-installer/lightdm.conf /etc/init
```

之后再执行```sudo reboot``` 命令重启或者执行```sudo start lightdm```命令便可以进入图形登录界面。如**图4.2.16**所示

![](/chapter5/experiment01/4.2.16.png)

**图4.2.16**

* 进入系统后，按照下面提示继续安装VMware Tools工具。打开Terminal终端，进入VMware Tools光盘目录将其解压到本地目录/tmp下。命令如下：
```
$ cd /media/VMware Tools
$ tar xvf VMwareTools-8.8.0*.tar.gz -C /tmp
$ cd /tmp/VMware-tools-distrib/
```
执行：

```
$sudo ./VMware-install.pl 
```

* 接下来的安装步骤中，一直按回车键确认安装即可。此步骤可能需要一定时间，如**图4.2.17**所示

![](/chapter5/experiment01/4.2.17.png)

**图4.2.17**

* 在Terminal命令行终端中中连接互联网更新之前skip过的内容，执行如下命令：
```
$ sudo apt-get upgrade
$ sudo apt-get update
```

* 安装vim编辑器：
```
$ sudo apt-get install vim-gtk
```

### 安装Android 配套的交叉编译器

* 打开Terminal，输入cp 之后把Android光盘\src\chapter05\experiment01中的arm-linux-gcc-4.5.1.tar.gz压缩包拖拽拷贝到 ~/目录下，如**图4.3.1**所示

![](/chapter5/experiment01/4.3.1.png)

**图4.3.1**

* 执行解压命令
```
$ tar  xvf  arm-linux-gcc-4.5.1.tar.gz 
```

执行该命令，将把arm-linux-gcc 安装到~/toolschain/4.5.1目录下。

* 把编译器的路径加入到系统环境变量，运行命令
```
$ sudo gedit ~/.bashrc 
```
编辑~/.bashrc文件，修改最后一行为 
```
export PATH=$PATH:~/toolschain/4.5.1/bin
```

注意路径一定要依据上面解压步骤更改正确，否则将不会有效，如**图4.3.2**所示，保存退出。

![](/chapter5/experiment01/4.3.2.png)

**图4.3.2**

* 执行命令：
```
 source ~/.bashrc
```

* 在命令行输入```arm-linux-gcc -v ```，出现如下打印信息，则说明交叉编译环境已经成功搭建。
```
gcc version 4.5.1 (ctng-1.8.1-FA) 
```

注：若由于切换root用户无法在终端中输入或找到arm-linux-gcc命令，可手动执行
source ~/.bashrc命令导入环境变量，之后即可输入上述命令查看。





