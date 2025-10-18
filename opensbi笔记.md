# opensbi 概述
sbi(Supervisor Binary Interface)是运行在m态的程序，启动加载流程，为操作系统提供m态处理程序

# riscv 启动流程介绍
开发板运行On-Chip ROM,这一步骤出场后就定死了，一般使用prom，只能写一次
运行sram中的数据，初始化ddr，加载opensbi和uboot到内存
opensbi运行，然后跳转到u-boot
u-boot完成操作系统引导，跳转到linux内核

# opensbi到三种编译方式
payload:u-boot与opensbi编译到一个文件
jump:opensbi跳转到固定位置执行u-boot代码
dynamic:通过参数跳转

# macos下编译opensbi方法
由于opensbi依赖的库不多，可以使用riscv64-unknown-elf-gcc编译，macos下如果出现bool定义相关问题，尝试在CFLAGS后加上--std=gun99
