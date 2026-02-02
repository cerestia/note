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
MachineClass的init函数负责检测pcu个数是否超出上限，调用外设的init函数，包括ram/rom和clint/plic函数。
例如：
```c

```
### cpu
### 内存
### 外设

## 指令执行流程分析
