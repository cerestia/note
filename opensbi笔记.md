# sbi概述
sbi(Supervisor Binary Interface)是一个标准，规定了给s模式提供的环境(see)需要实现什么功能,有最小集和扩展集，其中每个扩展和函数都有自己的id,分别为eid和fid。linux通过ecall调用sbi提供的功能。

# opensbi 概述
opensbi是一个开源sbi实现。通过解析设备树，完成对串口，中断管理等硬件的配置，支撑sbi需要的服务。
前后状态变化：opensbi运行后，每个hart有自己的栈，mtvec指向opensbi的异常处理函数，设置了scratch结构体，配置外设驱动，从m态跳转到设置的状态（m或者s）,跳转到下一级的首地址，下一级可能是u-boot或者os
副作用：设置了串口寄存器，打印opensbi的信息和设备树中的部分内容

# opensbi到三种编译方式
payload:u-boot与opensbi编译到一个文件
jump:opensbi跳转到固定位置执行u-boot代码
dynamic:通过参数跳转

# opensbi编译
## 官方代码编译
 ``` shell
 make PLATFORM=generic CROSS_COMPILE=riscv64-unknown-linux-gnu- PLATFORM_RISCV_XLEN=64
```
如果想要带上调试信息，加上DEBUG=1
由于opensbi依赖的库不多，可以使用riscv64-unknown-elf-gcc编译，macos下如果出现bool定义相关问题，尝试在CFLAGS后加上--std=gun99。建议使用orb虚拟机在x86环境编译

# 在qemu上对opensbi分析
## jump模式流程内容分析：
选择一个hart，如果链接地址和加载地址不一样，搬运过去。bss段清0。
如果规定了设备树地址，重新加载设备树地址。
运行fw_platform_init解析设备树，获取hart的编号，获取栈大小
为scratch_init做准备，构造scratch结构体,Scratch Space 存储了如下一阶段的入口地址、运行模式、设备树地址等关键信息,剩余部分被拿来做动态内存区域
设置stack
非启动核运行_start_warm，重制寄存器，设置scratch space和stack
进入sbi_init函数运行
sbi最终提供文档中的函数功能

# 在milk-v duo上运行opensbi
## riscv 开发板启动流程介绍
开发板运行On-Chip ROM,这一步骤出场后就定死了，一般使用prom，只能写一次
运行sram中的数据，初始化ddr，加载opensbi和uboot到内存
opensbi运行，然后跳转到u-boot
u-boot完成操作系统引导，跳转到linux内核

# 分析milk-v duo opensbi编译流程

## 移植mlik-v duo opensbi流程
在platform目录下创建自己的板子文件夹，编写platform.c。编写 config.mk

# 在milk-v duo上搭建调试流程
我使用ubuntu系统调试，jtag调试器使用ft232h，riscv-openocd从编译
todo 配置文件

# 在milk-v duo上运行
ubuntu下使用picocom，bl2阶段有乱码，opensbi可以打印。minicom在opensbi阶段也有乱码。


