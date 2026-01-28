# qemu说明
qemu是一个模拟器，模拟器计算机。本文分析qemu的纯软模拟，不分析虚拟化行为。
# qemu功能介绍
qemu 模拟裸板行为。包括cpu,drm，spi,xpi，flash等外设。
cpu功能包括模拟寄存器，执行取指，译码，执行。页表转换,中断处理器，外设和时钟中断。

## 各个硬件模拟
### qom说明
qemu有自己的一套接口模拟包括cpu在内的硬件模型，所有硬件的创建都通过这套接口。
参考文章：
https://gevico.github.io/learning-qemu-docs/ch1/sec3/qemu-qom/
https://martins3.github.io/qemu/qom.html#basic
qom实现一套类机制来方便设备的描述。包括：继承，静态成员。设备允许设定自定义属性,执行的时候设置属性的值。
class相关的变量和函数是类的静态变量和函数。
注册流程大概分为三个阶段：
type_init : 注册一个 TypeInfo
TypeInfo::class_init : 初始化静态成员
TypeInfo::instance_init : 初始化非静态成员
在 qdev 中还有 qdev_realize 来进行和 device 相关的初始化
* type_init:将注册的类型加入到链表init_type_list[MODULE_INIT_QOM]
* 调用class_init函数，进行静态成员变量初始化
* 调用instance_init函数，动态对象的初始化


### cpu
### 内存
### 外设

## 指令执行流程分析
