#  SOC



## guideline



> Key area：U-boot，linux内核（linux各子系统驱动，现成的应用（ls，cd），各种配置），应用层应用
>
> 把开发的所有软件打包成img：buildroot        烧写到板子上或者OTA
>
> 
>
> 
>
> 知识体系复杂，不要钻在某一个子问题，应该先快速入门，边学边补：
>
> 基本认知》linux/工具基础》搭建环节》》APP开发基础 》驱动基础 》项目
>
> 所以，可以先掌握必备的APP基础、驱动基础，然后马上开始学习项目开发，以后想深入时，再去学习相关的专题。



-------------

可能需要理解宏观知识：

1.命令行工具的使用：gcc，make，交叉编译gcc，gdb，cmake，objdump，fdisk，dmesg，mkfs, systemctl，inxi, dmesg，mknod ...



2.交叉编译链的配置和使用 （使用前，要提前运行环境）



3.内核的配置，编译（make menuconfig）与版本替换



4.协议以及驱动的认知：基础的驱动，比如GPIO(控制）、UART(数据传输）、SPI、I2C、LCD、MMC，高级的驱动，比如USB、PCIE、HDMI、MIPI、GPU、WIFI、蓝牙、摄像头、声卡。



 5.标准系统库的使用：glibc，gcc，libstdc++（使用man 查询文档，标准化使用内核态的功能）



6.多线程



7.板级通信，板间通信



8.性能优化：内存泄漏，进程管理 （使用ebpf）

------------



linux中第一个启动的进程是init/systemd，systemd是linux中用于管理系统服务和进程的框架/守护进程（后台运行：用户态），以并行的方式启动它们，还提供了系统日志，用户管理，权限管理，网络，文件挂载等。（ststemctl是操作systemd的命令行工具）



文件位置：/proc/devices, /dev, /sys/class, /sys/devices， sys/bus

   

## MCU Configuration过程



### 板子与linux主机通信方式

#### 1:网线连接：使用ifconfig分别为linux有线网口，板子有线网口设置同一网段的ip地址

ip address：192.168.1.200

soc：192.168.1.100

soc 直接连接 linux主机 ：网线连接，使用静态路由：把网口的IP固定

>通过ifconfig eth0 192.168.1.100 netmask 255.255.255.0 up 进行配置只能暂时生效



ubuntu端：/etc/network/interface已经没有使用，而是通过/etc/netplan进行配置

配置后，sudo netplan apply生效





soc端：没有/etc/network/interface，没有/etc/netplan

使用

ip addr add 192.168.1.200/24 dev eth0 进行配置



> :soc+路由器+linux：可以使用有线或者无线连接，路由器开启DHCP
>
> linux端使用ssh则可以直接访问









#### 2.串口通信：通过UART链接，使用putty串口连接

~~~linux
1.
ls -l /dev/tty*     //查看设备节点的权限
// tty设备：前身打字机。现在允许用户输入，在计算机上查看输出，通常与串口相连接的设备。实体设备（USB tty），软件设备（终端，终端模拟器
Note：串口：UART，RS-232   /dev是一个虚拟文件系统，用于访问设备和设备文件，包含了所有可用的文件（没有对应驱动可能没有显示）
2.
chmod 666 /dev/ttyUSB0     //暂时打开权限

3.使用putty直接与tty设备建立连接
使用serial连接
~~~





### 开启NFS(文件共享)

linux安装nfs，把某目录配置为nfs，然后使用soc将该目录挂载到自己的目录中进行访问

~~~linux

1.先确定设备之间可以连通（处于一个局域网）
2.sudo apt-get install nfs-kernel-server
3. sudo mkdir /home/myshare
4.sudo vim /etc/exports
5./home/myshare  *(rw,sync,no_subtree_check)
6.sudo systemctl restart nfs-kernel-server

soc：
sudo mount ip:/home/myshare /mnt/location
~~~







### 新板子提供SDK，进行配置并启动SOC

一个嵌入式SDK包含：交叉编译工具链，操作手册，源代码（库和头文件），调试烧录工具/脚本，内核

1.安装编译环境(shell)

启动编译环境，才能使用aarch64-pock-linux-gcc

>重启linux或者打开新的shell都需要重新执行启动环境。（同一个shell属于一个进程组）

2.解压，按照makefile编译生成镜像/内核/其他程序


3.烧写镜像到sd卡中，soc使用sd卡/emmc启动板子

~~~linux
相关指令：

lsblk  //查看挂载情况，找到sdx
blkid  //检查sd卡文件类型
fdisk dev/sdx //进行分区
mkfs ext4 /dev/sdx   //格式化
mount /dev/sdx /xxpath  //挂载


~~~

case one：全新的sd卡

case two：已经有内容

先格式化再分区





### **编译内核**

1.配置编译内核 （顺便dtb，driver module ---》 dtb /lib）

>linux kernel,dtb,lib的关系：Linux Kernel通过驱动程序来控制硬件设备，其中驱动程序可以直接编写在内核中，也可以以模块的形式加载。驱动程序需要知道硬件设备的地址、中断、时钟等信息，这些信息可以通过读取DTB文件来获取。应用程序可以使用库来访问硬件设备，这些库通常使用Linux Kernel提供的驱动程序来控制硬件设备。
>
>下载的linux kernel是否包含：下载的Linux Kernel包中只包含内核代码和相关工具，不包含DTB和库。DTB文件是针对特定硬件平台生成的，它描述了设备的硬件配置和资源分配等信息。下载的Linux Kernel包中只包含内核代码和相关工具，不包含DTB和库。DTB文件是针对特定硬件平台生成的，它描述了设备的硬件配置和资源分配等信息。因此，DTB文件需要根据具体的硬件平台进行生成，一般由硬件厂商或开发者根据硬件平台的特性进行制作。DTB是在linux配置编译时生成。



基于makefile：

配置      make config  // make menuconfig    配置完成后会生成.config文件

> 驱动程序中头文件来自于内核，依赖于内核源码，要进行内核配置，然后编译内核。驱动和内核是配套的。

编译：

make  / make all（全编译）

make clean

make install

make <target>  // 快速编译

make <module>

make -j4 //多核心运行

2.移植到板子上 

~~~linux
连接串口，设置开发版启动模式
1.将新的内核镜像文件和相应的设备树文件（DTB）拷贝到目标设备的/boot下
cp /mnt/ZImage /boot  //
cp /mnt/xx.dtb /boot  //
cp /mnt/lib/modules /lib -rfd //更新模块   （覆盖目录）

2.修改引导配置文件，将新的内核镜像文件和设备树文件设置为启动时使用的内核和设备树
 sudo nano /boot/config.txt
 kernel=zImage
 device_tree=bcm2708-rpi-b.dtb
 
sync 
reboot






编译好的内核一般是vmlinux或者bbzimage文件，把他们复制到/boot下，修改grub的配置文件 
~~~





### gcc

预处理，编译，汇编，链接

-c：汇编，但不链接

-o：生成 xx

-I：指定头文件目录 （某人的头文件是放在/usr中

-L：链接时候库文件

>
>
>静态库：编译后不链接生成xx.o, 使用ar压缩。-》xx.a
>
>动态库：.so文件

当有多个.c文件和头文件时候，应该把编译和链接的过程分开，集中在一起销量不高



### makefile

~~~linux
// main.c and test.c

test: main.o test.o
	gcc -o test main.0 test.o
	
main.o: main.c
	gcc -c -o main.o main.c

test.o: test.c
	@gcc -c -o test.o test.c   ///@则运行时，不会显示本身

后两条或者：
%.o:%.c
	gcc -c -o $@ $<

	
//运行规则：
1.当目标文件不存在时，执行规则
2.当依赖文件比目标文件新时，执行规则
3.通配符的使用 $@(表示目标） $< (表示第一个依赖文件 $^(表示所有依赖文件）
4.假想目标.PHONY （例如：本项目中可能有文件于clean重名）   .PHONY: clean
5.变量：多种方式 := 即时变量 出现就被定义好了  = 延时变量 需要用的时候才初始化确定
6.make后不带目标则去生成第一个规则的第一个目标，否则：make clean
7.makefile自带函数 filter，wildcard，filter-out
8.工程的子目录下也可以有makefile，在主makefile中递归的调用它们
ex：make -C /a -f $(TOPDIR)/Makefile.build   (makefile.build是子目录的makefile）
~~~



## 文件类型与操作



文件（linux中一切且文件）

1.挂载各种sd卡，硬盘，flash，u盘。访问硬件上的各种真实普通文件 

2.linux内核提供的虚拟文件系统： /proc （内核信息）

3.特殊文件：/dev 设备文件 （设备节点，通过驱动程序去访问硬件）字符设备（tty）+块设备（sd*）



> （Linux下的设备通常分为三类，字符设备，块设备和网络设备。
> 设备驱动程序也分为对应的三类：字符设备驱动程序、块设备驱动程序和网络设备驱动程序。
> 常见的字符设备有鼠标、键盘、串口、控制台等。
> 常见的块设备有各种硬盘、flash磁盘、RAM磁盘等）



#### /dev 目录详解

描述硬件设备，用于内核（驱动）与应用层程序之间交互的通道。

设备对象（块设备，字符设备，网络设备）  2. 设备驱动程序（设备驱动程序负责将设备对象映射到设备节点，处理中断） 3. 设备节点



设备节点：/dev目录下都是设备节点。它用于连接用户程序和设备驱动程序，每个设备节点都对应一个特定的设备对象，用户程序可以通过设备节点来操作设备，从而控制设备的操作。使用标准的POSIX接口函数(read, write, ioctl等)可以操作大多数设备节点，而无需了解具体的设备类型和驱动程序。



>一个设备可以对应多个设备节点是因为在Linux系统中，每个设备节点都是一个文件，通过这个文件可以访问设备驱动程序提供的设备功能。对于同一个设备，不同的设备节点可以提供不同的访问权限和访问方式，例如读写、控制等等。
>
>例如，对于一个USB摄像头设备，它可以对应多个设备节点，例如/dev/video0、/dev/video1等等。这些设备节点可以提供不同的访问方式，例如使用视频采集软件来捕获视频流、使用控制台命令来控制摄像头的参数等等。
>
>另外，一个设备可以通过多种总线类型连接到计算机上，例如USB、PCI、串口等等。每种总线类型都有自己的设备驱动程序和设备节点，因此同一个设备在不同的总线类型下可以对应不同的设备节点。

#### 单片机mcu和mpu/soc的区别？

MCU 单片机：c语言使用芯片厂商提供的很多库函数，直接控制寄存器来操作硬件（点亮小灯） 《直接访问硬件》

（应用程序和驱动程序没有明显的界限）



MPU linux嵌入式：c程序中无法直接读写寄存器，通过驱动程序。（加上了操作系统层，权限级 ） 《不能直接访问硬件》

应用程序和驱动程序有明显的界限  ex：必须使用标准库glibc的open，read，write才能访问驱动程序 （触发中断 用户态-》内核态）



#### MPU应用层程序如何与驱动程序联动

1.应用层使用标准库函数ex open read write实现逻辑 （库函数内部触发中断swi software interrupt）

中断设置某寄存器value

2.陷入内核态，中断处理函数根据value，寻找到对应的驱动程序

驱动程序 = 支持标准库调用的框架 + 硬件操作



```linux
fd = open("/dev/led0", **)   //通过open找到对应的驱动程序，并返回相关信息在结构体fd中。/dev/led0 与 test.txt的区别，是根据文件类型来进行不同的操作（字符类型vs 普通文件）

1.确定为led为字符设备，对应一个字符驱动结构体数组
cat /proc/devices  查看已经支持的设备有哪些，
主设备号标识设备类型 来确定对应驱动程序
次设备号用于标识同一类型设备中的不同设备，每个设备的次设备号都是唯一的

开发驱动程序：

1.选择空闲的主设备号

2.创建包含众多file_operations的驱动函数的结构体(该类型的驱动程序应该执行什么逻辑 ex对于open的驱动实现）

3.注册：把结构体放到字符驱动程序中（数组或链表），通过主设备号调用

module_init(xxx)
//把xxx作为驱动的入口函数注册：把刚写的驱动结构体放到字符结构体数组中


具体流程可以查看：https://www.bilibili.com/video/BV1Yb4y1t7Uj?p=5&vd_source=8c2c0205fb00c73b1b17ce2d2925d1de
```

![1](./image/1.jpeg)

![1](./image/2.png)



![1](./image/3.png)

case 1:对于文件，都可以用标准的接口去访问他们 （open/read/write） 使用man 命令去查找具体的使用方法 

```c
///例如一个用glibc库实的复制读写的程序，read/write/open 使用 man 2 xxx 查看手册，看看参数怎么用

#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>

int main(int argc, char **argv){

    int fd_old, fd_new;
    char buffer[1024];
    int len;

    if(argc != 3){
        printf("Usage: %s <old-file> <new-file>\n", argv[0]); 
        return -1;
    }

    fd_old = open(argv[1], O_RDONLY);
    if(fd_old == -1){
        printf("you can't open file %s", argv[1]);
        return -1;
        
    }
  
    fd_new = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC , S_IRUSR |S_IWUSR| S_IRGRP |S_IWGRP|S_IROTH|S_IWOTH);
    if(fd_new == -1){
        printf("can't create the file %s\n", argv[2]);
        return -1;
    }
    
    while((len = read(fd_old, buffer, 1024)) > 0){
        if(write(fd_new, buffer, len) != len){
            printf("can't write %s\n",argv[2]);
            return -1;
        }
    }

    close(fd_old);
    close(fd_new);

    return 0;



}
```



case 2:framebuffer上数据影响LCD屏幕输出



LCD控制器定期去内存framebuffer中读取每个像素点对应的数据



bpp：bits per pixel（屏幕上一个像素点需要用几个字节来表达，RGB，透明度。。。）

r行c列： framebuffer[r* 列数 + c]*bpp/8 进行设置

mmap函数奖framebuffer映射到用户空间进行操作

```c
fd_fb = open("/dev/fb0", O_RDWR);
if(fd_fb <0){
  printf("can't open fb0\n");
}
static struct fb_var_screeninfo var;   //可变信息用一个定制的结构体来接收 ：bpp，分辨率。。。

//ioctl函数也是标准库函数
if(ioctl(fd_fb, FBIOGET_VSCREENINFO,&var)){
  
  printf("can't get var\n");
  return -1;}    //获取屏幕的可变信息

 line_width = var.xres * var.bit_per_pixel /8; 
 pixel_width = var.xres * var.yres * var.bit_per_pixel /8;
 fb_base = (unsigned char *)mmap(NULL, screen_size,....
  //获取到了buffer
                                 
                                 
  for(){}
                                 
  //for循环调用描点函数，绘制                               
                                 
  void lcd_put_pixel(int x,int y, unsigned int color){
    unsigned char *pen_8 = fb_base + y *line_width + x* pixel_width; //基地址 + 便宜的位置
    
  
                                 
 switch(var.bits_per_pixel){
   case 8:...
     
     case 16:
   {
     red = (color >> 16) &0xff;
     green = (color >>8) &0xff;
     blue = (color >>0) &oxff;
     color = ((red >>3) << 11) | ((green >>2)<< 5 ) (blue>>3);
     *pen = color;
   }
 }
                                   
```







## 内核模块

> 编进内核：假如驱动出现问题，只能debug整个内核重新编译
>
> 动态加载：只需要改ko，（内核轻量化，启动快）
>
> 
>
> 如何在内核中加入模块？
>
> 在对应的内核drive目录中将驱动c程序复制进去，修改makefile，各层的Kconfig
>
> 
>
> 简单来说就是去饭店点菜：Kconfig是菜单，Makefile是做法，.config就是你点的菜。
>
> Makefile：一个文本形式的文件，编译源文件的方法。
>
> Kconfig：一个文本形式的文件，内核的配置菜单。
>
> .config：编译内核所依据的配置。
>
> https://blog.csdn.net/thisway_diy/article/details/76981113
>
> 其中.config只有根目录下有，Kconfig和Makefile在根目录和每个子目录都有
>
> make menuconfig 根据各层的Kconfig，修改后，.config也会改变
>
> 因此配置内核，我们就可以得到下面结论了：
> 1、添加功能涉及到3类文件：.config，Kconfig，Makefile。在Kconfig中**描述定义**功能，在Makefile中功能编译方法，在.config中打开功能。
> 2、.config可以不修改，因为修改Kconfig后，make menuconfig中就有对应条目了，在图形界面中修改对应条目实际上就是修改.config。
> 3、如果新的功能都添加完了，那么.config控制着每个功能的开关，因此是很重要的。make clean会清除它，因此幸幸苦苦make menuconfig裁剪完功能后，推荐它备份一下。
>
> 4.make %_defconfig命令会将arch/arm/configs/%_defconfig 文件复制为根目录下的.config 文件。因此作用和`make menuconfig`相同
>
> 
>
> 在Linux内核的根目录下，运行`make`命令会生成两个文件，一个是`Image`，另一个是`zImage`。其中，`Image`为普通的内核映像文件，而`zImage`为压缩过的内核映像文件。一般情况下，编译出来的`Image`大约为4M，而`zImage`不到2M 。
>
> 而运行`make uImage`命令会生成一个名为`uImage`的文件。它是uboot专用的映像文件，它是在zImage之前加上一个长度为64字节的“头”，说明这个内核的版本、加载位置、生成时间、大小等信息；其0x40之后与zImage没区别。换句话说，如果直接从uImage的0x40位置开始执行，那么zImage和uImage没有任何区别 





> 内核驱动模块的开发遵循一定的框架和原则
>
> 1.入口函数，出口函数。内核模块加载器会执行固定操作
>
> 2.内存管理：kmalloc(),kfree()
>
> 3.一些函数发生变化：应用层printf(), 内核 printk  -》存放在内核的一个buffer中



~~~linux
/hello目录下

hello.c 最简单的驱动程序

#include<linux/module.h>

static int __init hello_init(void){     ///__init 是宏	只用一次，运行后可在内存中释放
return 0;
}

static void __exit hello_exit(void){
}

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");


Makefile:

obj-m += hello.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
	
clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

//
在makefile中还有make，因为是需要调用指定内核（或者不同的工具链）的makefile，使用一些配置选项
obj-m :告诉需要编译成模块的源文件是hello.o
~~~



#### 应用层+驱动层开发逻辑

调用逻辑：glibc库函数操作某类型的硬件-》硬件类型在内核中有对应的内核驱动 （主设备号  -〉实现功能

驱动也需要有分层思想

 

1.确定空闲的主设备号（cat /proc/devices） 

> cat /proc/devices  //查看已经被使用的主设备号

2.驱动程序：入口函数在相应类型的驱动数组/链表中注册（如字符设备在字符类型驱动数组中注册）

>驱动程序的标识符：主设备号
>
>register_chrdev(主设备号，驱动name，驱动实现结构体/句柄)
>
>register中申请设备号输入为0时，表示让内核自动分配，非负整数则自己指定
>
>不使用时，也记得卸载

3.编写子系统结构体（函数指针）的实现

> Ex file_operations
>
>  Static struct file_operations hello{
>
> .owner = THIS_MOUDLE,
>
> .read = hello_read,
>
> .write = hello_write,
>
> };
>
> 应用层 open，write，read，当驱动中file_operation没有实现对应的驱动函数逻辑的时候，可以open，但无法read write
>
> 内核中copy_to_user()可以把驱动中的值返回给应用层
>
> 相反：copy_from_user();  

出口函数卸载

4. 在驱动中生成设备驱动对应class和基于class的设备

![1](./image/5.jpg)

对应在/sys/class中出现hello_class

用hello_class作为软件模版，生成设备  

在/dev中根据class信息生成myhello设备结点



copy_from_user()

copy_to_user()  

5.编写应用层程序

>open("/dev/xtz"....)
>
>Write()
>
>poll()
>
>应用层 ./xxx -r/-w  一个c的可执行文件，加上参数，可以加入path形成命令行工具
>
>如果不是真实的设备结点，可以创建一个
>
>mknod [name] [type] [主设备号] [次设备号]
>
>rm 





#### 驱动程序和硬件交互

操作硬件

1.查看硬件原理图确定连接 （确定引脚）

2.查看芯片手册 （确定如何控制引脚）

3.写程序

ex：gpio，使能gpio模块，设置模式（设置寄存器），方向寄存器，写入读取数据（数据寄存器）

>多程序：同时运行一个程序两次，输入不同的值，但程序输出是相同的值，矛盾
>
>单片机同一个程序运行多次，需要手动指定内存地址，运行在不同的物理地址空间上
>
>Os虚拟地址的：os加载程序到内存，内核帮我完成映射，得到虚拟地址 （两个进程运行，运行在**虚拟地址**上）
>
>

> 硬件相关：
>
> 芯片手册列出了不同硬件的物理地址
>
> 单片机：内存控制器 memory controller连接不同部分硬件   
>
>  soc：mmu memory management unit （虚实转换）
>
> 操作系统是操作虚拟地址，使用ioremap(物理地址，长度)完成映射 「注意是在内核中驱动程序使用」
>
> ## 在内核态用ioremap将硬件地址映射到内核虚拟地址空间，以便对硬件完成访问 （可以理解向mmu注册） 后面引入设备树 

![1](./image/4.png)





ex：通过gpio点亮led灯的驱动程序

![1](./image/6.jpg)



 

#### 分离原则/总线设备驱动模型   

**（之前的方法，一次就写死，不具有重复使用的属性）**



1. 驱动程序类型复杂，同类型下可能有不同设备

相同类型都在独立代码内实现（同一个代码中包含驱动和硬件信息），代码重复且冗余



2. 把驱动程序拆为**通用的框架（ex：gpio 可以操作led风扇等等）和具体的硬件操作**  

（直接更改硬件.c程序，不用阅读复杂的驱动框架程序，驱动框架程序去调用硬件程序）



3. 引入统一类型结构体（分为通用的drive.c, 硬件相关的device.c  g）

例如：bus/platform类型

总线类型很多：USB_bus, I2C_bus, spi_bus

注册后位于/sys/bus下



4. 在结构体中注册，加入链表：drive，device分别注册

 

5.  若drive和device匹配，则运行probe函数，执行file_operation,注册，实现内核态open那套 





(以前的file_operating 现在在probe函数中实现，且参数由硬件驱动来传入)

![Screenshot 2023-05-22 at 1.21.02 PM](./image/Screenshot 2023-05-22 at 1.21.02 PM.png)





## 设备树

>**描述硬件信息的数据结构并提供驱动程序使用**
>
>设备树中有三个重要的概念：DTS、DTC 和 DTB。DTS（Device Tree Source）是设备树的源文件，它是一种 ASCII 文本格式的设备树描述文件。DTC 是将 DTS 文件编译成 DTB 的工具。DTB（Device Tree Blob）是 DTS 被 DTC 编译后的二进制格式的设备树文件，它可以被 Linux 内核解析
>
>软硬件解耦：传统的裸机开发中，硬件设备的信息是直接编码到程序中的，这样会导致代码的可移植性差，难以适应不同的硬件平台。
>
>[在 Linux 2.6 中，ARM 架构的板极硬件细节过多地被硬编码在 arch/arm/plat-xxx 和 arch/arm/mach-xxx 中，采用设备树后，许多硬件的细节可以直接通过它传递给 Linux，而不再需要在内核中进行大量的冗余编码](https://zhuanlan.zhihu.com/p/425420889)[1](https://zhuanlan.zhihu.com/p/425420889)。

>硬件物理连接/配置  -》dts
>
>
>
>x86-64：ACPI   通过BIOS或UEFI提供硬件描述信息，包括系统配置、设备类型、资源分配、设备状态等。它还提供了一种标准的接口，使得操作系统可以动态地管理设备，包括设备的开启、关闭、休眠、唤醒等。
>
>设备树（Device Tree）是一种用于描述硬件的数据结构，最初由PowerPC架构引入，现在被广泛应用于嵌入式系统和ARM架构中

太多硬件相关的代码存放在c文件中，例如驱动程序硬件部分过于庞大 .ko ---》**使用配置文件 设备树**

（不同板子的硬件配置不同，驱动框架可以从设备树dts 中获取相应的硬件信息）



内核解析配置文件 ： dtb（编译好的设备树文件）dts（device tree source） 在内核之外 

dts -〉dtb -〉（内核解析）device_node -> platform_device(含有compatible属性的  )



如何找到某一个硬件的设备树信息：

从电路原理图入手，在/boot/dtb目录下寻找pinctl信息，最后找到结点信息



硬件分类：

1.GPIO引脚  （可编程控制的物理接口） 

连接led，红外模块（指定硬脚，遵循的信号规范）

pinctl：引脚配置控制

compatible：运行的驱动

![1](./image/drive.png) 

2.协议 （串口UART，spi，i2c） （通信协议，硬件实现）

![1](./image/i2c.png) 

那些设备在i2c总线上：创建子节点

3.内存接口（网卡，内存）

4.模拟电路

  



**驱动程序中硬件部分的信息可以在代码中指定在设备树文件中寻找并匹配**

![1](./image/7.png) 





## 中断

设置中断控制器：GIC（generic interfere controller） 集成在主板上

注册中断处理函数

中断也需要驱动程序

![Screenshot 2023-05-31 at 3.29.42 PM](/Users/mac/Desktop/CS/SOC_learning/8.png)



发生中断：

1.保存上下文，保存现场

2.调用中断处理函数

3.恢复现场



硬件-》写一个设备树硬件节点（通过compatible与驱动程序匹配） 应用层操作对象-〉驱动 -〉设备树





### 补充：

进程同步（运行有依赖），进程异步（临界区资源互斥访问） -》进程通信，进程调度



> 中断处理函数，异常处理函数，进程异步通信：信号处理函数
>
> 
>
> 应用层-》驱动程序-〉硬件
>
> 进程异步通知-》信号-〉信号处理函数  （堆区的处理）
>
> 
>
> 信号量机制：解决进程互斥和进程同步问题        用来表示某一种资源的数量
>
> 使用操作系统提供的一对原语（一气呵成，不可中断）来实现
>
> PV操作：wait申请，signal释放
>
> 整形信号量：表示系统中资源的数量 
>
> 记录型信号量：有一个等待队列（效率更高）
>
> 
>
> 当中断，或者信号到达时候，进程需要有机制进行应答



> 在操作系统中，当某些重要的事件发生时（比如进程收到一个信号，或者内核发生某些异常情况），它会向相关进程发送一个信号。相关进程可以通过注册一个信号处理函数来响应这个信号，对信号进行处理。信号处理函数可以用于实现进程通信、异常处理、事件处理等功能





 ## 网络通信

简单网络编程：TCP（可靠有连接） UDP（无连接不可靠）应用于不同的场景

> Keyword：客户端，通信协议，服务端 ：IP，端口 



> 类似于 open read write ， 网络编程方式： socket  bind listen accept send connect  创建绑定套接字，监听发送接收  



socket是一个编程接口，不是一个网络协议，因此它并不位于TCP/IP五层模型中的任何一层。**socket API是对网络协议栈的封装，使得应用程序可以方便地使用TCP/IP协议进行通信**。在TCP/IP五层模型中，socket所使用的协议，如TCP和UDP，位于传输层。传输层是负责提供端到端的可靠数据传输服务的层次，它位于网络层和应用层之间。



> ex客户端：
>
> socket创建客户端通信的套接字文件：指定通信的协议族和数据类型
> 使用connect主动向服务器发起连接请求，与服务器的accept实现三次握手建立连接。连接成功之后客户端可以通过socket返回的文件描述符进行通信 ，服务器使用accept返回的通信文件描述符进行通信
> 使用send/recv和服务器 发送接收数据进行通信
> 通信结束之后断开连接close或者shutdown





## 多线程编程 

进程间通信（两个main函数）：效率低

线程间通信（一个main函数）：效率高 

pthread_create() : 创建线程

信号量触发，否则死循环太浪费cpu资源

互斥量：防止同时访问









--------



串口：UART一种常见硬件接口  （通用，串行，异步，低速）

>异步通信：发送和接收不需要共享一个时间，而是在数据帧中添加同步信息，来判断数据的开始和结束 

  用途：打印信息，连接外接模块



双方约定波特率，不同的逻辑电平（判断01的标准：RS-232（电压高，长距离），TTL/CMOS（电压低，短距离））



> tty设备：  UART类型（键盘鼠标），console，terminal 
>
> ex：向键盘输入lsa，发现输入错误，删除a，再回车执行的过程  





