LINUX系统调用
=============

1. `系统调用简介`_
2. `INT80中断调用`_
3. `中断向量表`_
4. `32位快速调用`_
5. `64位快速调用`_
6. `syscall库函数`_
7. `虚拟系统调用`_
8. `glibc包装函数`_
9. `系统调用约定`_
10. `系统调用列表`_
11. `32位系统调用编号`_
12. `64位系统调用编号`_

参考链接：

* https://blog.packagecloud.io/author/joe-damato/
* https://github.com/torvalds/linux

系统调用简介
------------

什么是系统调用？当你运行一个程序，调用了 open、fork、read、write（以及许多其他函数）
时，你实际上在发起系统调用。系统调用是程序进入内核以执行某些任务的方式。程序使用系统调用
来执行各种操作，例如创建进程、进行网络和文件操作、等等。你可以通过查看 syscalls(2) 的
手册页来获取系统调用的列表。

用户程序发起系统调用有几种不同的方式，发起系统调用的底层指令在不同的 CPU 架构之间有所不
同。作为一名应用程序开发人员，你通常不需要考虑系统调用是如何发起的。你只需要包含适当的头
文件，并像调用普通函数一样发起调用即可。glibc 提供了包装代码，将你从底层代码中抽象出
来，这些底层代码会安排你传递的参数，并进入内核。在深入探讨系统调用是如何发起的细节之前，
我们需要定义一些术语和核心概念。

硬件和软件，这里讨论的内容基于以下假设：你使用的是 32 位或 64 位的 Intel 或 AMD CPU。
这里的讨论可能对使用其他系统的人也有帮助，但下面的代码示例包含特定于 CPU 的代码。使用的
Linux 内核版本是 3.13.0，其他内核版本会类似，但确切的行号、代码组织和文件路径会有所不
同。使用 glibc 或基于 glibc 的实现的 libc 如 eglibc。另外 x86-64 指的是基于 x86 架构
的 64 位 Intel 和 AMD CPU。

用户程序、内核和 CPU 特权级别，用户程序（比如你的编辑器、终端、SSH 守护进程等）需要与
Linux 内核交互，以便内核代表用户程序执行一组用户程序自身无法执行的操作。例如，如果用户
程序需要进行某种 I/O 操作（open、read、write 等）或修改其地址空间（mmap、sbrk 等），
它必须触发内核运行以代表其完成这些操作。

是什么阻止用户程序自行执行这些操作呢？x86-64 CPU 有一个名为特权级别的概念。特权级别是
一个复杂的话题，适合单独讨论。但可以简要介绍一下特权级别的概念。特权级别是一种访问控制手
段。当前特权级别决定了可以执行哪些 CPU 指令和 I/O 操作。内核在最高特权级别运行，称为
Ring 0。用户程序在较低的级别运行，通常是 Ring 3。为了执行某些特权操作，用户程序必须变
更特权级别（从 Ring 3 变为 Ring 0），以便内核执行。有几种方法可以导致特权级别变更并触
发内核执行某些操作。

INT80中断调用
--------------

::

    # 汇编代码
    .equ SYS_EXIT, 1
    .equ EXIT_CODE, 0

    movl $SYS_EXIT, %eax
    movl $EXIT_CODE, %ebx
    int $0x80 # 中断会自动记录用户空间的返回地址

    # INT 0x80调用约定
    %eax system call number
    %ebx arg1
    %ecx arg2
    %edx arg3
    %esi arg4
    %edi arg5
    %ebp arg6 # note: not saved in the stack frame, should not be touched

我们先从一个常见的触发内核执行的方式开始：中断。你可以将中断视为由硬件或软件生成（或
“触发”）的事件。硬件中断是由硬件设备触发的，以通知内核某个特定事件已经发生。这种中断的
一个常见例子是当网络接口卡收到数据包时触发的中断。软件中断是通过执行一段代码触发的。在
x86-64 系统上，可以通过执行 int 指令来触发软件中断。中断通常都有分配给它的编号，其中一
些中断编号有特殊含义。你可以想象 CPU 内存中有一个数组。该数组中的每个条目都映射到一个中
断编号。每个条目包含 CPU 在收到该中断时开始执行的函数地址，以及一些选项，例如中断处理函
数应该在什么特权级别下执行。据此 CPU 才知道在收到某种类型的事件时应该执行哪个地址的代
码，以及该事件的处理程序应该在什么特权级别下执行。

.. _8259 可编程中断控制器: https://wiki.osdev.org/8259_PIC
.. _I/O 高级中断控制器: https://wiki.osdev.org/IOAPIC
.. _高级中断控制器: https://wiki.osdev.org/APIC

实际上，x86-64 系统上处理中断的方式有很多种。如果你有兴趣了解更多，请阅读有关
`8259 可编程中断控制器`_、 `高级中断控制器`_ 和 `I/O 高级中断控制器`_ 的内容。处理硬
件和软件中断还涉及其他复杂性，例如中断编号冲突和重新映射，我们先不关心这些细节。

型号特定寄存器（MSRs）是具有特定用途的控制寄存器，用于控制 CPU 的某些功能。CPU 文档中
列出了每个 MSR 的地址。你可以使用 CPU 指令 rdmsr 和 wrmsr 分别读取和写入 MSR。还有一
些命令行工具允许你读取和写入 MSR，但不建议这样做，因为更改这些值（尤其是当系统正在运行
时）如果不小心是非常危险的。如果你不介意可能会使系统不稳定或不可逆地损坏数据，你可以通过
安装 msr-tools 并加载 msr 内核模块来读取和写入 MSR，我们后面会看到，一些系统调用方法会
使用 MSR。 ::

    % sudo apt-get install msr-tools
    % sudo modprobe msr
    % sudo rdmsr

使用汇编语言进行系统调用是个糟糕的主意，一个主要原因是，某些系统调用在 glibc 中在系统调
用运行之前或之后会运行额外的代码。在下面的例子中，我们将使用 exit 系统调用。但事实是程
序可以通过 atexit 注册一些函数，这些函数需要在 exit 调用时执行。这些函数是从 glibc 中
执行的，而不是内核。因此，如果你像我们下面展示的那样自己编写汇编代码来调用 exit，你注册
的处理函数将不会被执行，因为你绕过了 glibc。尽管如此，手动使用汇编发起系统调用是一个很
好的学习体验。

