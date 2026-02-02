# qemu说明
qemu是一个模拟器，模拟器计算机。本文分析qemu的纯软模拟，不分析虚拟化行为。本文按运行的裸机程序的从简单到复杂，逐步分析使用的特性。
# qemu功能介绍
qemu 模拟裸板行为。包括cpu,drm，spi,xpi，flash等外设。
cpu功能包括模拟寄存器，执行取指，译码，执行。页表转换,中断处理器，外设和时钟中断。

## 使用qemu运行裸机程序流程分析
### qemu运行指令流程解析
实际的机器一般包括什么，qemu这里运行命令，需要模拟什么.
以运行一些赋值和运算类的指令为例子，要模运行这个elf，我们需要cpu，内存完成取指令，译码，执行。cpu的pc寄存器复位后指向特定地址，不断从内存中取出指令，译码为对应的操作后执行。
#### cpu复位
#### 指令加载流程
#### 
### qemu指令trace打印与格式修改
### qemu调用uart流程分析
### qemu触发中断流程分析

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

### machine
machine是soc的模拟，结构体如下：
```c
struct MachineState {
    /*< private >*/
    Object parent_obj;
    Notifier sysbus_notifier;

    /*< public >*/

    void *fdt;
    char *dtb;
    char *dumpdtb;
    int phandle_start;
    char *dt_compatible;
    bool dump_guest_core;
    bool mem_merge;
    bool usb;
    bool usb_disabled;
    char *firmware;
    bool iommu;
    ...
    /*
     * convenience alias to ram_memdev_id backend memory region
     * or to numa container memory region
     */
    MemoryRegion *ram;
    DeviceMemoryState *device_memory;

    ram_addr_t ram_size;
    ram_addr_t maxram_size;
    uint64_t   ram_slots;
    ...
    char *kernel_filename;
    char *kernel_cmdline;
    char *initrd_filename;
    const char *cpu_type;
    ...
    CpuTopology smp;
    ...
};
```
自定义的machine维护自己的结构体，包含cpu，flash等外设，构成一个完整的soc模拟。下面以virt这个riscv架构默认的machine为例,分析如何添加machine
一个machine包括cpu簇和flash,外设，中断处理器等内容。使用一个结构体表示machie的状态。
```c
riscv/virt.h
struct RISCVVirtState {
    /*< private >*/
    MachineState parent;

    /*< public >*/
    RISCVHartArrayState soc[VIRT_SOCKETS_MAX];
    DeviceState *plic[VIRT_SOCKETS_MAX];
    PFlashCFI01 *flash[2];
    FWCfgState *fw_cfg;

    int fdt_size;
};
```

machine在实现上是一个qom，需要实现class_init和instance_init。父类是machine.
```c
static const TypeInfo virt_machine_typeinfo = {
    .name       = MACHINE_TYPE_NAME("virt"),
    .parent     = TYPE_MACHINE,
    .class_init = virt_machine_class_init,
    .instance_init = virt_machine_instance_init,
    .instance_size = sizeof(RISCVVirtState),
};

static void virt_machine_init_register_types(void)
{
    type_register_static(&virt_machine_typeinfo);
}

type_init(virt_machine_init_register_types)

```
class_init函数里的包括MachineClass的init函数，设置cpu类型，最大cpu数量
MachineClass的init函数负责以下内容：
- 检测pcu个数是否超出上限
- 调用外设的init函数，包括ram/rom和clint/plic函数
- 设置property
- 创建设备树，（sdk中需要自己写设备树，这里是方便内部os使用？）
#### 检测cpu数量
socket有插槽的意思，调用socket相关函数检测cpu能否找到，是否超出上限
#### machine在地址上挂载设备

### cpu
### 内存
### 外设

