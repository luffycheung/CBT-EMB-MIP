# 实验十一. 基于IIC接口的FM收音机开发实验

----------
##  实验目的
- 掌握使用IIC对FM模块进行控制;

##  实验环境
* 硬件：CBT-EMB-MIP 实验平台,PC机,mini USB数据线;
* 软件： Android Studio 2.2 或更高版本， Android Plugin for Gradle 版本 2.2.0 或更高版本，CMake;

##  实验内容
- 编写NDK程序，实现控制收音机模块。


##  实验原理

本实验中使用的收音机芯片为飞利浦生产的`TEA5767`。

### 硬件连接

![连接电路](chapter5/experiment11/tea5767.png)

**图4.1** FM模块(M1)与主板的连接电路

FM模块（图中的M1）通过J3与平台主板相连接，其中，J3的3、4脚是I2C通信接口。芯片地址C0，I2C通信时，取芯片地址的高7位（0x60）。TEA5767数据传输直接向芯片顺序发送1个地址跟5个数据，不需要知道每一个寄存器的地址。

### IIC编程接口

驱动实现采用IIC设备驱动的方式，同三轴加速度。我们可以通过`open`函数打开IIC的设备文件，通过`ioctl`函数设定要访问从设备的地址，然后就可以通过`read`和`write`或者`ioctl`函数完成对IIC设备的读写操作。

- 打开适配器对应的设备节点
如果使用read()跟write(),必须提前设置从机地址，运用
`ioctl(fd, I2C_SLAVE,addr)`；此次实验没有用到read()跟write()，所以不用设置：
```
        fd = open(I2C_DEV, O_RDWR);
        if(fd < 0)
        {
            printf("####i2c test device open failed####\n");
            return (-1);
        }   
        res = ioctl(fd,I2C_TENBIT,0);   //在读写时，取地址的高七位
        if(res<0)
        {
            printf("ioctl:%s\n",strerror(errno));
            return;
        }
        ioctl(fd,I2C_TIMEOUT,1);/*超时时间*/
        ioctl(fd,I2C_RETRIES,2);/* 收不到ACK时重复次数*/
```
- 向FM写数据
```
void write_FM(int fd)
{
             /***write data to tea5767**/
                 tea_data.nmsgs=1;
                 (tea_data.msgs[0]).flags=0; //write

                 int ret = ioctl(fd,I2C_RDWR,(unsigned long)&tea_data);
                 if(ret<0)
                 {
                    printf("ioctl write data:%s\n",strerror(errno));
                    return ;
                 }
                bzero((tea_data.msgs[0]).buf,5*sizeof(char));//每次写完清空数据，防止污染读取的数据
                usleep(10);
}
```
- 从FM读数据
读数据时，至少在写入数据40ms之后再读取，不然数据不稳定 
```
void read_FM(int fd)
{
/******read data from tea*******/
tea_data.nmsgs=1;
(tea_data.msgs[0]).len=0;//每次传送要5个字节
(tea_data.msgs[0]).addr=0x60;// tea5767地址
(tea_data.msgs[0]).flags=0;//write
(tea_data.msgs[0]).buf[0]=0xc1; 
write_FM(fd);//先写入要读取的地址
tea_data.nmsgs=1;
(tea_data.msgs[0]).len=5;//每次传送要5个字节
(tea_data.msgs[0]).addr=0x60;// tea5767地址
(tea_data.msgs[0]).flags=I2C_M_RD;//read
(tea_data.msgs[0]).buf[0]=0; 
(tea_data.msgs[0]).buf[1]=0; 
(tea_data.msgs[0]).buf[2]=0; 
(tea_data.msgs[0]).buf[3]=0; 
(tea_data.msgs[0]).buf[4]=0; 
int ret = ioctl(fd,I2C_RDWR,(unsigned long)&tea_data);
if(ret<0)
{
   printf("ioctl read data:%s\n",strerror(errno));
     return ;
    }
unsigned int pll_h,pll_l;
pll_h=(tea_data.msgs[0]).buf[0];
pll_l=(tea_data.msgs[0]).buf[1];
pll_h &= 0x3f;
pll = (pll_h*256)|pll_l;
get_frequency();
} 
```
- 读写数据结构体
```
tea_data.nmsgs=1;//消息的数量
(tea_data.msgs[0]).len=5;//每次传送要5个字节
(tea_data.msgs[0]).addr=0x60;// tea5767地址 高七位
(tea_data.msgs[0]).flags=I2C_M_RD;//read 。//0是write，
(tea_data.msgs[0]).buf[0]=0; //5个数据
(tea_data.msgs[0]).buf[1]=0; 
(tea_data.msgs[0]).buf[2]=0; 
(tea_data.msgs[0]).buf[3]=0;
(tea_data.msgs[0]).buf[4]=0;
```
- 自动搜台
```
void auto_search(int fd)
{
    printf("\nsearching....\n");
    frequency=max_freq;
    bzero(freq_array,80*sizeof(char));
    count=0;
    while(1)
    {
        while(((tea_data.msgs[0]).buf[0])<0x30)//信号强度，可调节
        {
            get_pll();
            bzero((tea_data.msgs[0]).buf,5*sizeof(char))
            (tea_data.msgs[0]).buf[2]=0x20;//the data to write
            (tea_data.msgs[0]).buf[0]=(pll/256) ;
            (tea_data.msgs[0]).buf[0] |= 0x40;// 
            (tea_data.msgs[0]).buf[1]=pll%256;//the data to write   
tea_data.msgs[0]).buf[3]=0x11;//the data to write       
(tea_data.msgs[0]).buf[4]=0x00;//the data to write
write_FM(fd);
bzero((tea_data.msgs[0]).buf,5*sizeof(char));
read_FM(fd)；
}
if(frequency>min_freq)
{       
freq_array[count]=frequency;    
count++;
}else
    ;
usleep(100);
bzero((tea_data.msgs[0]).buf,5*sizeof(char));
frequency-=100; 
if(frequency<min_freq)
{
printf("END search .all %d channel.\n",count);
printf("Input the 1~%d to choose the channel to listen\n",count);
into_clience(fd);
        return;
    }
}
}
```

## 实验步骤

说明：将FM模块插入到EXPORT扩展接口上。模块上的`J1`两个引脚为天线接头，可插上杜邦线提高信号质量。

### 导入工程源码

1.  打开Android Studio，从菜单栏选择 **File \> Open**。
2.  弹窗中浏览选择光盘src目录下的Gradle工程 **CH05_NDK** ,点击**OK**导入。
3.  等待工程构建完成后，在工具栏中的*Android App*列表中选择本实验例程**CH05_11_FM**。

### 演示运行

- 平台主板通过miniUSB线连接电脑后，点击 **Run**
![从菜单栏运行应用](https://developer.android.com/studio/images/buttons/toolbar-run.png)
运行程序。
- 界面启动后会自动搜台，之后点击下方的按键即可收听指定频段的电台节目。如**图5.2.1**所示：

![FM](chapter5/experiment11/ch05_11_ui.png)

**图5.2.1** FM收音机