遗留或老式系统调用，根据我们的前提知识，我们知道可以通过触发软件中断来使内核执行，我们可
以通过 int 汇编指令来触发软件中断。Linux 内核预留了一个特定的软件中断编号，供用户空间
程序用来进入内核并执行系统调用。Linux 内核为中断编号 128（0x80）注册了一个名为
ia32_syscall 的中断处理程序。让我们看看实际执行此操作的代码。来自内核 3.13.0 源代码
arch/x86/kernel/traps.c 中的 trap_init 函数，其中 IA32_SYSCALL_VECTOR 在
arch/x86/include/asm/irq_vectors.h 中定义为 0x80。 ::

    void __init trap_init(void) {
        /* ..... 其他代码 ... */
    set_system_intr_gate(IA32_SYSCALL_VECTOR, ia32_syscall);

但是，如果内核为用户程序预留了一个软件中断来触发内核，内核是如何知道应该执行许多系统调用
中的哪一个呢？用户程序需要将系统调用编号放入 eax 寄存器中。系统调用本身的参数应该放在剩
余的通用寄存器中。一个记录这一点的地方是在 arch/x86/ia32/ia32entry.S 中的注释： ::

    模拟 IA32 通过 int 0x80 执行系统调用
    参数：
    %eax 系统调用编号
    %ebx 参数 1
    %ecx 参数 2
    %edx 参数 3
    %esi 参数 4
    %edi 参数 5
    %ebp 参数 6    [注意：不会在栈帧中保存，不应触碰]

.. _Linux 内核源码: https://github.com/torvalds/linux/blob/v3.13/

现在我们知道了如何发起系统调用以及参数应该放在哪里，让我们尝试通过编写一些内联汇编来发起
一个系统调用。在这个例子中，我们将尝试调用 exit，它接受一个参数：退出状态。首先，我们需
要找到 exit 的系统调用编号。 `Linux 内核源码`_ 包含一个文件
arch/x86/syscalls/syscall_32.tbl，其中列出了每个系统调用的表格。这个文件在构建时由各
种脚本处理，以生成用户程序可以使用的头文件。 ::

    1 i386  exit   sys_exit

可以看到 exit 系统调用的编号是 1。根据上面描述的接口，我们只需要将系统调用编号移入 eax
寄存器，将第一个参数（退出状态）移入 ebx，我们将退出状态设置 42，这个例子可以简化，但我
认为让它比必要的更冗长会很有趣，这样那些以前没有见过 GCC 内联汇编的人可以将此作为示例或
参考。 ::

    int main(int argc, char *argv[])
    {
        unsigned int syscall_nr = 1;
        int exit_status = 42;

        asm (
            "movl %0, %%eax \n"
            "movl %1, %%ebx \n"
            "int $0x80      \n"
            : /* 无输出参数，我们没有输出任何东西，所以没有 */
            : "m" (syscall_nr), "m" (exit_status) /* 输入参数映射 %0 和 %1 */
            : "eax", "ebx" /* 我们 “破坏” 的寄存器，其实不需要因为调用了 exit */
        );
    }

接下来，编译、执行并检查退出状态，成功了： ::

    $ gcc -o test test.c
    $ ./test
    $ echo $?
    42

**内核处理中断**

内核端：int $0x80 入口点。既然我们已经看到了如何从用户程序触发系统调用，让我们看看内核
是如何使用系统调用编号来执行系统调用代码的。回顾上面的内容，内核注册了一个名为
ia32_syscall 的系统调用处理函数。这个函数是在 arch/x86/ia32/ia32entry.S 中用汇编实现
的，我们可以在该函数中看到几件重要的事情，其中最重要的是对实际系统调用本身的调用： ::

    ia32_do_call:
        IA32_ARG_FIXUP
        call *ia32_sys_call_table(,%rax,8) # xxx: rip 相对调用

IA32_ARG_FIXUP 是一个宏，它重新排列老式参数，以便它们可以被当前系统调用层正确理解。
ia32_sys_call_table 标识符指的是在 arch/x86/ia32/syscall_ia32.c 中定义的一个表。注
意代码末尾的 #include 行： ::

    const sys_call_ptr_t ia32_sys_call_table[__NR_ia32_syscall_max+1] = {
        [0 ... __NR_ia32_syscall_max] = &compat_ni_syscall,
    #include <asm/syscalls_32.h>
    };

回想一下，我们之前看到了在 arch/x86/syscalls/syscall_32.tbl 中定义的系统调用表。
在构建时会运行几个脚本，这些脚本会处理这个表并从中生成 syscalls_32.h 文件。生成的头文
件由有效的 C 代码组成，这些代码通过上面的 #include 插入，以使用系统调用编号索引填充
ia32_sys_call_table 的函数地址。这就是通过老式系统调用进入内核的方式。

**内核中断返回**

从老式系统调用返回：使用 iret。我们已经看到了如何通过软件中断进入内核，但内核在完成运行
后是如何返回给用户程序并降低特权级别的呢？如果我们查阅英特尔软件开发人员手册，我们可以找
到一个有助于说明程序栈在特权级别变更时如何排列的图表。当用户程序通过执行软件中断将执行权
转移到内核函数 ia32_syscall 时，会发生特权级别变更。结果是，当进入 ia32_syscall 时，
栈看起来如下： ::

            SS
            ESP
            EFLAGS
            CS
            EIP 返回用户空间的地址
    ESP --> Error Code

这意味着返回地址和编码特权级别（以及其他内容）的 CPU 标志等都被保存在程序栈中，然后
ia32_syscall 执行。因此，为了恢复执行，内核只需要从程序栈中复制回它们所属的寄存器值，
然后执行就会在用户空间恢复。那么，如何做到这一点呢？有几种方法可以做到这一点，但其中一种
最简单的方法是使用 iret 指令。英特尔指令集手册解释说，iret 指令会按照它们准备的顺序从
栈中弹出返回地址和保存的寄存器值：就像实模式中断返回一样，IRET 指令从栈中弹出返回指令指
针、返回代码段选择器和 EFLAGS 标志，分别加载到 EIP、CS 和 EFLAGS 寄存器中，然后恢复被
中断的程序或过程的执行。

在 Linux 内核中找到这段代码有点困难，因为它隐藏在几个宏后面，需要对信号和 ptrace 系统
调用特别小心避免跟丢。最终，内核汇编代码中的宏揭示了是 iret 负责从系统调用中返回到用户
程序。arch/x86/kernel/entry_64.S 中的 irq_return，其中 INTERRUPT_RETURN 在
arch/x86/include/asm/irqflags.h 中定义为 iretq。 ::

    irq_return:
        INTERRUPT_RETURN

中断向量表
-----------

中断向量表相关描述位于以下文件或类似的位置：arch\x86\include\asm\irq_vectors.h。
Linux 中断向量布局，共有 256 个可以由 Linux 定义的中断描述表（IDT，Interrupt
Descriptor Table）条目，每个 CPU 都有并且每个条目 8 字节。当给定的向量被触发时，无论
是由 CPU 外部、CPU 内部还是软件触发的事件，CPU 都会使用它们作为跳转表。

Linux 在启动初期就设置了每个条目跳转到的内核代码地址，并且永远不会更改它们。这是 IDT
条目的通用布局：

* 向量 0 ~ 31：系统陷阱和异常 —— 硬编码事件
* 向量 32 ~ 127：设备中断
* 向量 128：遗留 int80 系统调用接口
* 向量 129 ~ FIRST_SYSTEM_VECTOR-1：设备中断
* 向量 FIRST_SYSTEM_VECTOR ~ 255：特殊中断

64 位 x86 每个 CPU 都有自己的 IDT 表，32 位则有一个共享的 IDT 表。本文件
（irq_vectors.h）列举了它们的确切布局： ::

    #define NMI_VECTOR      0x02 // 当编程 APIC 时，这被用作一个中断向量
    #define NR_IRQS_LEGACY  16

    // 可用于外部中断源的 IDT 向量从 0x20 开始，0x80 是系统调用向量，0x30-0x3f 用于
    // ISA 中断。ISA 中断是指与 ISA 总线（Industry Standard Architecture）相关的中
    // 断。ISA 总线是一种早期的计算机硬件总线标准，主要用于连接各种硬件设备，如声卡、
    // 网卡、硬盘控制器等。
    #define FIRST_EXTERNAL_VECTOR   0x20
    #define ISA_IRQ_VECTOR(irq) (((FIRST_EXTERNAL_VECTOR + 16) & ~15) + irq)
    #define IA32_SYSCALL_VECTOR     0x80
    #ifdef CONFIG_X86_32
    #define SYSCALL_VECTOR          0x80
    #endif

    // 被对称多处理器（SMP，Symmetric Multi-Processing）架构使用的特殊 IRQ 向量，范
    // 围 0xf0-0xff。以下部分向量是 “罕见” 的，它们被合并到一个单独的向量
    // CALL_FUNCTION_VECTOR 中，以节省向量空间。TLB、重新调度和本地 APIC 向量对性能
    // 至关重要。
    #define SPURIOUS_APIC_VECTOR        0xff
    #if ((SPURIOUS_APIC_VECTOR & 0x0F) != 0x0F)
    # error SPURIOUS_APIC_VECTOR definition error
    #endif
    #define ERROR_APIC_VECTOR           0xfe
    #define RESCHEDULE_VECTOR           0xfd
    #define CALL_FUNCTION_VECTOR        0xfc
    #define CALL_FUNCTION_SINGLE_VECTOR 0xfb
    #define THERMAL_APIC_VECTOR         0xfa
    #define THRESHOLD_APIC_VECTOR       0xf9
    #define REBOOT_VECTOR               0xf8

    // 发送给主机内核的所有设备消息信号中断（MSI，Message Signaled Interrupts）的中
    // 断通知向量。
    #define POSTED_MSI_NOTIFICATION_VECTOR 0xeb
    #define NR_VECTORS 256
    #ifdef CONFIG_X86_LOCAL_APIC
    #define FIRST_SYSTEM_VECTOR POSTED_MSI_NOTIFICATION_VECTOR
    #else
    #define FIRST_SYSTEM_VECTOR NR_VECTORS
    #endif
    #define NR_EXTERNAL_VECTORS (FIRST_SYSTEM_VECTOR - FIRST_EXTERNAL_VECTOR)
    #define NR_SYSTEM_VECTORS (NR_VECTORS - FIRST_SYSTEM_VECTOR)

**X86硬编码中断**

X86 前 32 个硬编码中断，定义在该文件或相似文件中 arch\x86\include\asm\trapnr.h。其中
NMI（Non-Maskable Interrupt）是不可屏蔽中断。它是一种特殊的硬件中断，不能被普通的中断
屏蔽位（如 CPU 的中断标志位）屏蔽。NMI 通常用于处理一些紧急情况，例如硬件错误、系统故障
或调试目的。 ::

    // FRED、Intel VT-x 和 AMD SVM 使用的事件类型代码
    #define EVENT_TYPE_EXTINT       0   // 外部中断
    #define EVENT_TYPE_RESERVED     1   // 保留
    #define EVENT_TYPE_NMI          2   // NMI
    #define EVENT_TYPE_HWEXC        3   // 硬件来源的陷阱、异常
    #define EVENT_TYPE_SWINT        4   // INT n
    #define EVENT_TYPE_PRIV_SWEXC   5   // INT1
    #define EVENT_TYPE_SWEXC        6   // INTO、INT3
    #define EVENT_TYPE_OTHER        7   // FRED SYSCALL/SYSENTER，VT-x MTF

    // 中断或异常
    #define X86_TRAP_DE         0   // 除零异常（Divide by zero）
    #define X86_TRAP_DB         1   // 调试异常（Debug）INT1 产生 DB trap
    #define X86_TRAP_NMI        2   // 非屏蔽中断（Non-maskable interrupt）
    #define X86_TRAP_BP         3   // 断点异常（Breakpoint）INT3 产生 BP trap
    #define X86_TRAP_OF         4   // 溢出异常（Overflow）INTO 如果OF为1产生中断
    #define X86_TRAP_BR         5   // 超出范围异常（Bound range Exceeded）
    #define X86_TRAP_UD         6   // 无效操作码（Invalid Opcode）
    #define X86_TRAP_NM         7   // 设备不可用（Device Not Available）
    #define X86_TRAP_DF         8   // 双重故障（Double Fault）
    #define X86_TRAP_OLD_MF     9   // 协处理器段超限（Coprocessor Seg Overrun）
    #define X86_TRAP_TS         10  // 无效任务状态段（Invalid TSS）
    #define X86_TRAP_NP         11  // 段不存在（Segment Not Present）
    #define X86_TRAP_SS         12  // 堆栈段异常（Stack Segment Fault）
    #define X86_TRAP_GP         13  // 通用保护异常（General Protection Fault）
    #define X86_TRAP_PF         14  // 页面错误（Page Fault）
    #define X86_TRAP_SPURIOUS   15  // 伪中断（Spurious Interrupt），Intel 保留
    #define X86_TRAP_MF         16  // x87 浮点异常（x87 FP Exception）
    #define X86_TRAP_AC         17  // 对齐检查（Alignment Check）
    #define X86_TRAP_MC         18  // 机器检查（Machine Check）
    #define X86_TRAP_XF         19  // SIMD 浮点异常（SIMD FP Expiton）
    #define X86_TRAP_VE         20  // 虚拟化异常（Virtualization Exception）
    #define X86_TRAP_CP         21  // 控制保护异常（Control Protection Except.）
    #define X86_TRAP_VC         29  // VMM 通信异常（VMM Communication Except.）
    #define X86_TRAP_IRET       32  // IRET 异常（IRET Exception）
                                    // 22 ~ 31 Intel 保留，不要使用

INT n、INTO、INT3 和 BOUND 指令允许程序或任务显式调用中断或异常处理程序。INT n 指令使
用向量作为参数，允许程序调用任何中断处理程序。INTO 指令如果 EFLAGS 寄存器中的溢出标志
（OF）被设置，则显式调用溢出异常（#OF）处理程序。OF 标志表示算术指令的溢出，但它不会自
动引发溢出异常。只有通过以下两种方式之一，才能显式引发溢出异常：执行 INTO 指令；检查 OF
标志，并在标志被设置时，使用参数 4（溢出异常的向量）执行 INT n 指令。这两种处理溢出条件
的方法都允许程序在指令流的特定位置检测溢出。

INT3 指令显式调用断点异常（#BP）处理程序。类似地，INT1 指令（操作码 F1）显式调用调试异
常（#DB）处理程序。BOUND 指令如果发现操作数不在内存中预定义的边界内，则显式调用超出范围
异常（#BR）处理程序。这条指令用于检查对数组和其他数据结构的引用。与溢出异常类似，超出范
围异常只能通过 BOUND 指令或使用参数 5（边界检查异常的向量）的 INT n 指令显式引发。处理
器不会隐式地执行边界检查并引发超出范围异常。

**异常分类**

异常根据报告方式以及导致异常的指令是否可以在不丢失程序或任务连续性的情况下重新启动，被分
类为故障（faults）、陷阱（traps）或中止（aborts）。

* 故障（Faults）

  故障是一种通常可以纠正的异常，一旦纠正后，允许程序重新启动而不会丢失连续性。当报告
  故障时，处理器将机器状态恢复到执行故障指令之前的状态。故障处理程序的返回地址（保存
  的 CS 和 EIP 寄存器的内容）指向故障指令，而不是故障指令之后的指令。

* 陷阱（Traps）

  陷阱是在执行陷阱指令后立即报告的异常。陷阱允许程序或任务在不丢失程序连续性的情况下
  继续执行。陷阱处理程序的返回地址指向陷阱指令之后要执行的指令。

* 中止（Aborts）

  中止不总是能精确报告导致异常的指令发生的位置，且不允许重新启动发生异常的程序或任
  务。中止用于报告严重错误，如硬件错误以及系统表中的不一致或非法值。

故障（Faults）异常的一个异常子集是不可重新启动的异常，这类异常会导致丢失部分处理器状
态。例如，执行一个跨越栈段末尾的栈帧 POPAD 指令，会导致报告故障。在这种情况下，异常处理
程序会看到指令指针（CS:EIP）已恢复，就好像 POPAD 指令未被执行一样。然而，内部处理器状
态（通用寄存器）将被修改。这类情况被视为编程错误。导致此类异常的应用程序应由操作系统终
止。

32位快速调用
------------

::

    # 汇编代码
    .equ SYS_EXIT, 1
    .equ EXIT_CODE, 0

    .section .data

    kernel_vsyscall:
    .4byte 0 # 从辅助向量或getauxval(AT_SYSINFO)获取sysenter包装函数的入口地址

    .section .text

    movl $SYS_EXIT, %eax
    movl $EXIT_CODE, %ebx
    call kernel_vsyscall # 该函数是用户空间函数，它会自动处理系统调用如何返回

    # SYSENTER调用约定
    %eax    system call number
    %ebx    arg1
    %ecx    arg2
    %edx    arg3
    %esi    arg4
    %edi    arg5
    %ebp    user stack %esp
    0(%ebp) arg6

    # 内核包装，vDSO（virtual Dynamic Shared Object，虚拟动态共享对象）
    __kernel_vsyscall: # v3.13 arch/x86/vsdo/vdso32/sysenter.S
    .LSTART_vsyscall:
        push %ecx
        push %edx
        push %ebp
    .Lenter_kernel:
        movl %esp,%ebp
        sysenter
        ...
    VDSO32_SYSENTER_RETURN:
        pop %ebp
        pop %edx
        pop %ecx
        ret

老式方法看起来相当合理，但还有更新的触发系统调用的方法，这些方法不涉及软件中断，比使用软
件中断要快得多。每种更快的方法都由两条指令组成。一条用于进入内核，一条用于离开。这两种方
法在英特尔 CPU 文档中都被描述为 “快速系统调用”。不幸的是，英特尔和 AMD 在决定 CPU 位于
32 位或 64 位模式下，使用哪种方法存在一些分歧。为了在英特尔和 AMD CPU 上实现最大兼容
性：在 32 位系统上使用 sysenter 和 sysexit，在 64 位系统上使用 syscall 和 sysret。

32 位快速系统调用使用 sysenter 发起系统调用比使用老式中断方法更复杂，它需要用户程序
（通过 glibc）和内核之间更多的协调。让我们一步一步来，先弄清楚细节。首先，让我们看看英
特尔指令集参考手册中关于 sysenter 的文档以及如何使用它：

在执行 SYSENTER 指令之前，软件必须通过向以下 MSR 写入值来指定特权级别0的代码段和代码入
口点，以及特权级别0的栈段和栈指针：IA32_SYSENTER_CS（MSR 地址 174H）—— 此 MSR 的低
16 位是特权级别0代码段的段选择器。此值还用于确定特权级别0栈段的段选择器。此值不能表示空
选择器。IA32_SYSENTER_EIP（MSR 地址 176H）—— 此 MSR 的值加载到 RIP（此值指向所选操作
程序或例程的第一条指令），在保护模式下仅加载位 31:0。IA32_SYSENTER_ESP（MSR 地址
175H）—— 此 MSR 的值加载到 RSP（此值包含特权级别0栈的栈指针）。此值不能表示非规范地
址，在保护模式下仅加载位 31:0。

换句话说：为了让内核能够通过 sysenter 接收传入的系统调用，内核必须设置 3 个型号特定寄
存器（MSR）。在我们的例子中，最有趣的 MSR 是 IA32_SYSENTER_EIP（地址为 0x176）。这个
MSR 是内核指定系统调用处理函数的地方，当用户程序执行 sysenter 指令时，最后会使用该函数
去真正进行处理。我们可以在 Linux 内核的 arch/x86/vdso/vdso32-setup.c 中找到写入 MSR
的代码，可以看到 MSR 中写入的这个函数是 ia32_sysenter_target。 ::

    void enable_sep_cpu(void)
    {
        /* ... 其他代码 ... */
        wrmsr(MSR_IA32_SYSENTER_EIP, (unsigned long)ia32_sysenter_target, 0);

其中 MSR_IA32_SYSENTER_EIP 在 arch/x86/include/uapi/asm/msr-index.h 中定义为
0x00000176。与老式软件中断系统调用类似，使用 sysenter 发起系统调用也有对应的调用约定。
一个记录该信息的地方是 arch/x86/ia32/ia32entry.S 中的注释： ::

    32 位 SYSENTER 指令入口
    参数：
    %eax 系统调用编号。
    %ebx 参数 1
    %ecx 参数 2
    %edx 参数 3
    %esi 参数 4
    %edi 参数 5
    %ebp 用户栈
    0(%ebp) 参数 6

回想一下，老式系统调用方法包括一个返回到被中断的用户空间程序的机制：iret 指令。捕获使
sysenter 正常工作所需的逻辑很复杂，因为与软件中断不同，sysenter 不会保存返回地址。内核
在执行 sysenter 指令之前如何做以及做了哪些簿记工作，随着时间的推移而变化（并且它已经发
生了变化，正如你将在下面的 “漏洞” 部分看到的）。为了防止未来的变化，用户程序应该使用一
个名为 __kernel_vsyscall 的函数，该函数在内核中实现，但在进程启动时映射到每个用户进程
中。这有点奇怪；它是内核附带的代码，但在用户空间运行。其实 __kernel_vsyscall 是一个名
为虚拟动态共享对象（vDSO）的一部分，它存在是为了允许程序在用户空间执行内核代码。我们稍
后会深入探讨 vDSO 是什么、它做什么以及它是如何工作的。现在，让我们看看
__kernel_vsyscall 的内部结构。封装 sysenter 调用约定的 __kernel_vsyscall 函数可以在
arch/x86/vdso/vdso32/sysenter.S 中找到： ::

    __kernel_vsyscall:
    .LSTART_vsyscall:
        push %ecx
    .Lpush_ecx:
        push %edx
    .Lpush_edx:
        push %ebp
    .Lenter_kernel:
        movl %esp,%ebp
    sysenter

__kernel_vsyscall 是动态共享对象（也称为共享库）的一部分，用户程序如何在运行时找到该函
数的地址呢？__kernel_vsyscall 函数的地址被写入 ELF 辅助向量中，用户程序或库（通常是
glibc）可以找到它并使用它。有几种方法可以搜索 ELF 辅助向量：通过使用 getauxval 和
AT_SYSINFO 参数，通过迭代到环境变量的末尾并从内存中解析它们。第一种是最简单的选项，但
在 glibc 2.16 之前的版本中不存在。下面的示例代码展示了第二种方法。正如我们在上面的代码
中看到的，__kernel_vsyscall 在执行 sysenter 之前会进行一些簿记工作。因此，以下是手动
调用执行到 sysenter 进入内核的方法：在 ELF 辅助向量中搜索 AT_SYSINFO，
__kernel_vsyscall 的地址就写在这里，然后将系统调用编号和参数放入寄存器，就像我们通常对
老式系统调用一样，最后调用 __kernel_vsyscall 函数。

你绝对不应该编写自己的 sysenter 包装函数，因为内核使用 sysenter 进入和离开系统调用的约
定可能会发生变化，你的代码会出错。你应该始终通过调用 __kernel_vsyscall 来发起
sysenter 系统调用。 ::

    #include <stdlib.h>
    #include <elf.h>

    int main(int argc, char* argv[], char* envp[])
    {
        unsigned int syscall_nr = 1;
        int exit_status = 42;

        // stack [argc][argv][envp][auxiliary vector]
        Elf32_auxv_t *auxv; // 辅助向量位于环境变量末尾之后
        while(*envp++ != NULL);

        // envp 现在指向辅助向量，因为我们已经迭代了环境变量
        for (auxv = (Elf32_auxv_t *)envp; auxv->a_type != AT_NULL; auxv++) {
            if(auxv->a_type == AT_SYSINFO) {
                break;
            }
        }

        // 在 glibc 2.16 及更高版本中可以调用 getauxval(AT_SYSINFO)
        asm(
            "movl %0, %%eax    \n"
            "movl %1, %%ebx    \n"
            "call *%2          \n"
            : /* 无输出参数 */
            : "m" (syscall_nr), "m" (exit_status), "m" (auxv->a_un.a_val)
            : "eax", "ebx" /* 破坏的寄存器 */
        );
    }

ELF 辅助向量是一种机制，用于将某些内核级别的信息传递给用户进程。这些信息包括系统调用入
口函数的地址（AT_SYSINFO）。这类信息是动态的，只有在内核完成加载后才能知晓。 ::

     0 AT_NULL          向量结束
     1 AT_IGNORE        应忽略该条目
     2 AT_EXECFD        程序的文件描述符
     3 AT_PHDR          程序的程序头
     4 AT_PHENT         程序头条目的大小
     5 AT_PHNUM         程序头的数量
     6 AT_PAGESZ        系统页面大小
     7 AT_BASE          解释器的基地址
     8 AT_FLAGS         标志
     9 AT_ENTRY         程序的入口点
    10 AT_NOTELF        程序不是 ELF 格式
    11 AT_UID           真实用户 ID
    12 AT_EUID          有效用户 ID
    13 AT_GID           真实组 ID
    14 AT_EGID          有效组 ID
    15 AT_PLATFORM      用于标识平台的字符串
    16 AT_HWCAP         关于处理器能力的机器依赖提示
    17 AT_CLKTCK        times() 的频率
    18 AT_FPUCW         使用的 FPU 控制字
    19 AT_DCACHEBSIZE   数据缓存块大小
    20 AT_ICACHEBSIZE   指令缓存块大小
    21 AT_UCACHEBSIZE   统一缓存块大小
    22 AT_IGNOREPPC     应忽略该条目
    23 AT_SECURE        布尔值，exec 是否设置了类似 setuid 的权限
    24 AT_BASE_PLATFORM 用于标识真实平台的字符串
    25 AT_RANDOM        16 个随机字节的地址
    31 AT_EXECFN        可执行文件的文件名
    32 AT_SYSINFO       用于定位 vsyscall 入口点，在 64 位模式下，它不会被导出
    33 AT_SYSINFO_EHDR  是包含 vDSO 的页面的起始地址

**内核处理调用**

内核端 sysenter 入口点。让我们看看内核是如何使用系统调用编号来执行系统调用代码的。回顾
前面内容，内核注册了一个名为 ia32_sysenter_target 的系统调用处理函数。这个函数是在
arch/x86/ia32/ia32entry.S 中用汇编实现的。下面是保存系统调用编号的 eax 寄存器的值是
如何用于执行系统调用的，这与我们在老式系统调用模式中看到的代码完全相同：一个名为
ia32_sys_call_table 的表，使用系统调用编号进行索引。完成所有必要的簿记工作后，老式系统
调用模式和 sysenter 系统调用模式都使用相同的机制和系统调用表来分发系统调用。 ::

    sysenter_dispatch:
        call *ia32_sys_call_table(,%rax,8)

**内核调用返回**

使用 sysexit 从 sysenter 系统调用返回。内核可以使用 sysexit 指令恢复到用户程序中继续
执行。使用这条指令并不像使用 iret 那么直接。sysexit 的调用者需要将返回地址放入 rdx 寄
存器中，并将程序栈指针放入 rcx 寄存器中。这意味着必须计算出恢复执行的地址，保存该值并在
调用 sysexit 之前恢复它。可以在 arch/x86/ia32/ia32entry.S 文件中找到执行此操作的代
码： ::

    sysexit_from_sys_call:
        andl    $~TS_COMPAT,TI_status+THREAD_INFO(%rsp,RIP-ARGOFFSET)
        andl    $~0x200,EFLAGS-R11(%rsp) // 清除 IF，防止 popfq 过早启用中断
        movl    RIP-R11(%rsp),%edx       // 用户 %eip
    CFI_REGISTER rip,rdx
        RESTORE_ARGS 0,24,0,0,0,0
        xorq    %r8,%r8
        xorq    %r9,%r9
        xorq    %r10,%r10
        xorq    %r11,%r11
        popfq_cfi
        /*CFI_RESTORE rflags*/
        popq_cfi %rcx                   // 用户 %esp
    CFI_REGISTER rsp,rcx
        TRACE_IRQS_ON
        ENABLE_INTERRUPTS_SYSEXIT32

ENABLE_INTERRUPTS_SYSEXIT32 是在 arch/x86/include/asm/irqflags.h 中定义的一个宏，
其中包含 sysexit 指令。因为 __kernel_vsyscall 已经是用户空间的函数，该函数中的 ret
指令其实就可以自动回到原来用户函数的返回地址，只要这里 sysexit 按照其自己的恢复方式恢复
出 __kernel_vsyscall 的地址回到 __kernel_vsyscall 函数中就可以成功返回到用户函数了。
这就是 32 位快速系统调用的工作原理。

64位快速调用
------------

::

    # 汇编代码
    .equ SYS_EXIT, 60
    .equ EXIT_CODE, 0

    movq $SYS_EXIT, %rax
    movq $EXIT_CODE, %rdi
    syscall # syscall 机器指令

    # SYSCALL调用约定
    %eax    system call number
    %rdi    arg1
    %rsi    arg2
    %rdx    arg3
    %r10    arg4
    %r8     arg5
    %r9     arg6

接下来我们看 64 位快速系统调用，它使用 syscall 和 sysret 指令分别进入和退出系统调用。
英特尔指令集参考手册中解释了 syscall 指令的工作原理：SYSCALL 调用特权级别为0的操作系统
系统调用处理程序，该函数（即下面的 system_call）被写入在 LSTAR MSR 中，内核处理系统调
用时，从这个 MSR 加载其值到 RIP 中去处理系统调用，而内核在此之前已经将用户空间的 RIP
值保存到了 RCX 寄存器中，以便系统调用完毕之后能返回到用户空间。我们可以在内核的
arch/x86/kernel/cpu/common.c 中找到该 MSR 是如何注册的这段代码： ::

    void syscall_init(void)
    {
        /* ... 其他代码 ... */
        wrmsrl(MSR_LSTAR, system_call);

其中 MSR_LSTAR 在 arch/x86/include/uapi/asm/msr-index.h 中定义为 0xc0000082。与老
式软件中断系统调用类似，使用 syscall 发起系统调用也有对应的调用约定。用户程序需要将系统
调用编号放入 rax 寄存器，参数放到对应通用寄存器。相关信息在 x86-64 ABI 中记录如下：

1. 用户应用程序 C 函数调用约定使用整数寄存器 %rdi、%rsi、%rdx、%rcx、%r8 和 %r9，使
   用 syscall 指令时的调用约定使用 %rdi、%rsi、%rdx、%r10、%r8 和 %r9。

2. 通过 syscall 指令发起系统调用。内核会破坏寄存器 %rcx 和 %r11，但会保护其他除了
   %rax外的其他寄存器。

3. 系统调用编号必须放在寄存器 %rax 中。

4. 系统调用限制为六个参数，没有参数直接放在栈上。

5. 从系统调用返回后，寄存器 %rax 包含系统调用的结果。范围在 -4095 到 -1 之间的值表示错
   误，它是 -errno。

6. 只有 INTEGER 类或 MEMORY 类的值能传递给内核。

这也记录在 arch/x86/kernel/entry_64.S 文件的注释中。现在我们知道了如何发起系统调用以
及参数应该放在哪里，让我们尝试通过编写内联汇编来发起一个系统调用。在前面的示例基础上，让
我们构建一个带有内联汇编的小型 C 程序，执行退出状态为 42 的 exit 系统调用。首先，我们
需要找到 64 位系统下 exit 的系统调用编号，对应的是 arch/x86/syscalls/syscall_64.tbl 
文件中的定义的表： ::

    60  common  exit  sys_exit

可以看到在 64 为系统下 exit 系统调用的编号是 60。根据上面描述的接口，我们只需要将 60
移入 rax 寄存器，将第一个参数（退出状态）传到 rdi。 ::

    int main(int argc, char *argv[])
    {
        unsigned long syscall_nr = 60;
        long exit_status = 42;

        asm (
            "movq %0, %%rax \n"
            "movq %1, %%rdi \n"
            "syscall        \n"
            : /* 输出参数 */
            : "m" (syscall_nr), "m" (exit_status) /* 参数映射 */
            : "rax", "rdi" /* 破坏的寄存器 */
        );
    }

**内核处理调用**

内核端：syscall 入口点。既然我们已经看到了如何从用户程序触发系统调用，让我们看看内核是
如何使用系统调用编号来执行系统调用代码的。前面我们看到 system_call 的地址会写入到
LSTAR MSR。让我们看看 arch/x86/kernel/entry_64.S 中的代码，rax 中的系统调用编号是如
何将执行权交给系统调用。 ::

    call *sys_call_table(,%rax,8)  # XXX:    rip 相对调用

与老式系统调用方法类似，sys_call_table 是在 C 文件中定义的一个表，它使用 #include 插
入由脚本生成的 C 代码。从 arch/x86/kernel/syscall_64.c 中，注意底部的 #include： ::

    asmlinkage const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
        [0 ... __NR_syscall_max] = &sys_ni_syscall,
    #include <asm/syscalls_64.h>
    };

我们之前在 arch/x86/syscalls/syscall_64.tbl 中看到了系统调用表。与老式中断模式完全一
样，一个脚本在内核编译时运行，并从 syscall_64.tbl 中的表生成 syscalls_64.h 文件。上
面的代码只是包含了生成的 C 代码，产生了一个由系统调用编号索引的函数指针数组。这就是通过
syscall 系统调用进入内核的方式。

**内核调用返回**

内核可以使用 sysret 指令恢复到用户程序使用 syscall 离开的地方。sysret 比 sysexit 简
单，因为当使用 syscall 时，恢复执行的地址被复制到 rcx 寄存器中。内核只要维护好该值，并
在调用 sysret 之前将其恢复到 rcx，执行就会恢复到 syscall 调用之前离开的地方。我们可以
在 arch/x86/kernel/entry_64.S 中找到执行此操作的代码： ::

    movq    RIP-ARGOFFSET(%rsp),%rcx
    CFI_REGISTER    rip,rcx
    RESTORE_ARGS    1,-ARG_SKIP,0
    /*CFI_REGISTER  rflags,r11*/
    movq    PER_CPU_VAR(old_rsp), %rsp
    USERGS_SYSRET64

USERGS_SYSRET64 是在 arch/x86/include/asm/irqflags.h 中定义的一个宏，其中包含
sysret 指令。这就是 64 位快速系统调用的工作原理。

syscall库函数
--------------

::

    # C函数调用约定
    rdi  arg1
    rsi  arg2
    rdx  arg3
    rcx  arg4
    r8   arg5
    r9   arg6
    push arg7 # 其他参数使用栈传递

    long syscall(syscall_number, arg1, arg2, arg3, arg4, arg5, arg6);

    # 汇编代码
    .equ SYS_EXIT, 60
    .equ EXIT_CODE, 0

    movq $SYS_EXIT, %rdi
    movq $EXIT_CODE, %rsi
    call syscall # 调用C函数

使用 syscall(2) 半手动发起系统调用。前面，我们已经看到了如何通过为几种不同的系统调用方
法编写汇编代码来手动发起系统调用。通常，你不需要自己编写汇编代码。glibc 提供了包装函
数，为你处理所有的汇编代码。然而，有一些系统调用没有 glibc 包装函数。一个这样的系统调用
例子是 futex 用于快速用户空间锁定。但是为什么没有 futex 的系统调用包装函数？futex 只打
算由库调用，而不是应用程序代码，因此为了调用 futex，你必须通过以下方式之一：为你要支持
的每个平台生成汇编代码；使用 glibc 提供的 syscall 包装函数。

如果你发现自己需要调用没有包装函数的系统调用，你绝对应该选择第二种方法使用 glibc 中的
syscall 函数。下面我们使用 glibc 中的 syscall 函数，以退出状态 42 调用 exit： ::

    #include <unistd.h>

    int main(int argc, char *argv[])
    {
        unsigned long syscall_nr = 60;
        long exit_status = 42;

        syscall(syscall_nr, exit_status);
    }

下面是来自 sysdeps/unix/sysv/linux/x86_64/syscall.S 文件中的 glibc syscall 函数的
内部结构，我们看看它是如何工作的： ::

    // long syscall(syscall_number, arg1, arg2, arg3, arg4, arg5, arg6)
    // 我们需要对参数进行一些移动，以将系统调用编号放到 rax 中
            .text
    ENTRY (syscall)
            movq %rdi, %rax         /* 系统调用编号 -> rax */
            movq %rsi, %rdi         /* 移动参数 1 - 参数 5 */
            movq %rdx, %rsi
            movq %rcx, %rdx
            movq %r8, %r10
            movq %r9, %r8
            movq 8(%rsp),%r9        /* 参数 6 在栈上 */
            syscall                 /* 执行系统调用 */
            cmpq $-4095, %rax       /* 检查 %rax 是否出错 */
            jae SYSCALL_ERROR_LABEL /* 如果出错，跳转到错误处理程序 */
    L(pseudo_end):
            ret                     /* 返回给调用者 */

我们之前展示了 x86_64 ABI 文档的摘录，描述了用户空间和内核的调用约定。这个汇编代码很
酷，因为它展示了两种调用约定。传递给这个函数的参数遵循用户空间调用约定，但在进入内核之
前，它们被移动到另一组寄存器中，以遵循内核调用约定。这就是 glibc syscall 包装函数的工
作方式。

虚拟系统调用
------------

我们已经介绍了所有进入内核发起系统调用的方法，并展示了如何手动（或半手动）
进行这些调用，以从用户空间切换到内核。如果程序可以在不进入内核的情况下调用某些系统调用，
那该多好啊？这正是 Linux 虚拟动态共享对象（vDSO）存在的原因。Linux vDSO 是内核的一部
分代码，但它被映射到用户程序的地址空间中，在用户空间运行。这个想法是，某些系统调用可以在
不进入内核的情况下使用。其中一个这样的调用是：gettimeofday。调用 gettimeofday 系统调
用的程序实际上并没有进入内核。相反，它们只是调用了一段代码，这段代码由内核提供，但在用户
空间运行。不需要引发软件中断，不需要复杂的 sysenter 或 syscall 簿记。gettimeofday 就
是一个普通的函数调用。当使用 ldd 时你可以看到 vDSO 列在其中第一个条目中： ::

    $ ldd `which bash`
    linux-vdso.so.1 =>  (0x00007fff667ff000)
    libtinfo.so.5 => /lib/x86_64-linux-gnu/libtinfo.so.5 (0x00007f623df7d000)
    libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f623dd79000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f623d9ba000)
    /lib64/ld-linux-x86-64.so.2 (0x00007f623e1ae000)

内核中的 vDSO，让我们看看内核中 vDSO 是如何设置的。你可以在 arch/x86/vdso/ 中找到
vDSO 的源代码。有一些汇编和 C 源文件，以及一个链接脚本。链接脚本是一个很酷的东西，值得
一读。下面是来自 arch/x86/vdso/vdso.lds.S 链接脚本中的内容： ::

    // 指定要从 vDSO 中导出哪些用户空间符号
    VERSION {
        LINUX_2.6 {
        global:
            clock_gettime;
            __vdso_clock_gettime;
            gettimeofday;
            __vdso_gettimeofday;
            getcpu;
            __vdso_getcpu;
            time;
            __vdso_time;
        local: *;
        };
    }

链接脚本很有用，但并不特别为人所熟知。这个链接脚本指定了 vDSO 中导出的符号。我们可以看
到 vDSO 导出了 4 个不同的函数，每个函数都有两个名称。你可以在这个目录中的 C 文件中找到
这些函数的源代码。例如 gettimeofday 在 arch/x86/vdso/vclock_gettime.c 中： ::

    int gettimeofday(struct timeval *, struct timezone *)
        __attribute__((weak, alias("__vdso_gettimeofday")));

这是将 gettimeofday 定义为 __vdso_gettimeofday 的弱别名。同一文件中的
__vdso_gettimeofday 函数包含了当用户程序调用 gettimeofday 系统调用时将在用户空间执行
的实际源代码。

可以手动在内存中定位 vDSO。由于地址空间布局随机化的原因，vDSO 将在程序启动时加载到随机
地址。用户程序如何找到 vDSO 呢？如果你还记得前面在检查 sysenter 系统调用方法时，我们看
到用户程序应该调用 __kernel_vsyscall，而不是自己编写 sysenter 汇编代码。这个函数也是 
vDSO 的一部分。示例代码通过搜索 ELF 辅助头部来定位 __kernel_vsyscall，寻找类型为 
AT_SYSINFO 的头部内容，其中包含了 __kernel_vsyscall 的地址。同样地，为了定位 vDSO，
用户程序可以搜索类型为 AT_SYSINFO_EHDR（指定了 vDSO 所在内存页的起始地址） 的 ELF 辅
助向量头部。内核在程序加载时将地址写入 ELF 头部，这就是正确的地址总是出现在 
AT_SYSINFO_EHDR 和 AT_SYSINFO 中的原因。一旦定位到该头部，用户程序可以解析 ELF 对象
（也可以使用libelf），并根据需要调用 ELF 对象中的函数。

.. _符号版本控制: https://www.akkadia.org/drepper/symbol-versioning

这也意味着 vDSO 可以利用 ELF 的一些有用特性，例如 `符号版本控制`_。在内核文档的
Documentation/vDSO/ 目录中提供了解析和调用 vDSO 中函数的示例。

glibc包装函数
-------------

大多数情况下，人们在不知情的情况下访问 vDSO，因为 glibc 使用前面描述的接口对其进行了抽
象化包装。当程序加载时，动态链接器和加载器会加载程序依赖的 DSO，包括 vDSO。glibc 在解
析正在加载的程序的 ELF 头部时存储有关 vDSO 位置的一些数据。它还包含简短的汇编代码函
数，这些函数会在实际发起系统调用之前在 vDSO 中搜索对应的符号名称。例如文件
sysdeps/unix/sysv/linux/x86_64/gettimeofday.c 中的库函数 gettimeofday： ::

    void *gettimeofday_ifunc (void) __asm__ ("__gettimeofday");

    void *gettimeofday_ifunc (void)
    {
        PREPARE_VERSION(linux26, "LINUX_2.6", 61765110);
        // 如果 vDSO 不可用，我们回退到旧的 vsyscall
        return (_dl_vdso_vsym("gettimeofday", &linux26)
            ?: (void *)VSYSCALL_ADDR_vgettimeofday);
    }
    __asm (".type __gettimeofday, %gnu_indirect_function");

glibc 中的这段代码在 vDSO 中搜索 gettimeofday 函数并返回地址。这被很好地封装在一个间
接函数中。这就是调用 gettimeofday 的程序如何通过 glibc 访问 vDSO，而无需切换到内核模
式、发生特权级别变更或引发软件中断。至此，我们已经展示了 Linux 上 32 位和 64 位 Intel
和 AMD CPU 可用的每一种系统调用方法。

glibc 对系统调用的包装。既然我们在谈论系统调用，顺便提一下 glibc 是如何处理系统调用的
也是有意义的。对于许多系统调用，glibc 只需要一个包装函数，它将参数移入适当的寄存器，然
后执行 syscall 或 int $0x80 指令，或者调用 __kernel_vsyscall。它通过使用一系列在文本
文件中定义的表来实现，这些表由脚本处理并输出 C 代码。例如 sysdeps/unix/syscalls.list
文件描述了一些常见的系统调用。要了解每一列的更多信息，请查看处理此文件的脚本中的注释：
sysdeps/unix/make-syscalls.sh。 ::

    access          -       access          i:si    __access        access
    acct            -       acct            i:S     acct
    chdir           -       chdir           i:s     __chdir  chdir
    chmod           -       chmod           i:si    __chmod  chmod

.. _CVE-2010-3301: http://cve.mitre.org/cgi-bin/cvename.cgi?name=2010-3301
.. _Android ABI 破坏的修复: https://git.kernel.org/pub/scm/linux/kernel/git/tip/tip.git/commit/?id=30bfa7b3488bfb1bb75c9f50a5fcac1832970c60

系统调用相关的一些有趣漏洞。不利用这个机会提及 Linux 中与系统调用相关的两个精彩漏洞，那
将是很遗憾的。第一个安全漏洞 `CVE-2010-3301`_ 允许本地用户获得 root 权限。原因是
x86-64 系统上用户程序发起老式系统调用的汇编代码中存在一个小纰漏。利用代码相当聪明：它在
特定地址使用 mmap 生成一个内存区域，并利用整数溢出导致这段代码被执行。还记得上面老式中
断部分的代码吗，它将执行权交给任意地址，该地址会作为内核代码运行，可以将运行进程的权限提
升到 root。

.. parsed-literal::
    // x86_64 platform does not zero extend the %eax register after the 32-bit
    // entry path to ptrace is used
    call \*ia **32**_sys_call_table(,%rax, **8**)

还记得不要在应用程序代码中硬编码 sysenter ABI 的部分吗？不幸的是，android-x86 的开发
人员犯了这个错误。内核 ABI 发生了变化，突然 android-x86 就停止了工作。内核开发人员最终
选择恢复使用旧的 sysenter ABI，以避免破坏 Android 设备。这是添加到 Linux 内核的
`Android ABI 破坏的修复`_。记住永远不要自己编写 sysenter 汇编代码。如果你确实需要直接
实现它，至少使用前面例子中的代码，通过 __kernel_vsyscall 进行调用。

Linux 内核中的系统调用基础设施极其复杂，有多种不同的方法可以发起系统调用，每种方法都有
其自身的优缺点。通过自己编写汇编代码来发起系统调用通常是个坏主意，因为底层 ABI 可能会变
化。系统内核和 libc 实现（大概率）会选择在你的系统上选择某种发起系统调用的最快方法。如
果你不能使用 glibc 提供的包装函数（或者如果不存在），你至少应该使用 syscall 包装函数，
或者尝试通过 vDSO 提供的 __kernel_vsyscall 进行调用。

系统调用约定
------------

调用约定的源代码位置或类似位置：

* arch\x86\entry\entry_32.S
* arch\x86\entry\entry_64.S

**32位老式系统调用入口**

传统 32 位 x86 Linux 系统调用使用 INT $0x80 指令。INT $0x80 会进入此文件
（entry_32.S）的此处入口（entry_INT80_32）。任何 32 位程序都可以使用此入口点来执行系
统调用。在各种程序和库中可以找到内联的 INT $0x80 实例。

它还被 vDSO 的 __kernel_vsyscall 用作不支持更快系统调用入口（sysenter指令）的硬件平
台的回退。重新启动的 32 位系统调用也会回退到 INT $0x80，无论最初用于执行系统调用的指令
是什么。64 位程序也可以使用 INT $0x80，但它们只能在 64 位内核上运行，因而会进入
entry_INT80_compat 入口函数。

该中断方式与 sysenter 指令的方式相比，被视为是慢速调用方式。在现代硬件上，大多数 libc
实现都不会使用它，除了在进程启动期间之外。

参数传递约定如下： ::

    eax  系统调用编号
    ebx  参数 1
    ecx  参数 2
    edx  参数 3
    esi  参数 4
    edi  参数 5
    ebp  参数 6，因为是非易变寄存器，如果要使用必须先保护该寄存器

**32位SYSENTER指令入口**

如果 X86_FEATURE_SEP 可用，则通过 vDSO 的 __kernel_vsyscall 函数进行 32 位系统调用
并会进入此处（entry_SYSENTER_32）。这是 32 位系统上的首选的系统调用入口。原则上，
SYSENTER 指令 **仅** 应出现在 vDSO 中。在实际中，存在少部分 Android 设备搭载了内联
SYSENTER 指令的 Bionic 版本。但这也从未出现在任何 Google 的 Bionic 版本中，它仅出现
在英特尔提供的部分版本中。

SYSENTER 从预先编程的 MSR 中加载 SS、ESP、CS 和 EIP。清除 RFLAGS 中的 IF 和 VM
（IOW：中断已关闭）。SYSENTER 不会在栈上保存任何内容，也不会保存旧的 EIP（!!!）、ESP
或 EFLAGS。为了避免丢失 EFLAGS.VM（从而可能破坏用户或 vm86 状态），我们通过重新编程
MSR，在 vm86 模式中显式禁用 SYSENTER 指令。

参数传递约定如下： ::

    eax     系统调用编号
    ebx     参数 1
    ecx     参数 2
    edx     参数 3
    esi     参数 4
    edi     参数 5
    ebp     保存用户栈指针
    0(%ebp) 参数 6

**64位SYSCALL指令入口**

这是 64 位系统调用使用的唯一入口点（entry_SYSCALL_64）。硬件接口设计得相当合理，Linux
使用的寄存器到参数的映射与使用 SYSCALL 时可用的寄存器做好很好的适配。SYSCALL 指令可以
在 libc 实现以及其他一些程序和库中找到内联的实例。vDSO 中也有一些 SYSCALL 指令，例如作
为 clock_gettimeofday 的回退版本。

64 位 SYSCALL 将 rip 保存到 rcx，清除 rflags.RF，然后将 rflags 保存到 r11，然后从预
先编程的 MSR 中加载新的 ss、cs 和 rip。rflags 会被另一个 MSR 中的值屏蔽（因此不需要
CLD 和 CLAC）。SYSCALL 不会在栈上保存任何内容，也不会改变 rsp。寄存器中最多有 6 个参
数。通过 syscall 指令发起系统调用，内核会破坏寄存器 %rcx 和 %r11。从系统调用返回后，寄
存器 %rax 包含系统调用的结果。范围在 -4095 到 -1 之间的值表示错误，它是 -errno。只有
INTEGER 类或 MEMORY 类的值能传递给内核。

用户调用 syscall 触发内核进入此入口（entry_SYSCALL_64）时的寄存器值： ::

    rax  系统调用编号
    rcx  返回地址
    r11  保存的 rflags（r11 在 C ABI 中被调函数可以随意修改，调用者按需自行保护）
    rdi  参数 1
    rsi  参数 2
    rdx  参数 3
    r10  参数 4（不同于系统调用约定，C ABI 函数调用约定第4个参数保存在 rcx 中）
    r8   参数 5
    r9   参数 6
    注意 r12-r15、rbp、rbx 在 C ABI 中是被调函数负责保护的

只能从用户空间调用 entry_SYSCALL_64。当用户可以更改 pt_regs->foo 时，始终强制使用
IRET。这是因为它更好地处理了非规范地址。由于 AMD 和英特尔 CPU 中的错误，SYSRET 在处理
它们时会有问题。

系统调用列表
------------

执行 ``man syscalls``，括号中的 (2) 表示这是一个内核系统调用： ::

    The list of system calls that are available as at kernel 4.19 (or in a few
    cases only on older kernels) is as follows:

    System call                Kernel        Notes

    _llseek(2)                 1.2
    _newselect(2)              2.0
    _sysctl(2)                 2.0
    accept(2)                  2.0          See notes on socketcall(2)
    accept4(2)                 2.6.28
    access(2)                  1.0
    acct(2)                    1.0
    add_key(2)                 2.6.10
    adjtimex(2)                1.0
    alarm(2)                   1.0
    alloc_hugepages(2)         2.5.36       Removed in 2.5.44
    arc_gettls(2)              3.9          ARC only
    arc_settls(2)              3.9          ARC only
    arc_usr_cmpxchg(2)         4.9          ARC only
    arch_prctl(2)              2.6          x86_64, x86 since 4.12
    atomic_barrier(2)          2.6.34       m68k only
    atomic_cmpxchg_32(2)       2.6.34       m68k only
    bdflush(2)                 1.2          Deprecated (does nothing)
                                            since 2.6
    bfin_spinlock(2)           2.6.22       Blackfin only (port removed
                                            in Linux 4.17)
    bind(2)                    2.0          See notes on socketcall(2)
    bpf(2)                     3.18
    brk(2)                     1.0
    breakpoint(2)              2.2          ARM OABI only, defined with
                                            __ARM_NR prefix
    cacheflush(2)              1.2          Not on x86
    capget(2)                  2.2
    capset(2)                  2.2
    chdir(2)                   1.0
    chmod(2)                   1.0
    chown(2)                   2.2          See chown(2) for version details
    chown32(2)                 2.4
    chroot(2)                  1.0
    clock_adjtime(2)           2.6.39
    clock_getres(2)            2.6
    clock_gettime(2)           2.6
    clock_nanosleep(2)         2.6
    clock_settime(2)           2.6
    clone2(2)                  2.4          IA-64 only
    clone(2)                   1.0
    clone3(2)                  5.3
    close(2)                   1.0
    cmpxchg_badaddr(2)         2.6.36       Tile only (port removed
                                            in Linux 4.17)
    connect(2)                 2.0          See notes on socketcall(2)
    copy_file_range(2)         4.5
    creat(2)                   1.0
    create_module(2)           1.0          Removed in 2.6
    delete_module(2)           1.0
    dma_memcpy(2)              2.6.22       Blackfin only (port removed
                                            in Linux 4.17)
    dup(2)                     1.0
    dup2(2)                    1.0
    dup3(2)                    2.6.27
    epoll_create(2)            2.6
    epoll_create1(2)           2.6.27
    epoll_ctl(2)               2.6
    epoll_pwait(2)             2.6.19
    epoll_wait(2)              2.6
    eventfd(2)                 2.6.22
    eventfd2(2)                2.6.27
    execv(2)                   2.0          SPARC/SPARC64 only, for
                                            compatibility with SunOS
    execve(2)                  1.0
    execveat(2)                3.19
    exit(2)                    1.0
    exit_group(2)              2.6
    faccessat(2)               2.6.16
    fadvise64(2)               2.6
    fadvise64_64(2)            2.6
    fallocate(2)               2.6.23
    fanotify_init(2)           2.6.37
    fanotify_mark(2)           2.6.37
    fchdir(2)                  1.0
    fchmod(2)                  1.0
    fchmodat(2)                2.6.16
    fchown(2)                  1.0
    fchown32(2)                2.4
    fchownat(2)                2.6.16
    fcntl(2)                   1.0
    fcntl64(2)                 2.4
    fdatasync(2)               2.0
    fgetxattr(2)               2.6; 2.4.18
    finit_module(2)            3.8
    flistxattr(2)              2.6; 2.4.18
    flock(2)                   2.0
    fork(2)                    1.0
    free_hugepages(2)          2.5.36       Removed in 2.5.44
    fremovexattr(2)            2.6; 2.4.18
    fsconfig(2)                5.2
    fsetxattr(2)               2.6; 2.4.18
    fsmount(2)                 5.2
    fsopen(2)                  5.2
    fspick(2)                  5.2
    fstat(2)                   1.0
    fstat64(2)                 2.4
    fstatat64(2)               2.6.16
    fstatfs(2)                 1.0
    fstatfs64(2)               2.6
    fsync(2)                   1.0
    ftruncate(2)               1.0
    ftruncate64(2)             2.4
    futex(2)                   2.6
    futimesat(2)               2.6.16
    get_kernel_syms(2)         1.0          Removed in 2.6
    get_mempolicy(2)           2.6.6
    get_robust_list(2)         2.6.17
    get_thread_area(2)         2.6
    get_tls(2)                 4.15         ARM OABI only, has
                                            __ARM_NR prefix
    getcpu(2)                  2.6.19
    getcwd(2)                  2.2
    getdents(2)                2.0
    getdents64(2)              2.4
    getdomainname(2)           2.2          SPARC, SPARC64; available
                                            as osf_getdomainname(2)
                                            on Alpha since Linux 2.0
    getdtablesize(2)           2.0          SPARC (removed in 2.6.26),
                                            available since Linux 2.0 on Alpha
                                            as osf_getdtable-size(2)
    getegid(2)                 1.0
    getegid32(2)               2.4
    geteuid(2)                 1.0
    geteuid32(2)               2.4
    getgid(2)                  1.0
    getgid32(2)                2.4
    getgroups(2)               1.0
    getgroups32(2)             2.4
    gethostname(2)             2.0          Alpha, was available on
                                            SPARC up to Linux 2.6.26
    getitimer(2)               1.0
    getpeername(2)             2.0          See notes on socketcall(2)
    getpagesize(2)             2.0          Not on x86
    getpgid(2)                 1.0
    getpgrp(2)                 1.0
    getpid(2)                  1.0
    getppid(2)                 1.0
    getpriority(2)             1.0
    getrandom(2)               3.17
    getresgid(2)               2.2
    getresgid32(2)             2.4
    getresuid(2)               2.2
    getresuid32(2)             2.4
    getrlimit(2)               1.0
    getrusage(2)               1.0
    getsid(2)                  2.0
    getsockname(2)             2.0          See notes on socketcall(2)
    getsockopt(2)              2.0          See notes on socketcall(2)
    gettid(2)                  2.4.11
    gettimeofday(2)            1.0
    getuid(2)                  1.0
    getuid32(2)                2.4
    getunwind(2)               2.4.8        IA-64 only; deprecated
    getxattr(2)                2.6; 2.4.18
    getxgid(2)                 2.0          Alpha only; see NOTES
    getxpid(2)                 2.0          Alpha only; see NOTES
    getxuid(2)                 2.0          Alpha only; see NOTES
    init_module(2)             1.0
    inotify_add_watch(2)       2.6.13
    inotify_init(2)            2.6.13
    inotify_init1(2)           2.6.27
    inotify_rm_watch(2)        2.6.13
    io_cancel(2)               2.6
    io_destroy(2)              2.6
    io_getevents(2)            2.6
    io_pgetevents(2)           4.18
    io_setup(2)                2.6
    io_submit(2)               2.6
    io_uring_enter(2)          5.1
    io_uring_register(2)       5.1
    io_uring_setup(2)          5.1
    ioctl(2)                   1.0
    ioperm(2)                  1.0
    iopl(2)                    1.0
    ioprio_get(2)              2.6.13
    ioprio_set(2)              2.6.13
    ipc(2)                     1.0
    kcmp(2)                    3.5
    kern_features(2)           3.7          SPARC64 only
    kexec_file_load(2)         3.17
    kexec_load(2)              2.6.13
    keyctl(2)                  2.6.10
    kill(2)                    1.0
    lchown(2)                  1.0          See chown(2) for version details
    lchown32(2)                2.4
    lgetxattr(2)               2.6; 2.4.18
    link(2)                    1.0
    linkat(2)                  2.6.16
    listen(2)                  2.0          See notes on socketcall(2)
    listxattr(2)               2.6; 2.4.18
    llistxattr(2)              2.6; 2.4.18
    lookup_dcookie(2)          2.6
    lremovexattr(2)            2.6; 2.4.18
    lseek(2)                   1.0
    lsetxattr(2)               2.6; 2.4.18
    lstat(2)                   1.0
    lstat64(2)                 2.4
    madvise(2)                 2.4
    mbind(2)                   2.6.6
    memory_ordering(2)         2.2          SPARC64 only
    metag_get_tls(2)           3.9          Metag only (port removed
                                            in Linux 4.17)
    metag_set_fpu_flags(2)     3.9          Metag only (port removed
                                            in Linux 4.17)
    metag_set_tls(2)           3.9          Metag only (port removed
                                            in Linux 4.17)
    metag_setglobalbit(2)      3.9          Metag only (port removed
                                            in Linux 4.17)
    membarrier(2)              3.17
    memfd_create(2)            3.17
    migrate_pages(2)           2.6.16
    mincore(2)                 2.4
    mkdir(2)                   1.0
    mkdirat(2)                 2.6.16
    mknod(2)                   1.0
    mknodat(2)                 2.6.16
    mlock(2)                   2.0
    mlock2(2)                  4.4
    mlockall(2)                2.0
    mmap(2)                    1.0
    mmap2(2)                   2.4
    modify_ldt(2)              1.0
    mount(2)                   1.0
    move_mount(2)              5.2
    move_pages(2)              2.6.18
    mprotect(2)                1.0
    mq_getsetattr(2)           2.6.6
    mq_notify(2)               2.6.6
    mq_open(2)                 2.6.6
    mq_timedreceive(2)         2.6.6
    mq_timedsend(2)            2.6.6
    mq_unlink(2)               2.6.6
    mremap(2)                  2.0
    msgctl(2)                  2.0          See notes on ipc(2)
    msgget(2)                  2.0          See notes on ipc(2)
    msgrcv(2)                  2.0          See notes on ipc(2)
    msgsnd(2)                  2.0          See notes on ipc(2)
    msync(2)                   2.0
    munlock(2)                 2.0
    munlockall(2)              2.0
    munmap(2)                  1.0
    name_to_handle_at(2)       2.6.39
    nanosleep(2)               2.0
    newfstatat(2)              2.6.16       See stat(2)
    nfsservctl(2)              2.2          Removed in 3.1
    nice(2)                    1.0
    old_adjtimex(2)            2.0          Alpha only; see NOTES
    old_getrlimit(2)           2.4          Old variant of getrlimit(2)
                                            that used a different value
                                            for RLIM_INFINITY
    oldfstat(2)                1.0
    oldlstat(2)                1.0
    oldolduname(2)             1.0
    oldstat(2)                 1.0
    oldumount(2)               2.4.116      Name of the old umount(2)
                                            syscall on Alpha
    olduname(2)                1.0
    open(2)                    1.0
    open_by_handle_at(2)       2.6.39
    open_tree(2)               5.2
    openat(2)                  2.6.16
    or1k_atomic(2)             3.1          OpenRISC 1000 only
    pause(2)                   1.0
    pciconfig_iobase(2)        2.2.15; 2.4  Not on x86
    pciconfig_read(2)          2.0.26; 2.2  Not on x86
    pciconfig_write(2)         2.0.26; 2.2  Not on x86
    perf_event_open(2)         2.6.31       Was perf_counter_open() in
                                            2.6.31; renamed in 2.6.32
    personality(2)             1.2
    perfctr(2)                 2.2          SPARC only; removed in 2.6.34
    perfmonctl(2)              2.4          IA-64 only
    pidfd_send_signal(2)       5.1
    pidfd_open(2)              5.3
    pipe(2)                    1.0
    pipe2(2)                   2.6.27
    pivot_root(2)              2.4
    pkey_alloc(2)              4.8
    pkey_free(2)               4.8
    pkey_mprotect(2)           4.8
    poll(2)                    2.0.36; 2.2
    ppoll(2)                   2.6.16
    prctl(2)                   2.2
    pread(2)                                Used for pread64(2) on AVR32
                                            (port removed in Linux 4.12)
                                            and Blackfin (port removed
                                            in Linux 4.17)
    pread64(2)                              Added as "pread" in 2.2;
                                            renamed "pread64" in 2.6
    preadv(2)                  2.6.30
    preadv2(2)                 4.6
    prlimit64(2)               2.6.36
    process_vm_readv(2)        3.2
    process_vm_writev(2)       3.2
    pselect6(2)                2.6.16
    ptrace(2)                  1.0
    pwrite(2)                               Used for pwrite64(2) on AVR32 (port
                                            removed in Linux 4.12) and Blackfin
                                            (port removed in Linux 4.17)
    pwrite64(2)                             Added as "pwrite" in 2.2;
                                            renamed "pwrite64" in 2.6
    pwritev(2)                 2.6.30
    pwritev2(2)                4.6
    query_module(2)            2.2          Removed in 2.6
    quotactl(2)                1.0
    read(2)                    1.0
    readahead(2)               2.4.13
    readdir(2)                 1.0
    readlink(2)                1.0
    readlinkat(2)              2.6.16
    readv(2)                   2.0
    reboot(2)                  1.0
    recv(2)                    2.0          See notes on socketcall(2)
    recvfrom(2)                2.0          See notes on socketcall(2)
    recvmsg(2)                 2.0          See notes on socketcall(2)
    recvmmsg(2)                2.6.33
    remap_file_pages(2)        2.6          Deprecated since 3.16
    removexattr(2)             2.6; 2.4.18
    rename(2)                  1.0
    renameat(2)                2.6.16
    renameat2(2)               3.15
    request_key(2)             2.6.10
    restart_syscall(2)         2.6
    riscv_flush_icache(2)      4.15         RISC-V only
    rmdir(2)                   1.0
    rseq(2)                    4.18
    rt_sigaction(2)            2.2
    rt_sigpending(2)           2.2
    rt_sigprocmask(2)          2.2
    rt_sigqueueinfo(2)         2.2
    rt_sigreturn(2)            2.2
    rt_sigsuspend(2)           2.2
    rt_sigtimedwait(2)         2.2
    rt_tgsigqueueinfo(2)       2.6.31
    rtas(2)                    2.6.2        PowerPC/PowerPC64 only
    s390_runtime_instr(2)      3.7          S390 only
    s390_pci_mmio_read(2)      3.19         S390 only
    s390_pci_mmio_write(2)     3.19         S390 only
    s390_sthyi(2)              4.15         S390 only
    s390_guarded_storage(2)    4.12         S390 only
    sched_get_affinity(2)      2.6          Name of sched_getaffinity(2)
                                            on SPARC and SPARC64
    sched_get_priority_max(2)  2.0
    sched_get_priority_min(2)  2.0
    sched_getaffinity(2)       2.6
    sched_getattr(2)           3.14
    sched_getparam(2)          2.0
    sched_getscheduler(2)      2.0
    sched_rr_get_interval(2)   2.0
    sched_set_affinity(2)      2.6          Name of sched_setaffinity(2)
                                            on SPARC and SPARC64
    sched_setaffinity(2)       2.6
    sched_setattr(2)           3.14
    sched_setparam(2)          2.0
    sched_setscheduler(2)      2.0
    sched_yield(2)             2.0
    seccomp(2)                 3.17
    select(2)                  1.0
    semctl(2)                  2.0          See notes on ipc(2)
    semget(2)                  2.0          See notes on ipc(2)
    semop(2)                   2.0          See notes on ipc(2)
    semtimedop(2)              2.6; 2.4.22
    send(2)                    2.0          See notes on socketcall(2)
    sendfile(2)                2.2
    sendfile64(2)              2.6; 2.4.19
    sendmmsg(2)                3.0
    sendmsg(2)                 2.0          See notes on socketcall(2)
    sendto(2)                  2.0          See notes on socketcall(2)
    set_mempolicy(2)           2.6.6
    set_robust_list(2)         2.6.17
    set_thread_area(2)         2.6
    set_tid_address(2)         2.6
    set_tls(2)                 2.6.11       ARM OABI/EABI only (constant
                                            has __ARM_NR prefix)
    setdomainname(2)           1.0
    setfsgid(2)                1.2
    setfsgid32(2)              2.4
    setfsuid(2)                1.2
    setfsuid32(2)              2.4
    setgid(2)                  1.0
    setgid32(2)                2.4
    setgroups(2)               1.0
    setgroups32(2)             2.4
    sethae(2)                  2.0          Alpha only; see NOTES
    sethostname(2)             1.0
    setitimer(2)               1.0
    setns(2)                   3.0
    setpgid(2)                 1.0
    setpgrp(2)                 2.0          Alternative name for
                                            setpgid(2) on Alpha
    setpriority(2)             1.0
    setregid(2)                1.0
    setregid32(2)              2.4
    setresgid(2)               2.2
    setresgid32(2)             2.4
    setresuid(2)               2.2
    setresuid32(2)             2.4
    setreuid(2)                1.0
    setreuid32(2)              2.4
    setrlimit(2)               1.0
    setsid(2)                  1.0
    setsockopt(2)              2.0          See notes on socketcall(2)
    settimeofday(2)            1.0
    setuid(2)                  1.0
    setuid32(2)                2.4
    setup(2)                   1.0          Removed in 2.2
    setxattr(2)                2.6; 2.4.18
    sgetmask(2)                1.0
    shmat(2)                   2.0          See notes on ipc(2)
    shmctl(2)                  2.0          See notes on ipc(2)
    shmdt(2)                   2.0          See notes on ipc(2)
    shmget(2)                  2.0          See notes on ipc(2)
    shutdown(2)                2.0          See notes on socketcall(2)
    sigaction(2)               1.0
    sigaltstack(2)             2.2
    signal(2)                  1.0
    signalfd(2)                2.6.22
    signalfd4(2)               2.6.27
    sigpending(2)              1.0
    sigprocmask(2)             1.0
    sigreturn(2)               1.0
    sigsuspend(2)              1.0
    socket(2)                  2.0          See notes on socketcall(2)
    socketcall(2)              1.0
    socketpair(2)              2.0          See notes on socketcall(2)
    spill(2)                   2.6.13       Xtensa only
    splice(2)                  2.6.17
    spu_create(2)              2.6.16       PowerPC/PowerPC64 only
    spu_run(2)                 2.6.16       PowerPC/PowerPC64 only
    sram_alloc(2)              2.6.22       Blackfin (port removed
                                            in Linux 4.17)
    sram_free(2)               2.6.22       Blackfin (port removed
                                            in Linux 4.17)
    ssetmask(2)                1.0
    stat(2)                    1.0
    stat64(2)                  2.4
    statfs(2)                  1.0
    statfs64(2)                2.6
    statx(2)                   4.11
    stime(2)                   1.0
    subpage_prot(2)            2.6.25       PowerPC/PowerPC64 only
    swapcontext(2)             2.6.3        PowerPC/PowerPC64 only
    switch_endian(2)           4.1          PowerPC64 only
    swapcontext(2)             2.6.3        PowerPC only
    swapoff(2)                 1.0
    swapon(2)                  1.0
    symlink(2)                 1.0
    symlinkat(2)               2.6.16
    sync(2)                    1.0
    sync_file_range(2)         2.6.17
    sync_file_range2(2)        2.6.22
    syncfs(2)                  2.6.39
    sys_debug_setcontext(2)    2.6.11       PowerPC only
    syscall(2)                 1.0          Still available on ARM OABI
                                            and MIPS O32 ABI
    sysfs(2)                   1.2
    sysinfo(2)                 1.0
    syslog(2)                  1.0
    sysmips(2)                 2.6.0        MIPS only
    tee(2)                     2.6.17
    tgkill(2)                  2.6
    time(2)                    1.0
    timer_create(2)            2.6
    timer_delete(2)            2.6
    timer_getoverrun(2)        2.6
    timer_gettime(2)           2.6
    timer_settime(2)           2.6
    timerfd_create(2)          2.6.25
    timerfd_gettime(2)         2.6.25
    timerfd_settime(2)         2.6.25
    times(2)                   1.0
    tkill(2)                   2.6; 2.4.22
    truncate(2)                1.0
    truncate64(2)              2.4
    ugetrlimit(2)              2.4
    umask(2)                   1.0
    umount(2)                  1.0
    umount2(2)                 2.2
    uname(2)                   1.0
    unlink(2)                  1.0
    unlinkat(2)                2.6.16
    unshare(2)                 2.6.16
    uselib(2)                  1.0
    ustat(2)                   1.0
    userfaultfd(2)             4.3
    usr26(2)                   2.4.8.1      ARM OABI only
    usr32(2)                   2.4.8.1      ARM OABI only
    utime(2)                   1.0
    utimensat(2)               2.6.22
    utimes(2)                  2.2
    utrap_install(2)           2.2          SPARC64 only
    vfork(2)                   2.2
    vhangup(2)                 1.0
    vm86old(2)                 1.0          Was "vm86"; renamed in 2.0.28/2.2
    vm86(2)                    2.0.28; 2.2
    vmsplice(2)                2.6.17
    wait4(2)                   1.0
    waitid(2)                  2.6.10
    waitpid(2)                 1.0
    write(2)                   1.0
    writev(2)                  2.0
    xtensa(2)                  2.6.13       Xtensa only

在许多平台上，包括 x86-32，套接字调用都通过 socketcall(2)（glibc 包装函数）进行多路复
用，同样 System V IPC 调用通过 ipc(2) 进行多路复用。尽管在系统调用表中为它们预留了条
目，但以下系统调用在标准内核中并未实现：afs_syscall(2)、break(2)、ftime(2)、
getpmsg(2)、gtty(2)、idle(2)、lock(2)、madvise1(2)、mpx(2)、phys(2)、prof(2)、
profil(2)、putpmsg(2)、security(2)、stty(2)、tuxcall(2)、ulimit(2) 和 
vserver(2)，还可以参阅 unimplemented(2)。然而 ftime(3)、profil(3) 和 ulimit(3) 作
为库例程存在。自内核 2.1.116 起，phys(2) 已被用于 umount(2)；phys(2) 将永远不会被实
现。getpmsg(2) 和 putpmsg(2) 调用是为修补以支持 STREAMS 的内核准备的，可能永远不会出
现在标准内核中。曾经短暂出现过 set_zone_reclaim(2)，它在 Linux 2.6.13 中被添加，在
2.6.16 中被移除；这个系统调用从未对用户空间开放。

32位系统调用编号
----------------

位于 arch\x86\entry\syscalls\syscall_32.tbl： ::

    32-bit system call numbers and entry vectors

    <number> <abi> <name> <entry point> [<compat entry point> <noreturn>]]
    0   i386 restart_syscall sys_restart_syscall
    1   i386 exit sys_exit - noreturn
    2   i386 fork sys_fork
    3   i386 read sys_read
    4   i386 write sys_write
    5   i386 open sys_open compat_sys_open
    6   i386 close sys_close
    7   i386 waitpid sys_waitpid
    8   i386 creat sys_creat
    9   i386 link sys_link
    10  i386 unlink sys_unlink
    11  i386 execve sys_execve compat_sys_execve
    12  i386 chdir sys_chdir
    13  i386 time sys_time32
    14  i386 mknod sys_mknod
    15  i386 chmod sys_chmod
    16  i386 lchown sys_lchown16
    17  i386 break
    18  i386 oldstat sys_stat
    19  i386 lseek sys_lseek compat_sys_lseek
    20  i386 getpid sys_getpid
    21  i386 mount sys_mount
    22  i386 umount sys_oldumount
    23  i386 setuid sys_setuid16
    24  i386 getuid sys_getuid16
    25  i386 stime sys_stime32
    26  i386 ptrace sys_ptrace compat_sys_ptrace
    27  i386 alarm sys_alarm
    28  i386 oldfstat sys_fstat
    29  i386 pause sys_pause
    30  i386 utime sys_utime32
    31  i386 stty
    32  i386 gtty
    33  i386 access sys_access
    34  i386 nice sys_nice
    35  i386 ftime
    36  i386 sync sys_sync
    37  i386 kill sys_kill
    38  i386 rename sys_rename
    39  i386 mkdir sys_mkdir
    40  i386 rmdir sys_rmdir
    41  i386 dup sys_dup
    42  i386 pipe sys_pipe
    43  i386 times sys_times compat_sys_times
    44  i386 prof
    45  i386 brk sys_brk
    46  i386 setgid sys_setgid16
    47  i386 getgid sys_getgid16
    48  i386 signal sys_signal
    49  i386 geteuid sys_geteuid16
    50  i386 getegid sys_getegid16
    51  i386 acct sys_acct
    52  i386 umount2 sys_umount
    53  i386 lock
    54  i386 ioctl sys_ioctl compat_sys_ioctl
    55  i386 fcntl sys_fcntl compat_sys_fcntl64
    56  i386 mpx
    57  i386 setpgid sys_setpgid
    58  i386 ulimit
    59  i386 oldolduname sys_olduname
    60  i386 umask sys_umask
    61  i386 chroot sys_chroot
    62  i386 ustat sys_ustat compat_sys_ustat
    63  i386 dup2 sys_dup2
    64  i386 getppid sys_getppid
    65  i386 getpgrp sys_getpgrp
    66  i386 setsid sys_setsid
    67  i386 sigaction sys_sigaction compat_sys_sigaction
    68  i386 sgetmask sys_sgetmask
    69  i386 ssetmask sys_ssetmask
    70  i386 setreuid sys_setreuid16
    71  i386 setregid sys_setregid16
    72  i386 sigsuspend sys_sigsuspend
    73  i386 sigpending sys_sigpending compat_sys_sigpending
    74  i386 sethostname sys_sethostname
    75  i386 setrlimit sys_setrlimit compat_sys_setrlimit
    76  i386 getrlimit sys_old_getrlimit compat_sys_old_getrlimit
    77  i386 getrusage sys_getrusage compat_sys_getrusage
    78  i386 gettimeofday sys_gettimeofday compat_sys_gettimeofday
    79  i386 settimeofday sys_settimeofday compat_sys_settimeofday
    80  i386 getgroups sys_getgroups16
    81  i386 setgroups sys_setgroups16
    82  i386 select sys_old_select compat_sys_old_select
    83  i386 symlink sys_symlink
    84  i386 oldlstat sys_lstat
    85  i386 readlink sys_readlink
    86  i386 uselib sys_uselib
    87  i386 swapon sys_swapon
    88  i386 reboot sys_reboot
    89  i386 readdir sys_old_readdir compat_sys_old_readdir
    90  i386 mmap sys_old_mmap compat_sys_ia32_mmap
    91  i386 munmap sys_munmap
    92  i386 truncate sys_truncate compat_sys_truncate
    93  i386 ftruncate sys_ftruncate compat_sys_ftruncate
    94  i386 fchmod sys_fchmod
    95  i386 fchown sys_fchown16
    96  i386 getpriority sys_getpriority
    97  i386 setpriority sys_setpriority
    98  i386 profil
    99  i386 statfs sys_statfs compat_sys_statfs
    100 i386 fstatfs sys_fstatfs compat_sys_fstatfs
    101 i386 ioperm sys_ioperm
    102 i386 socketcall sys_socketcall compat_sys_socketcall
    103 i386 syslog sys_syslog
    104 i386 setitimer sys_setitimer compat_sys_setitimer
    105 i386 getitimer sys_getitimer compat_sys_getitimer
    106 i386 stat sys_newstat compat_sys_newstat
    107 i386 lstat sys_newlstat compat_sys_newlstat
    108 i386 fstat sys_newfstat compat_sys_newfstat
    109 i386 olduname sys_uname
    110 i386 iopl sys_iopl
    111 i386 vhangup sys_vhangup
    112 i386 idle
    113 i386 vm86old sys_vm86old sys_ni_syscall
    114 i386 wait4 sys_wait4 compat_sys_wait4
    115 i386 swapoff sys_swapoff
    116 i386 sysinfo sys_sysinfo compat_sys_sysinfo
    117 i386 ipc sys_ipc compat_sys_ipc
    118 i386 fsync sys_fsync
    119 i386 sigreturn sys_sigreturn compat_sys_sigreturn
    120 i386 clone sys_clone compat_sys_ia32_clone
    121 i386 setdomainname sys_setdomainname
    122 i386 uname sys_newuname
    123 i386 modify_ldt sys_modify_ldt
    124 i386 adjtimex sys_adjtimex_time32
    125 i386 mprotect sys_mprotect
    126 i386 sigprocmask sys_sigprocmask compat_sys_sigprocmask
    127 i386 create_module
    128 i386 init_module sys_init_module
    129 i386 delete_module sys_delete_module
    130 i386 get_kernel_syms
    131 i386 quotactl sys_quotactl
    132 i386 getpgid sys_getpgid
    133 i386 fchdir sys_fchdir
    134 i386 bdflush sys_ni_syscall
    135 i386 sysfs sys_sysfs
    136 i386 personality sys_personality
    137 i386 afs_syscall
    138 i386 setfsuid sys_setfsuid16
    139 i386 setfsgid sys_setfsgid16
    140 i386 _llseek sys_llseek
    141 i386 getdents sys_getdents compat_sys_getdents
    142 i386 _newselect sys_select compat_sys_select
    143 i386 flock sys_flock
    144 i386 msync sys_msync
    145 i386 readv sys_readv
    146 i386 writev sys_writev
    147 i386 getsid sys_getsid
    148 i386 fdatasync sys_fdatasync
    149 i386 _sysctl sys_ni_syscall
    150 i386 mlock sys_mlock
    151 i386 munlock sys_munlock
    152 i386 mlockall sys_mlockall
    153 i386 munlockall sys_munlockall
    154 i386 sched_setparam sys_sched_setparam
    155 i386 sched_getparam sys_sched_getparam
    156 i386 sched_setscheduler sys_sched_setscheduler
    157 i386 sched_getscheduler sys_sched_getscheduler
    158 i386 sched_yield sys_sched_yield
    159 i386 sched_get_priority_max sys_sched_get_priority_max
    160 i386 sched_get_priority_min sys_sched_get_priority_min
    161 i386 sched_rr_get_interval sys_sched_rr_get_interval_time32
    162 i386 nanosleep sys_nanosleep_time32
    163 i386 mremap sys_mremap
    164 i386 setresuid sys_setresuid16
    165 i386 getresuid sys_getresuid16
    166 i386 vm86 sys_vm86 sys_ni_syscall
    167 i386 query_module
    168 i386 poll sys_poll
    169 i386 nfsservctl
    170 i386 setresgid sys_setresgid16
    171 i386 getresgid sys_getresgid16
    172 i386 prctl sys_prctl
    173 i386 rt_sigreturn sys_rt_sigreturn compat_sys_rt_sigreturn
    174 i386 rt_sigaction sys_rt_sigaction compat_sys_rt_sigaction
    175 i386 rt_sigprocmask sys_rt_sigprocmask compat_sys_rt_sigprocmask
    176 i386 rt_sigpending sys_rt_sigpending compat_sys_rt_sigpending
    177 i386 rt_sigtimedwait sys_rt_sigtimedwait_time32 compat_*
    178 i386 rt_sigqueueinfo sys_rt_sigqueueinfo compat_sys_rt_sigqueueinfo
    179 i386 rt_sigsuspend sys_rt_sigsuspend compat_sys_rt_sigsuspend
    180 i386 pread64 sys_ia32_pread64
    181 i386 pwrite64 sys_ia32_pwrite64
    182 i386 chown sys_chown16
    183 i386 getcwd sys_getcwd
    184 i386 capget sys_capget
    185 i386 capset sys_capset
    186 i386 sigaltstack sys_sigaltstack compat_sys_sigaltstack
    187 i386 sendfile sys_sendfile compat_sys_sendfile
    188 i386 getpmsg
    189 i386 putpmsg
    190 i386 vfork sys_vfork
    191 i386 ugetrlimit sys_getrlimit compat_sys_getrlimit
    192 i386 mmap2 sys_mmap_pgoff
    193 i386 truncate64 sys_ia32_truncate64
    194 i386 ftruncate64 sys_ia32_ftruncate64
    195 i386 stat64 sys_stat64 compat_sys_ia32_stat64
    196 i386 lstat64 sys_lstat64 compat_sys_ia32_lstat64
    197 i386 fstat64 sys_fstat64 compat_sys_ia32_fstat64
    198 i386 lchown32 sys_lchown
    199 i386 getuid32 sys_getuid
    200 i386 getgid32 sys_getgid
    201 i386 geteuid32 sys_geteuid
    202 i386 getegid32 sys_getegid
    203 i386 setreuid32 sys_setreuid
    204 i386 setregid32 sys_setregid
    205 i386 getgroups32 sys_getgroups
    206 i386 setgroups32 sys_setgroups
    207 i386 fchown32 sys_fchown
    208 i386 setresuid32 sys_setresuid
    209 i386 getresuid32 sys_getresuid
    210 i386 setresgid32 sys_setresgid
    211 i386 getresgid32 sys_getresgid
    212 i386 chown32 sys_chown
    213 i386 setuid32 sys_setuid
    214 i386 setgid32 sys_setgid
    215 i386 setfsuid32 sys_setfsuid
    216 i386 setfsgid32 sys_setfsgid
    217 i386 pivot_root sys_pivot_root
    218 i386 mincore sys_mincore
    219 i386 madvise sys_madvise
    220 i386 getdents64 sys_getdents64
    221 i386 fcntl64 sys_fcntl64 compat_sys_fcntl64
    222 is unused
    223 is unused
    224 i386 gettid sys_gettid
    225 i386 readahead sys_ia32_readahead
    226 i386 setxattr sys_setxattr
    227 i386 lsetxattr sys_lsetxattr
    228 i386 fsetxattr sys_fsetxattr
    229 i386 getxattr sys_getxattr
    230 i386 lgetxattr sys_lgetxattr
    231 i386 fgetxattr sys_fgetxattr
    232 i386 listxattr sys_listxattr
    233 i386 llistxattr sys_llistxattr
    234 i386 flistxattr sys_flistxattr
    235 i386 removexattr sys_removexattr
    236 i386 lremovexattr sys_lremovexattr
    237 i386 fremovexattr sys_fremovexattr
    238 i386 tkill sys_tkill
    239 i386 sendfile64 sys_sendfile64
    240 i386 futex sys_futex_time32
    241 i386 sched_setaffinity sys_sched_setaffinity compat_*
    242 i386 sched_getaffinity sys_sched_getaffinity compat_*
    243 i386 set_thread_area sys_set_thread_area
    244 i386 get_thread_area sys_get_thread_area
    245 i386 io_setup sys_io_setup compat_sys_io_setup
    246 i386 io_destroy sys_io_destroy
    247 i386 io_getevents sys_io_getevents_time32
    248 i386 io_submit sys_io_submit compat_sys_io_submit
    249 i386 io_cancel sys_io_cancel
    250 i386 fadvise64 sys_ia32_fadvise64
    251 is available for reuse (was briefly sys_set_zone_reclaim)
    252 i386 exit_group sys_exit_group - noreturn
    253 i386 lookup_dcookie
    254 i386 epoll_create sys_epoll_create
    255 i386 epoll_ctl sys_epoll_ctl
    256 i386 epoll_wait sys_epoll_wait
    257 i386 remap_file_pages sys_remap_file_pages
    258 i386 set_tid_address sys_set_tid_address
    259 i386 timer_create sys_timer_create compat_sys_timer_create
    260 i386 timer_settime sys_timer_settime32
    261 i386 timer_gettime sys_timer_gettime32
    262 i386 timer_getoverrun sys_timer_getoverrun
    263 i386 timer_delete sys_timer_delete
    264 i386 clock_settime sys_clock_settime32
    265 i386 clock_gettime sys_clock_gettime32
    266 i386 clock_getres sys_clock_getres_time32
    267 i386 clock_nanosleep sys_clock_nanosleep_time32
    268 i386 statfs64 sys_statfs64 compat_sys_statfs64
    269 i386 fstatfs64 sys_fstatfs64 compat_sys_fstatfs64
    270 i386 tgkill sys_tgkill
    271 i386 utimes sys_utimes_time32
    272 i386 fadvise64_64 sys_ia32_fadvise64_64
    273 i386 vserver
    274 i386 mbind sys_mbind
    275 i386 get_mempolicy sys_get_mempolicy
    276 i386 set_mempolicy sys_set_mempolicy
    277 i386 mq_open sys_mq_open compat_sys_mq_open
    278 i386 mq_unlink sys_mq_unlink
    279 i386 mq_timedsend sys_mq_timedsend_time32
    280 i386 mq_timedreceive sys_mq_timedreceive_time32
    281 i386 mq_notify sys_mq_notify compat_sys_mq_notify
    282 i386 mq_getsetattr sys_mq_getsetattr compat_sys_mq_getsetattr
    283 i386 kexec_load sys_kexec_load compat_sys_kexec_load
    284 i386 waitid sys_waitid compat_sys_waitid
    285 # sys_setaltroot
    286 i386 add_key sys_add_key
    287 i386 request_key sys_request_key
    288 i386 keyctl sys_keyctl compat_sys_keyctl
    289 i386 ioprio_set sys_ioprio_set
    290 i386 ioprio_get sys_ioprio_get
    291 i386 inotify_init sys_inotify_init
    292 i386 inotify_add_watch sys_inotify_add_watch
    293 i386 inotify_rm_watch sys_inotify_rm_watch
    294 i386 migrate_pages sys_migrate_pages
    295 i386 openat sys_openat compat_sys_openat
    296 i386 mkdirat sys_mkdirat
    297 i386 mknodat sys_mknodat
    298 i386 fchownat sys_fchownat
    299 i386 futimesat sys_futimesat_time32
    300 i386 fstatat64 sys_fstatat64 compat_sys_ia32_fstatat64
    301 i386 unlinkat sys_unlinkat
    302 i386 renameat sys_renameat
    303 i386 linkat sys_linkat
    304 i386 symlinkat sys_symlinkat
    305 i386 readlinkat sys_readlinkat
    306 i386 fchmodat sys_fchmodat
    307 i386 faccessat sys_faccessat
    308 i386 pselect6 sys_pselect6_time32 compat_sys_pselect6_time32
    309 i386 ppoll sys_ppoll_time32 compat_sys_ppoll_time32
    310 i386 unshare sys_unshare
    311 i386 set_robust_list sys_set_robust_list compat_sys_set_robust_list
    312 i386 get_robust_list sys_get_robust_list compat_sys_get_robust_list
    313 i386 splice sys_splice
    314 i386 sync_file_range sys_ia32_sync_file_range
    315 i386 tee sys_tee
    316 i386 vmsplice sys_vmsplice
    317 i386 move_pages sys_move_pages
    318 i386 getcpu sys_getcpu
    319 i386 epoll_pwait sys_epoll_pwait
    320 i386 utimensat sys_utimensat_time32
    321 i386 signalfd sys_signalfd compat_sys_signalfd
    322 i386 timerfd_create sys_timerfd_create
    323 i386 eventfd sys_eventfd
    324 i386 fallocate sys_ia32_fallocate
    325 i386 timerfd_settime sys_timerfd_settime32
    326 i386 timerfd_gettime sys_timerfd_gettime32
    327 i386 signalfd4 sys_signalfd4 compat_sys_signalfd4
    328 i386 eventfd2 sys_eventfd2
    329 i386 epoll_create1 sys_epoll_create1
    330 i386 dup3 sys_dup3
    331 i386 pipe2 sys_pipe2
    332 i386 inotify_init1 sys_inotify_init1
    333 i386 preadv sys_preadv compat_sys_preadv
    334 i386 pwritev sys_pwritev compat_sys_pwritev
    335 i386 rt_tgsigqueueinfo sys_rt_tgsigqueueinfo compat_*
    336 i386 perf_event_open sys_perf_event_open
    337 i386 recvmmsg sys_recvmmsg_time32 compat_sys_recvmmsg_time32
    338 i386 fanotify_init sys_fanotify_init
    339 i386 fanotify_mark sys_fanotify_mark compat_sys_fanotify_mark
    340 i386 prlimit64 sys_prlimit64
    341 i386 name_to_handle_at sys_name_to_handle_at
    342 i386 open_by_handle_at sys_open_by_handle_at compat_*
    343 i386 clock_adjtime sys_clock_adjtime32
    344 i386 syncfs sys_syncfs
    345 i386 sendmmsg sys_sendmmsg compat_sys_sendmmsg
    346 i386 setns sys_setns
    347 i386 process_vm_readv sys_process_vm_readv
    348 i386 process_vm_writev sys_process_vm_writev
    349 i386 kcmp sys_kcmp
    350 i386 finit_module sys_finit_module
    351 i386 sched_setattr sys_sched_setattr
    352 i386 sched_getattr sys_sched_getattr
    353 i386 renameat2 sys_renameat2
    354 i386 seccomp sys_seccomp
    355 i386 getrandom sys_getrandom
    356 i386 memfd_create sys_memfd_create
    357 i386 bpf sys_bpf
    358 i386 execveat sys_execveat compat_sys_execveat
    359 i386 socket sys_socket
    360 i386 socketpair sys_socketpair
    361 i386 bind sys_bind
    362 i386 connect sys_connect
    363 i386 listen sys_listen
    364 i386 accept4 sys_accept4
    365 i386 getsockopt sys_getsockopt sys_getsockopt
    366 i386 setsockopt sys_setsockopt sys_setsockopt
    367 i386 getsockname sys_getsockname
    368 i386 getpeername sys_getpeername
    369 i386 sendto sys_sendto
    370 i386 sendmsg sys_sendmsg compat_sys_sendmsg
    371 i386 recvfrom sys_recvfrom compat_sys_recvfrom
    372 i386 recvmsg sys_recvmsg compat_sys_recvmsg
    373 i386 shutdown sys_shutdown
    374 i386 userfaultfd sys_userfaultfd
    375 i386 membarrier sys_membarrier
    376 i386 mlock2 sys_mlock2
    377 i386 copy_file_range sys_copy_file_range
    378 i386 preadv2 sys_preadv2 compat_sys_preadv2
    379 i386 pwritev2 sys_pwritev2 compat_sys_pwritev2
    380 i386 pkey_mprotect sys_pkey_mprotect
    381 i386 pkey_alloc sys_pkey_alloc
    382 i386 pkey_free sys_pkey_free
    383 i386 statx sys_statx
    384 i386 arch_prctl sys_arch_prctl compat_sys_arch_prctl
    385 i386 io_pgetevents sys_io_pgetevents_time32 compat_sys_io_pgetevents
    386 i386 rseq sys_rseq
    393 i386 semget sys_semget
    394 i386 semctl sys_semctl compat_sys_semctl
    395 i386 shmget sys_shmget
    396 i386 shmctl sys_shmctl compat_sys_shmctl
    397 i386 shmat sys_shmat  compat_sys_shmat
    398 i386 shmdt sys_shmdt
    399 i386 msgget sys_msgget
    400 i386 msgsnd sys_msgsnd compat_sys_msgsnd
    401 i386 msgrcv sys_msgrcv compat_sys_msgrcv
    402 i386 msgctl sys_msgctl compat_sys_msgctl
    403 i386 clock_gettime64 sys_clock_gettime
    404 i386 clock_settime64 sys_clock_settime
    405 i386 clock_adjtime64 sys_clock_adjtime
    406 i386 clock_getres_time64 sys_clock_getres
    407 i386 clock_nanosleep_time64 sys_clock_nanosleep
    408 i386 timer_gettime64 sys_timer_gettime
    409 i386 timer_settime64 sys_timer_settime
    410 i386 timerfd_gettime64 sys_timerfd_gettime
    411 i386 timerfd_settime64 sys_timerfd_settime
    412 i386 utimensat_time64 sys_utimensat
    413 i386 pselect6_time64 sys_pselect6 compat_sys_pselect6_time64
    414 i386 ppoll_time64 sys_ppoll compat_sys_ppoll_time64
    416 i386 io_pgetevents_time64 sys_io_pgetevents compat_*_time64
    417 i386 recvmmsg_time64 sys_recvmmsg compat_sys_recvmmsg_time64
    418 i386 mq_timedsend_time64 sys_mq_timedsend
    419 i386 mq_timedreceive_time64 sys_mq_timedreceive
    420 i386 semtimedop_time64 sys_semtimedop
    421 i386 rt_sigtimedwait_time64 sys_rt_sigtimedwait compat_*_time64
    422 i386 futex_time64 sys_futex
    423 i386 sched_rr_get_interval_time64 sys_sched_rr_get_interval
    424 i386 pidfd_send_signal sys_pidfd_send_signal
    425 i386 io_uring_setup sys_io_uring_setup
    426 i386 io_uring_enter sys_io_uring_enter
    427 i386 io_uring_register sys_io_uring_register
    428 i386 open_tree sys_open_tree
    429 i386 move_mount sys_move_mount
    430 i386 fsopen sys_fsopen
    431 i386 fsconfig sys_fsconfig
    432 i386 fsmount sys_fsmount
    433 i386 fspick sys_fspick
    434 i386 pidfd_open sys_pidfd_open
    435 i386 clone3 sys_clone3
    436 i386 close_range sys_close_range
    437 i386 openat2 sys_openat2
    438 i386 pidfd_getfd sys_pidfd_getfd
    439 i386 faccessat2 sys_faccessat2
    440 i386 process_madvise sys_process_madvise
    441 i386 epoll_pwait2 sys_epoll_pwait2 compat_sys_epoll_pwait2
    442 i386 mount_setattr sys_mount_setattr
    443 i386 quotactl_fd sys_quotactl_fd
    444 i386 landlock_create_ruleset sys_landlock_create_ruleset
    445 i386 landlock_add_rule sys_landlock_add_rule
    446 i386 landlock_restrict_self sys_landlock_restrict_self
    447 i386 memfd_secret sys_memfd_secret
    448 i386 process_mrelease sys_process_mrelease
    449 i386 futex_waitv sys_futex_waitv
    450 i386 set_mempolicy_home_node sys_set_mempolicy_home_node
    451 i386 cachestat sys_cachestat
    452 i386 fchmodat2 sys_fchmodat2
    453 i386 map_shadow_stack sys_map_shadow_stack
    454 i386 futex_wake sys_futex_wake
    455 i386 futex_wait sys_futex_wait
    456 i386 futex_requeue sys_futex_requeue
    457 i386 statmount sys_statmount
    458 i386 listmount sys_listmount
    459 i386 lsm_get_self_attr sys_lsm_get_self_attr
    460 i386 lsm_set_self_attr sys_lsm_set_self_attr
    461 i386 lsm_list_modules sys_lsm_list_modules
    462 i386 mseal sys_mseal
    463 i386 setxattrat sys_setxattrat
    464 i386 getxattrat sys_getxattrat
    465 i386 listxattrat sys_listxattrat
    466 i386 removexattrat sys_removexattrat

