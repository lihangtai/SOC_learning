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

gcc，make，交叉编译gcc，gdb，cmake，objdump，fdisk，dmesg，mkfs, systemctl，

------------



### 目标

1. 交叉编译的配置和使用
2. 内核的配置与编译（make menuconfig）
3. 驱动基础：基础的驱动，比如GPIO、UART、SPI、I2C、LCD、MMC，高级的驱动，比如USB、PCIE、HDMI、MIPI、GPU、WIFI、蓝牙、摄像头、声卡。（总线，串口）

  4.glibc库的了解使用（使用man 3查询手册）

5. 多线程
5. 板级通信，板间通信





## configuration

### 板子与linux主机通信方式

1:网线连接：使用ifconfig分别为linux有线网口，板子有线网口设置同一网段的ip地址

ip address：192.168.1.100



方法1：soc + linux ：UART串口连接，使用静态路由：把网口的IP固定

修改配置文件

sudo systemctl restart networking



方法2:soc+路由器+linux：可以使用有线或者无线连接，路由器开启DHCP



linux端使用ssh则可以直接访问



方法2.串口通信：通过UART链接，使用putty连接

~~~linux
1.
ls -l /dev/tty*     
// tty设备：前生打字机。现在允许用户输入，在计算机上查看输出，通常与串口相连接的设备。实体设备（USB tty），软件设备（终端，终端模拟器
Note：串口：UART，RS-232   /dev是一个虚拟文件系统，用于访问设备和设备文件，包含了所有可用的文件（没有对应驱动可能没有显示）
2.
chmod 666 /dev/ttyUSB0     //暂时打开权限

使用serial连接
~~~





### 开启NFS

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







### 使用SDK配置，启动SOC

一个嵌入式SDK包含：交叉编译工具链，操作手册，源代码（库和头文件），调试烧录工具/脚本，内核

1.安装编译环境(shell)

启动编译环境，才能使用aarch64-pock-linux-gcc

>重启linux或者打开新的shell都需要重新执行启动环境。（同一个shell属于一个进程组）

2.解压，按照makefile编译生成镜像/内核/其他程序





3.烧写到sd卡中，soc使用sd卡启动板子

~~~linux
lsblk  //查看挂载情况，找到sdx
blkid  //检查sd卡文件类型
fdisk dev/sdx //进行分区
mkfs ext4 /dev/sdx   //格式化
mount /dev/sdx /xxpath  //挂载


~~~

case one：全新的sd卡

case two：已经有内容

先格式化再分区





**编译内核**

1.配置编译内核 （顺便dtb，driver module ---》 dtb /lib）

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

2，移植到板子上 

~~~linux
cp /mnt/ZImage /boot  //zImage是编译出来的内核
cp /mnt/xx.dtb /boot  //更新设备树
cp /mnt/lib/modules /lib -rfd //更新模块   （覆盖目录）
sync 
reboot

//驱动程序
更改makefile的内核路径
然后： insmod xxxx.ko

~~~



3.编译测试驱动程序





question

1.编译后生成 内核img， dtb，lib

而/boot下的文档有efi，grub，各种版本内核。

如何实现内核切换？vmlinuz是什么？

case 1:把dtb和img换到/boot下

case：个人电脑什么逻辑呢？





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

![ ](/Users/mac/Desktop/CS/SOC_learning/Screenshot 2023-04-27 at 1.05.04 PM.png)







## 文件

文件（linux中一切且文件，普通文件读写磁盘（也调用驱动），字符设备调用驱动

1.挂载各种sd卡，硬盘，flash，u盘。访问硬件上的各种真实普通文件

2.linux内核提供的虚拟文件系统： /proc （内核信息）

3.特殊文件：/dev 设备文件 （通过驱动程序去访问硬件）

> 字符类驱动程序（char）：寻找合适驱动，块设备驱动程序（block）：文件系统-》块设备-〉块设备驱动）
>
> 主设备号：确定对应哪个驱动
>
> 次设备号：这个驱动中的哪个硬件



对于文件，都可以用标准的接口去访问他们 （open/read/write） 使用man 命令去查找具体的使用方法 

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



### case 2:framebuffer上数据影响LCD屏幕输出



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

