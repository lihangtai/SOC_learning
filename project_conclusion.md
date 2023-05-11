# project 1

>设备：linux主机，飞凌MX8 SOC
>
>实现：使用sdk中的交叉编译工具链，在linux主机上编译简单驱动程序为.ko模块，linux主机通过nfs在局域网中共享目录/home/jerry/Desktop/share，在MX8中挂载文件系统后，运行并成功挂载模块



## 过程：

### 1.配置MX8

>根据soc手册

sdk：手册，工具，编译链，源码（内核，文件系统），烧写软件

----------

使用脚本在linux上安装交叉编译环境

每次使用交叉编译环境时，需要运行环境 （ARCH，PATH ....）

编译源码生成镜像，使用SDK中工具烧录到emmc/sd卡中

通过MX8上开关选择启动方式 （reset 按钮）

>soc Linux运行平稳

----------------



网络连接：

Linux先用串口连接上mx8（注意设备节点/dev/ttyUSB的权限设置）

使用tty配置mx8的网口为静态ip

在linux上使用/etc/netplan 配置网口

ping通，通过ssh连接到mx8上

linux开启nfs-kerner-server 

mx8挂载  mount ip:/home/jerry/Desktop/share /mnt



### 2.驱动开发逻辑

> 应用层，驱动层程序都为.c程序
>
> 详情看README



**逻辑**：

应用层：统一使用glibc的标准接口函数：open，write，read （获得句柄）

标准接口函数：设置寄存器（识别驱动程序），触发中断进入内核层

内核根据：文件类型，主设备号，次设备号，识别对应的驱动程序

驱动程序根据子系统框架规范（ex：字符设备对应的file_operation），分别实现对饮的需要的函数

内核驱动子函数实现对硬件的操作



**句柄**：

~~~linux
// 假设我们要在 C 代码中创建一个文本文件，我们可以使用标准输入输出库中的文件句柄来实现。具体步骤如下：
打开文件：我们需要使用 fopen() 函数打开指定的文本文件。该函数会返回一个指向类型为 FILE 的结构体的指针。

FILE *fp;
fp = fopen("test.txt", "w+");
if(fp == NULL) {
    printf("Failed to open the file.\n");
    return -1;
}

写入数据：现在，我们已经成功打开了文件，下一步是写入数据。我们可以使用 fprintf() 函数来将文本写入该文件。

fprintf(fp, "This is a sample text.");

关闭文件：最后，我们需要使用 fclose() 函数来关闭该文件并释放资源。

fclose(fp);

在上述例子中，fp 就是一个文件句柄，它是将数据写入文本文件的关键所在。使用句柄的好处是，它可以让我们直接操作文件，而无需了解其内部实现细节。


FILE 结构体定义了一些变量，如文件指针、缓冲区指针、缓冲区长度、当前缓冲区位置、是否有错误等，用于标识和访问文件。

以下是 FILE 结构体的定义：

struct _IO_FILE {
    int _flags;          // 文件状态标识
    char* _IO_read_ptr;  // 读取指针位置
    char* _IO_read_end;  // 读取缓冲区末尾
    char* _IO_read_base; // 读取缓冲区地址
    char* _IO_write_ptr; // 写入指针位置
    char* _IO_write_base;// 写入缓冲区地址
    char* _IO_write_end; // 写入缓冲区末尾
    char* _IO_buf_base;  // 缓冲区基地址
    char* _IO_buf_end;   // 缓存区末尾地址
    char* _IO_save_base; // 保存原始缓冲区基地址
    char* _IO_backup_base; // 缓存区起始位置的副本
    char* _IO_save_end;  // 保存原始缓冲区末尾地址
    struct _IO_FILE *_chain; // 链表地址
    int _fileno;        // 文件号码（系统唯一标识文件的整数值）
    int _flags2;        // 文件状态标识2
    __off_t _old_offset; // 此处不赘述，用于兼容一些旧 API 的值
    __off64_t _offset;  // 文件指针位置
    unsigned short _cur_column; // 当前列数
    signed char _vtable_offset; // vtable 偏移量
    char _shortbuf[1]; // 留给小缓冲区使用
    void *_lock;       // 文件锁对象
    _IO_lock_t *_lockfile; // 锁文件对象
    __off64_t _bytes_sent; // 此处不赘述
};
~~~



### 3.运行模块

>使用编译工具链编译模块
>
>make
>
>lsmod
>
>insmod
>
>rmmod
>
>dmesg



# Project 2

>设备：linux主机
>
>实现：创建字符类型设备节点，并为其开发驱动函数，了解了字符驱动程序的大致开发框架
>
>上一环节驱动程序的进阶版

## 逻辑

应用层使用glibc库函数操作设备 （触发中断进入内核态）

根据文件类型（字符/块）并根据主设备号寻找到合适的类型驱动程序

驱动执行对应应用层的驱动函数：write()  -> kernel_write()



## 过程

驱动程序申请主设备号：

>目前主机已经使用的主设备号 
>
>cat /proc/devices
>
>如果register注册时，申请为0，则是请内核主动分配

编译，挂载，卸载模块

>lsmod
>
>insmod
>
>rmmod
>
>Make >makefile中含有make，是调用内核中的makefile

设备节点与驱动对应

>sudo mknod name type 主设备号 次设备号