64位系统调用编号
----------------

位于文件 arch\x86\entry\syscalls\syscall_64.tbl。其中 ``__x64_sys_*()`` 对应的汇编
代码是为 ``sys_*()`` 系统调用即时创建的。下表中的 abi 列的值可以是 common、64、以及
x32 三种。 ::

    64-bit system call numbers and entry vectors

    <number> <abi> <name> <entry point> [<compat entry point> [noreturn]]
    0   common    read            sys_read
    1   common    write            sys_write
    2   common    open            sys_open
    3   common    close            sys_close
    4   common    stat            sys_newstat
    5   common    fstat            sys_newfstat
    6   common    lstat            sys_newlstat
    7   common    poll            sys_poll
    8   common    lseek            sys_lseek
    9   common    mmap            sys_mmap
    10  common    mprotect        sys_mprotect
    11  common    munmap            sys_munmap
    12  common    brk            sys_brk
    13  64        rt_sigaction        sys_rt_sigaction
    14  common    rt_sigprocmask        sys_rt_sigprocmask
    15  64        rt_sigreturn        sys_rt_sigreturn
    16  64        ioctl            sys_ioctl
    17  common    pread64            sys_pread64
    18  common    pwrite64        sys_pwrite64
    19  64        readv            sys_readv
    20  64        writev            sys_writev
    21  common    access            sys_access
    22  common    pipe            sys_pipe
    23  common    select            sys_select
    24  common    sched_yield        sys_sched_yield
    25  common    mremap            sys_mremap
    26  common    msync            sys_msync
    27  common    mincore            sys_mincore
    28  common    madvise            sys_madvise
    29  common    shmget            sys_shmget
    30  common    shmat            sys_shmat
    31  common    shmctl            sys_shmctl
    32  common    dup            sys_dup
    33  common    dup2            sys_dup2
    34  common    pause            sys_pause
    35  common    nanosleep        sys_nanosleep
    36  common    getitimer        sys_getitimer
    37  common    alarm            sys_alarm
    38  common    setitimer        sys_setitimer
    39  common    getpid            sys_getpid
    40  common    sendfile        sys_sendfile64
    41  common    socket            sys_socket
    42  common    connect            sys_connect
    43  common    accept            sys_accept
    44  common    sendto            sys_sendto
    45  64        recvfrom        sys_recvfrom
    46  64        sendmsg            sys_sendmsg
    47  64        recvmsg            sys_recvmsg
    48  common    shutdown        sys_shutdown
    49  common    bind            sys_bind
    50  common    listen            sys_listen
    51  common    getsockname        sys_getsockname
    52  common    getpeername        sys_getpeername
    53  common    socketpair        sys_socketpair
    54  64        setsockopt        sys_setsockopt
    55  64        getsockopt        sys_getsockopt
    56  common    clone            sys_clone
    57  common    fork            sys_fork
    58  common    vfork            sys_vfork
    59  64        execve            sys_execve
    60  common    exit            sys_exit - noreturn
    61  common    wait4            sys_wait4
    62  common    kill            sys_kill
    63  common    uname            sys_newuname
    64  common    semget            sys_semget
    65  common    semop            sys_semop
    66  common    semctl            sys_semctl
    67  common    shmdt            sys_shmdt
    68  common    msgget            sys_msgget
    69  common    msgsnd            sys_msgsnd
    70  common    msgrcv            sys_msgrcv
    71  common    msgctl            sys_msgctl
    72  common    fcntl            sys_fcntl
    73  common    flock            sys_flock
    74  common    fsync            sys_fsync
    75  common    fdatasync        sys_fdatasync
    76  common    truncate        sys_truncate
    77  common    ftruncate        sys_ftruncate
    78  common    getdents        sys_getdents
    79  common    getcwd            sys_getcwd
    80  common    chdir            sys_chdir
    81  common    fchdir            sys_fchdir
    82  common    rename            sys_rename
    83  common    mkdir            sys_mkdir
    84  common    rmdir            sys_rmdir
    85  common    creat            sys_creat
    86  common    link            sys_link
    87  common    unlink            sys_unlink
    88  common    symlink            sys_symlink
    89  common    readlink        sys_readlink
    90  common    chmod            sys_chmod
    91  common    fchmod            sys_fchmod
    92  common    chown            sys_chown
    93  common    fchown            sys_fchown
    94  common    lchown            sys_lchown
    95  common    umask            sys_umask
    96  common    gettimeofday        sys_gettimeofday
    97  common    getrlimit        sys_getrlimit
    98  common    getrusage        sys_getrusage
    99  common    sysinfo            sys_sysinfo
    100 common    times            sys_times
    101 64        ptrace            sys_ptrace
    102 common    getuid            sys_getuid
    103 common    syslog            sys_syslog
    104 common    getgid            sys_getgid
    105 common    setuid            sys_setuid
    106 common    setgid            sys_setgid
    107 common    geteuid            sys_geteuid
    108 common    getegid            sys_getegid
    109 common    setpgid            sys_setpgid
    110 common    getppid            sys_getppid
    111 common    getpgrp            sys_getpgrp
    112 common    setsid            sys_setsid
    113 common    setreuid        sys_setreuid
    114 common    setregid        sys_setregid
    115 common    getgroups        sys_getgroups
    116 common    setgroups        sys_setgroups
    117 common    setresuid        sys_setresuid
    118 common    getresuid        sys_getresuid
    119 common    setresgid        sys_setresgid
    120 common    getresgid        sys_getresgid
    121 common    getpgid            sys_getpgid
    122 common    setfsuid        sys_setfsuid
    123 common    setfsgid        sys_setfsgid
    124 common    getsid            sys_getsid
    125 common    capget            sys_capget
    126 common    capset            sys_capset
    127 64        rt_sigpending        sys_rt_sigpending
    128 64        rt_sigtimedwait        sys_rt_sigtimedwait
    129 64        rt_sigqueueinfo        sys_rt_sigqueueinfo
    130 common    rt_sigsuspend        sys_rt_sigsuspend
    131 64        sigaltstack        sys_sigaltstack
    132 common    utime            sys_utime
    133 common    mknod            sys_mknod
    134 64        uselib
    135 common    personality        sys_personality
    136 common    ustat            sys_ustat
    137 common    statfs            sys_statfs
    138 common    fstatfs            sys_fstatfs
    139 common    sysfs            sys_sysfs
    140 common    getpriority        sys_getpriority
    141 common    setpriority        sys_setpriority
    142 common    sched_setparam        sys_sched_setparam
    143 common    sched_getparam        sys_sched_getparam
    144 common    sched_setscheduler    sys_sched_setscheduler
    145 common    sched_getscheduler    sys_sched_getscheduler
    146 common    sched_get_priority_max    sys_sched_get_priority_max
    147 common    sched_get_priority_min    sys_sched_get_priority_min
    148 common    sched_rr_get_interval    sys_sched_rr_get_interval
    149 common    mlock            sys_mlock
    150 common    munlock            sys_munlock
    151 common    mlockall        sys_mlockall
    152 common    munlockall        sys_munlockall
    153 common    vhangup            sys_vhangup
    154 common    modify_ldt        sys_modify_ldt
    155 common    pivot_root        sys_pivot_root
    156 64        _sysctl            sys_ni_syscall
    157 common    prctl            sys_prctl
    158 common    arch_prctl        sys_arch_prctl
    159 common    adjtimex        sys_adjtimex
    160 common    setrlimit        sys_setrlimit
    161 common    chroot            sys_chroot
    162 common    sync            sys_sync
    163 common    acct            sys_acct
    164 common    settimeofday        sys_settimeofday
    165 common    mount            sys_mount
    166 common    umount2            sys_umount
    167 common    swapon            sys_swapon
    168 common    swapoff            sys_swapoff
    169 common    reboot            sys_reboot
    170 common    sethostname        sys_sethostname
    171 common    setdomainname        sys_setdomainname
    172 common    iopl            sys_iopl
    173 common    ioperm            sys_ioperm
    174 64        create_module
    175 common    init_module        sys_init_module
    176 common    delete_module        sys_delete_module
    177 64        get_kernel_syms
    178 64        query_module
    179 common    quotactl        sys_quotactl
    180 64        nfsservctl
    181 common    getpmsg
    182 common    putpmsg
    183 common    afs_syscall
    184 common    tuxcall
    185 common    security
    186 common    gettid            sys_gettid
    187 common    readahead        sys_readahead
    188 common    setxattr        sys_setxattr
    189 common    lsetxattr        sys_lsetxattr
    190 common    fsetxattr        sys_fsetxattr
    191 common    getxattr        sys_getxattr
    192 common    lgetxattr        sys_lgetxattr
    193 common    fgetxattr        sys_fgetxattr
    194 common    listxattr        sys_listxattr
    195 common    llistxattr        sys_llistxattr
    196 common    flistxattr        sys_flistxattr
    197 common    removexattr        sys_removexattr
    198 common    lremovexattr        sys_lremovexattr
    199 common    fremovexattr        sys_fremovexattr
    200 common    tkill            sys_tkill
    201 common    time            sys_time
    202 common    futex            sys_futex
    203 common    sched_setaffinity    sys_sched_setaffinity
    204 common    sched_getaffinity    sys_sched_getaffinity
    205 64        set_thread_area
    206 64        io_setup        sys_io_setup
    207 common    io_destroy        sys_io_destroy
    208 common    io_getevents        sys_io_getevents
    209 64        io_submit        sys_io_submit
    210 common    io_cancel        sys_io_cancel
    211 64        get_thread_area
    212 common    lookup_dcookie
    213 common    epoll_create        sys_epoll_create
    214 64        epoll_ctl_old
    215 64        epoll_wait_old
    216 common    remap_file_pages    sys_remap_file_pages
    217 common    getdents64        sys_getdents64
    218 common    set_tid_address        sys_set_tid_address
    219 common    restart_syscall        sys_restart_syscall
    220 common    semtimedop        sys_semtimedop
    221 common    fadvise64        sys_fadvise64
    222 64        timer_create        sys_timer_create
    223 common    timer_settime        sys_timer_settime
    224 common    timer_gettime        sys_timer_gettime
    225 common    timer_getoverrun    sys_timer_getoverrun
    226 common    timer_delete        sys_timer_delete
    227 common    clock_settime        sys_clock_settime
    228 common    clock_gettime        sys_clock_gettime
    229 common    clock_getres        sys_clock_getres
    230 common    clock_nanosleep        sys_clock_nanosleep
    231 common    exit_group        sys_exit_group - noreturn
    232 common    epoll_wait        sys_epoll_wait
    233 common    epoll_ctl        sys_epoll_ctl
    234 common    tgkill            sys_tgkill
    235 common    utimes            sys_utimes
    236 64        vserver
    237 common    mbind            sys_mbind
    238 common    set_mempolicy        sys_set_mempolicy
    239 common    get_mempolicy        sys_get_mempolicy
    240 common    mq_open            sys_mq_open
    241 common    mq_unlink        sys_mq_unlink
    242 common    mq_timedsend        sys_mq_timedsend
    243 common    mq_timedreceive        sys_mq_timedreceive
    244 64        mq_notify        sys_mq_notify
    245 common    mq_getsetattr        sys_mq_getsetattr
    246 64        kexec_load        sys_kexec_load
    247 64        waitid            sys_waitid
    248 common    add_key            sys_add_key
    249 common    request_key        sys_request_key
    250 common    keyctl            sys_keyctl
    251 common    ioprio_set        sys_ioprio_set
    252 common    ioprio_get        sys_ioprio_get
    253 common    inotify_init        sys_inotify_init
    254 common    inotify_add_watch    sys_inotify_add_watch
    255 common    inotify_rm_watch    sys_inotify_rm_watch
    256 common    migrate_pages        sys_migrate_pages
    257 common    openat            sys_openat
    258 common    mkdirat            sys_mkdirat
    259 common    mknodat            sys_mknodat
    260 common    fchownat        sys_fchownat
    261 common    futimesat        sys_futimesat
    262 common    newfstatat        sys_newfstatat
    263 common    unlinkat        sys_unlinkat
    264 common    renameat        sys_renameat
    265 common    linkat            sys_linkat
    266 common    symlinkat        sys_symlinkat
    267 common    readlinkat        sys_readlinkat
    268 common    fchmodat        sys_fchmodat
    269 common    faccessat        sys_faccessat
    270 common    pselect6        sys_pselect6
    271 common    ppoll            sys_ppoll
    272 common    unshare            sys_unshare
    273 64        set_robust_list        sys_set_robust_list
    274 64        get_robust_list        sys_get_robust_list
    275 common    splice            sys_splice
    276 common    tee            sys_tee
    277 common    sync_file_range        sys_sync_file_range
    278 64        vmsplice        sys_vmsplice
    279 64        move_pages        sys_move_pages
    280 common    utimensat        sys_utimensat
    281 common    epoll_pwait        sys_epoll_pwait
    282 common    signalfd        sys_signalfd
    283 common    timerfd_create        sys_timerfd_create
    284 common    eventfd            sys_eventfd
    285 common    fallocate        sys_fallocate
    286 common    timerfd_settime        sys_timerfd_settime
    287 common    timerfd_gettime        sys_timerfd_gettime
    288 common    accept4            sys_accept4
    289 common    signalfd4        sys_signalfd4
    290 common    eventfd2        sys_eventfd2
    291 common    epoll_create1        sys_epoll_create1
    292 common    dup3            sys_dup3
    293 common    pipe2            sys_pipe2
    294 common    inotify_init1        sys_inotify_init1
    295 64        preadv            sys_preadv
    296 64        pwritev            sys_pwritev
    297 64        rt_tgsigqueueinfo    sys_rt_tgsigqueueinfo
    298 common    perf_event_open        sys_perf_event_open
    299 64        recvmmsg        sys_recvmmsg
    300 common    fanotify_init        sys_fanotify_init
    301 common    fanotify_mark        sys_fanotify_mark
    302 common    prlimit64        sys_prlimit64
    303 common    name_to_handle_at    sys_name_to_handle_at
    304 common    open_by_handle_at    sys_open_by_handle_at
    305 common    clock_adjtime        sys_clock_adjtime
    306 common    syncfs            sys_syncfs
    307 64        sendmmsg        sys_sendmmsg
    308 common    setns            sys_setns
    309 common    getcpu            sys_getcpu
    310 64        process_vm_readv    sys_process_vm_readv
    311 64        process_vm_writev    sys_process_vm_writev
    312 common    kcmp            sys_kcmp
    313 common    finit_module        sys_finit_module
    314 common    sched_setattr        sys_sched_setattr
    315 common    sched_getattr        sys_sched_getattr
    316 common    renameat2        sys_renameat2
    317 common    seccomp            sys_seccomp
    318 common    getrandom        sys_getrandom
    319 common    memfd_create        sys_memfd_create
    320 common    kexec_file_load        sys_kexec_file_load
    321 common    bpf            sys_bpf
    322 64        execveat        sys_execveat
    323 common    userfaultfd        sys_userfaultfd
    324 common    membarrier        sys_membarrier
    325 common    mlock2            sys_mlock2
    326 common    copy_file_range        sys_copy_file_range
    327 64        preadv2            sys_preadv2
    328 64        pwritev2        sys_pwritev2
    329 common    pkey_mprotect        sys_pkey_mprotect
    330 common    pkey_alloc        sys_pkey_alloc
    331 common    pkey_free        sys_pkey_free
    332 common    statx            sys_statx
    333 common    io_pgetevents        sys_io_pgetevents
    334 common    rseq            sys_rseq
    335 common    uretprobe        sys_uretprobe
    # don't use numbers 387 through 423, add new calls after
    # the last 'common' entry
    424 common    pidfd_send_signal    sys_pidfd_send_signal
    425 common    io_uring_setup        sys_io_uring_setup
    426 common    io_uring_enter        sys_io_uring_enter
    427 common    io_uring_register    sys_io_uring_register
    428 common    open_tree        sys_open_tree
    429 common    move_mount        sys_move_mount
    430 common    fsopen            sys_fsopen
    431 common    fsconfig        sys_fsconfig
    432 common    fsmount            sys_fsmount
    433 common    fspick            sys_fspick
    434 common    pidfd_open        sys_pidfd_open
    435 common    clone3            sys_clone3
    436 common    close_range        sys_close_range
    437 common    openat2            sys_openat2
    438 common    pidfd_getfd        sys_pidfd_getfd
    439 common    faccessat2        sys_faccessat2
    440 common    process_madvise        sys_process_madvise
    441 common    epoll_pwait2        sys_epoll_pwait2
    442 common    mount_setattr        sys_mount_setattr
    443 common    quotactl_fd        sys_quotactl_fd
    444 common    landlock_create_ruleset    sys_landlock_create_ruleset
    445 common    landlock_add_rule    sys_landlock_add_rule
    446 common    landlock_restrict_self    sys_landlock_restrict_self
    447 common    memfd_secret        sys_memfd_secret
    448 common    process_mrelease    sys_process_mrelease
    449 common    futex_waitv        sys_futex_waitv
    450 common    set_mempolicy_home_node    sys_set_mempolicy_home_node
    451 common    cachestat        sys_cachestat
    452 common    fchmodat2        sys_fchmodat2
    453 common    map_shadow_stack    sys_map_shadow_stack
    454 common    futex_wake        sys_futex_wake
    455 common    futex_wait        sys_futex_wait
    456 common    futex_requeue        sys_futex_requeue
    457 common    statmount        sys_statmount
    458 common    listmount        sys_listmount
    459 common    lsm_get_self_attr    sys_lsm_get_self_attr
    460 common    lsm_set_self_attr    sys_lsm_set_self_attr
    461 common    lsm_list_modules    sys_lsm_list_modules
    462 common    mseal            sys_mseal
    463 common    setxattrat        sys_setxattrat
    464 common    getxattrat        sys_getxattrat
    465 common    listxattrat        sys_listxattrat
    466 common    removexattrat        sys_removexattrat
    # Due to a historical design error, certain syscalls are numbered
    # differently in x32 as compared to native x86_64. These syscalls have
    # numbers 512-547. Do not add new syscalls to this range. Numbers 548 and
    # above are available for non-x32 use.
    512 x32       rt_sigaction        compat_sys_rt_sigaction
    513 x32       rt_sigreturn        compat_sys_x32_rt_sigreturn
    514 x32       ioctl            compat_sys_ioctl
    515 x32       readv            sys_readv
    516 x32       writev            sys_writev
    517 x32       recvfrom        compat_sys_recvfrom
    518 x32       sendmsg            compat_sys_sendmsg
    519 x32       recvmsg            compat_sys_recvmsg
    520 x32       execve            compat_sys_execve
    521 x32       ptrace            compat_sys_ptrace
    522 x32       rt_sigpending        compat_sys_rt_sigpending
    523 x32       rt_sigtimedwait        compat_sys_rt_sigtimedwait_time64
    524 x32       rt_sigqueueinfo        compat_sys_rt_sigqueueinfo
    525 x32       sigaltstack        compat_sys_sigaltstack
    526 x32       timer_create        compat_sys_timer_create
    527 x32       mq_notify        compat_sys_mq_notify
    528 x32       kexec_load        compat_sys_kexec_load
    529 x32       waitid            compat_sys_waitid
    530 x32       set_robust_list        compat_sys_set_robust_list
    531 x32       get_robust_list        compat_sys_get_robust_list
    532 x32       vmsplice        sys_vmsplice
    533 x32       move_pages        sys_move_pages
    534 x32       preadv            compat_sys_preadv64
    535 x32       pwritev            compat_sys_pwritev64
    536 x32       rt_tgsigqueueinfo    compat_sys_rt_tgsigqueueinfo
    537 x32       recvmmsg        compat_sys_recvmmsg_time64
    538 x32       sendmmsg        compat_sys_sendmmsg
    539 x32       process_vm_readv    sys_process_vm_readv
    540 x32       process_vm_writev    sys_process_vm_writev
    541 x32       setsockopt        sys_setsockopt
    542 x32       getsockopt        sys_getsockopt
    543 x32       io_setup        compat_sys_io_setup
    544 x32       io_submit        compat_sys_io_submit
    545 x32       execveat        compat_sys_execveat
    546 x32       preadv2            compat_sys_preadv64v2
    547 x32       pwritev2        compat_sys_pwritev64v2
    # This is the end of the legacy x32 range.  Numbers 548 and above are
    # not special and are not to be used for x32-specific syscalls.
