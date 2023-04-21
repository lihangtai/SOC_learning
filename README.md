# SOC



## guideline



> KEY area：U-boot，linux内核，linux设备驱动，应用层项目
>
> 
>
> 知识体系复杂，不要钻在某一个子问题，应该先快速入门，边学边补：
>
> 基本认知》linux/工具基础》搭建环节》》APP开发基础 》驱动基础 》项目
>
> 所以，可以先掌握必备的APP基础、驱动基础，然后马上开始学习项目开发，以后想深入时，再去学习相关的专题。



-------------

需要理解的命令行工具：

gcc，make，arm-buildroot-linux-gnueabihf-gcc，gdb，cmake，objdump，fdisk，dmesg，mkfs

------------



### 目标

1. 交叉编译的使用 
2. 内核的配置与编译
3. 驱动基础：基础的驱动，比如GPIO、UART、SPI、I2C、LCD、MMC，高级的驱动，比如USB、PCIE、HDMI、MIPI、GPU、WIFI、蓝牙、摄像头、声卡。

  4.glibc的了解使用

5. 多线程



example：

一个IO的触发过程：

1.APP发起读的操作，若无数据则休眠

2.用户操作设备，产生硬中断

3.输入层驱动处理中断，转换为标准输入事件，向核心层汇报

4.核心层可以决定把输入事件转发给上面哪个handler来处理：

5.handler唤醒app，对输入事件进行处理

触发方式：

时不时进房间看一下：查询   简单，但是累

②进去房间陪小孩一起睡觉，小孩醒了会吵醒她：休眠-唤醒不累，但是妈妈干不了活了

③妈妈要干很多活，但是可以陪小孩睡一会，定个闹钟：poll方式

要浪费点时间，但是可以继续干活。

妈妈要么是被小孩吵醒，要么是被闹钟吵醒。

④妈妈在客厅干活，小孩醒了他会自己走出房门告诉妈妈：异步通知

妈妈、小孩互不耽误。



## configuration

### 板子与linux主机通信方式

1:网线连接：使用ifconfig分别为linux有线网口，板子有线网口设置同一网段的ip地址

ip address：11192.168.1.100

linux端使用ssh则可以直接访问

2.串口通信：通过UART链接，使用putty连接

~~~linux
1.
ls -l /dev/tty*     
// tty设备：前生打字机。现在允许用户输入，在计算机上查看输出，通常与串口相连接的设备。实体设备（USB tty），软件设备（终端，终端模拟器
Note：串口：UART，RS-232   /dev是一个虚拟文件系统，用于访问设备和设备文件，包含了所有可用的文件（没有对应驱动可能没有显示）
2.
chmod 666 /dev/ttyUSB0     //暂时打开权限

~~~



### 使用SDK配置，启动SOC

一个嵌入式SDK包含：交叉编译工具链，操作手册，源代码（库和头文件），调试烧录工具/脚本

1.安装编译环境(shell)

启动编译环境，才能使用aarch64-pock-linux-gcc

>重启linux或者打开新的shell都需要重新执行启动环境。（同一个shell属于一个进程组）

2.解压，按照makefile编译生成镜像/内核/其他程序



3.烧写到sd卡中，soc使用sd卡启动板子

case one：全新的sd卡

~~~linux
lsblk  //查看挂载情况，找到sdx
blkid  //检查sd卡文件类型
fdisk dev/sdx //进行分区
mkfs ext4 /dev/sdx   //对分区进行格式化
mount /dev/sdx /xxpath  //挂载
//
可以直接分区再格式化
~~~



case two：已经有内容

先格式化再分区