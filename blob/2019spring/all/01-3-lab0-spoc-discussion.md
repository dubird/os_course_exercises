# lec2：lab0 SPOC思考题


## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

*硬件需要支持中断和IO管理，地址映射机制和稳定的存储；需要提供中断，设置内存寻址，设置页表，执行I/O操作等特权指令。

- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意那些方面？

*保护模式和实模式的区别在于内存是否被保护。实模式访问的地址都是真实的物理地址，因而可能发生越界访问，而保护模式需要从虚拟地址再到物理地址，操作系统实现了对内存的管理和保护。实模式切换到保护模式前需要建立合适的全局描述符表，并使GDTR指向GDT，进入保护模式前先寄存ss及sp的值，便于返回后恢复堆栈。

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？
*物理地址是访问计算机中的内存和设备的最终地址。
*逻辑地址是经过地址变换后，访问指令时给出的地址。
*线性地址是通过段机制形成的地址空间。
*线性地址是逻辑地址到物理地址的中间层，程序访问指令先转换为线性地址再转化为物理地址从而访问到真实数据。

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？
*risc-v有三种特权模式:机器模式（M)、监管者模式（S)、用户模式（U),由M-S-U，权限依次递减，M模式可以拦截和处理异常，可以直接访问所有的物理地址。S模式可以处理中断，且提供了基于页面的虚拟内存，大多地址都是虚拟地址，需要通过页表转化为物理地址访问。U模式权限最低，不可使用M模式指令和访问CSR，且可访问的内存被限制在PMP范围内。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```
":"后面的数字表示每个域所占的位数


- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

`0x20003`


### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

.code16                                             # Assemble for 16-bit mode
    cli                                             # Disable interrupts
    cld                                             # String operations increment

    # Set up the important data segment registers (DS, ES, SS).
    xorw %ax, %ax                                   # Segment number zero
    movw %ax, %ds                                   # -> Data Segment
    movw %ax, %es                                   # -> Extra Segment
    movw %ax, %ss                                   # -> Stack Segment

*首先cli禁用硬件中断，cld使方向标志符DF置为0.然后初始化DS，ES，SS，使其均置为0。



#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

 * 进行复杂数据结构中的数据访问 defs.h:offsetof
 * 数据类型转换  defs.h:to_struct
 * 常用功能优化模块化 defs.h: ROUNDDOWN


## 问答题

#### 在配置实验环境时，你遇到了那些问题，是如何解决的。
*使用实验楼环境。

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
