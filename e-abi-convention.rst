二进制接口标准
==============

* `X86标准约定`_
* `X64标准约定`_
* `寄存器参数缺点`_
* `优化栈帧指针`_

X86标准约定
-----------

Intel386 System V ABI（应用程序二进制接口），描述了基于 Intel386 架构兼容的处理器平
台的 Linux IA-32 ABI 标准。自奔腾处理器之后推出的英特尔处理器（Pentium 4、Intel Core
系列及后续产品）引入了新的架构特性，特别是新的寄存器以及对这些寄存器进行操作的相应指令，
例如 MMX、英特尔 SSE（1-4）和英特尔 AVX 指令集扩展。C/C++ 编程语言也在不断发展，允许
程序员使用新的数据类型（例如 __m64、__m128 和 __m256）。许多编译器（包括英特尔编译器和
GCC）支持这些数据类型已有一段时间。自原始 ABI 文档编写以来，工具中的其他特性（例如十进
制浮点类型、64 位整数、异常处理等）也得到了开发。本文档描述了为实现各种工具之间的互操作
性，在实现这些新特性时所遵循的约定和限制。

函数调用约定仅用于全局函数，本地函数不会被其他编译单元使用可以使用任意约定，但还是推荐
所有函数尽可能都使用标准调用约定。

**类型与对齐** ::

    整数
    size_t定义为unsigned int
    enum大小最小4字节4字节对齐，默认类型为unsigned int
    int大小4字节4字节对齐
    指针大小4字节4字节对齐，指针为无符号类型
    long大小4字节4字节对齐
    long long大小8字节4字节对齐***

    浮点
    float大小4字节4字节对齐                             IEEE-754单精度
    double大小8字节4字节对齐***                         IEEE-754双精度
    long double大小8字节或10字节占12字节大小4字节对齐*** IEEE-754 80位扩展
    __float128大小16字节16字节对齐                      IEEE-754 128位扩展

    复数（Complex float point）
    _Complex float大小8字节4字节对齐                    IEEE-754复数单精度
    _Complex double大小16字节4字节对齐                  IEEE-754复数双精度
    _Complex long double大小16字节或24字节4字节对齐     IEEE-754复数80位扩展精度
    _Complex __float128大小32字节16字节对齐             IEEE-754复数128位扩展精度

    十进制浮点（Decimal floating point）
    _Decimal32大小4字节4字节对齐                        IEEE-754R 32位BID
    _Decimal64大小8字节8字节对齐                        IEEE-754R 64位BID
    _Decimal128大小16字节16字节对齐                     IEEE-754R 128位BID

    打包类型（Packed）
    __m64大小8字节8字节对齐                             MMX/3DNow!
    __m128大小16字节16字节对齐                          SSE/SEE2
    __m256大小32字节32字节对齐                          AVX
    __m512大小64字节64字节对齐                          AVX-512

    char/short/int/long位域与对应的无符号整型位域一样都是无符号数，只有明确指定了
    signed char/short/ing/long 才是有符号数。

仅__float128、_Complex __float128、_Decimal128、__m128、__m256、__m512要求强制对
齐，其他不对齐仅影响内存读取速度。

**寄存器保护**

由被调函数保护的可跨越函数调用边界的寄存器：%ebx %ebp %esi %edi %esp。状态寄存器
%eflags 的 df 位在函数入口处和返回时必须为 0（即方向向前），即由被调函数保护可以跨越函
数。状态寄存器的其他标记在标准调用约定中没有指定，没有跨函数保护。

CPU 在进入函数之前必须是 x87 模式。因此每个用了 MMX 寄存器的函数，必须在使用完 MMX 寄
存器之后，并在函数返回或调用另一个函数之前必须回到x87模式。所有x87寄存器都不能跨函数，
所有的 xmmN ymmN zmmN 都不能跨函数。但浮点控制寄存器%fcw由被调函数保护，被调函数如果修
改了它的值在返回之前必须进行恢复。媒体控制和状态寄存器mxcsr的控制位，也由被调函数保护可
以跨函数。

**返回值传递**

指针和大多数基本类型通过寄存器返回，结构体和联合体以及一些基本类型通过内存返回： ::

    寄存器%eax返回指针值、内存返回数据的地址、或以下基本类型：
    - 单字节整型：%al有意义高24位未定义，_Bool类型比特0包含真值比特1~7必须为零
    - 双字节整型：%ax有意义高16位未定义
    - 四字节整型：包括指针、_Decimal32通过%eax返回
    - 八字节类型：包括long long、_Decimal64、__Complex float高32位位于%edx
    - 数据的地址：结构体、联合体、十六字节及以上基本类型_Decimal128、__float128、
      __Complex double、__Complex long double通过内存返回，其地址保存到%eax中

    浮点类型float、double、long double通过寄存器%st(0)返回，调用者需要负责从寄存器栈
    中弹出该值，浮点寄存器中的单双扩展精度值的表示都是相同的，如果函数不返回一个浮点值
    这个寄存器退出函数前必须为空，在进入一个函数之前该寄存器也必须为空。类型__m64、
    __m128、__m256、__m512通过寄存器%mm0、%xmm0、%ymm0、%zmm0返回。

如果返回值通过内存返回，这个内存地址由调用者决定，并当作函数的第一个参数传递给被调函数，
该地址需要满足对应数据类型的对齐要求。被调函数将返回值写到给定的地址，并在返回前负责将地
址从栈中弹出并保存到%eax中。结构体和联合体返回值其大小是固定的，当前ABI没有指定怎样处理
变长数据对象的返回。让调用者提供返回值地址是为了让函数可以重入。

函数返回后，栈中的函数返回地址被ret指令弹出，栈中当作第一个参数传递的返回值内存地址也已
经被被调函数弹出，栈中仅剩下传递给被调函数的所有真实参数，这是调用者的责任将这些参数从栈
中弹出释放放。被调函数为了弹出栈中的返回值内存地址，一般在函数开头执行以下指令： ::

        ...                 # 函数参数
        arg2                # 第二个参数
        arg1                # 第一个参数
        retval address      # 内存返回值地址，该地址至少对齐到16字节边界
        return address      # 函数调用者压入的继续执行地址
    prologue:
        popl %eax           # 将函数返回地址出栈保存到%eax
        xchgl %eax,0(%esp)  # 返回值地址保存到%eax，%eax函数返回地址保存到栈顶
        pushl %ebp          # 保护栈基指针
        movl %esp,%ebp      # 设置当前的栈基指针
        subl $80, %esp      # 分配80字节栈空间
        pushl %edi          # 保护寄存器
        pushl %esi          # 保护寄存器
        pushl %ebx          # 保护寄存器
        movl %eax,-4(%ebp)  # 将返回值地址保存到第一个局部变量中
    epilogue:
        movl -4(%ebp),%eax  # 将返回值地址恢复到%eax寄存器
        popl %ebx           # 恢复寄存器
        popl %esi           # 恢复寄存器
        popl %edi           # 恢复寄存器
        leave               # 恢复栈基指针
        ret         # 弹出返回地址，跳到返回地址继续执行（即call的下一条指令）

因为返回值会改变函数调用约定，因此有返回值的函数必须正确进行声明。

**寄存器参数**

只有以下函数参数通过寄存器传递，前3个__m64类型的参数通过%mm0~%mm2传递，前3个__m128类
型的参数通过%xmm0~2传递，因为SSE、AVX、AVX-512寄存器的低位是共享的，第一个__m128类型
的参数回赋给%xmm0，后面如果还存在第一个__m256或__m512类型的参数会赋给%ymm1或%zmm1，而
不是%ymm0或%zmm0。

当调用接受可变参数的函数时，所有参数都通过栈传递，包括__m64、__m128、__m256类型，因为
可变参数函数可能会改变函数调用约定，因此这些函数必须正确声明。

**栈内存参数**

除了以上传递寄存器的情况外，所有其他参数都通过栈传递，包括所有整数类型、指针类型、浮点类
型、超出个数的__m64/__m128/__m256/__m512类型、以及所有结构体和联合体。为了满足类型的
对齐要求可能需要对参数进行填补。特殊的，__m64 和 _Decimal64 类型的参数只需要以4字节对
齐放在栈中。

函数调用进入函数时输入参数区的结束位置必须对齐到16字节边界，此时%esp指向返回地址，因此
（%esp+4）必须是16的整数倍。如果栈参数中包含__m256或__m512类型，则传入的参数必须对齐到
32字节或64字节。或者说在执行call指令之前，栈需要对齐到16字节（或32或64字节）边界。

参数不管是通过寄存器还是内存传递，小于字长（4字节）的参数会被零扩展或符号扩展到4字节，
且每个参数的长度会被调整到字长的整数倍。像在内存传递中的每个结构体和联合体参数，其大小必
须至少扩展到字长的整数倍。

当函数的返回值需要通过内存传递时，函数的第一个参数相当于是这个返回值内存地址，所有的内存
参数都要基于这个参数的起始地址为基准对齐。

函数如果要使用被保护的寄存器，必须先保护这些寄存器的值。函数在分配局部变量时，必须按照变
量类型的对齐要求严格对齐。

以下是X86栈帧布局： ::

             栈内容                位置
             memory argument n   | 4n+8(%ebp)
             memory argument 1   |  4+8(%ebp)
             memory argument 0   |    8(%ebp) <-- 该地址至少对齐到16字节边界
             return address      |    4(%ebp)
    %ebp --> previous %ebp value |    0(%ebp)
             unspecified         |   -4(%ebp) local variable 1
             ...                 |   -8(%ebp) local variable 2
             ...                 |       ...  ...
             variable size       |    0(%esp)

%esp寄存器总是指向当前栈帧的尾部，在不需要处理异常、栈展开、变长局部变量的函数中，可以
优化掉栈帧指针%ebp的使用（FPO，Frame Pointer Omission），省略使用%ebp可以减少指令或
者可以作其他额外用途。

一个函数调用示例： ::

    typedef struct {
        int a, b; double d;
    } param;
    param s;
    int i;
    __m128 v, x, y;
    __m256 w, z;
    extern param func(int i, __m128 v, param s, __m256 w, __m128 x y, __m256 z);
    func(i, v, s, w, x, y, z);

    参数分配    参数传递
                函数返回地址
    返回值地址   内存，位于 (%esp)   传入参数保持32为对齐
    i           内存，位于 4(%esp)  int大小4字节，下一个参数起始8(%esp)
    v           %xmm0寄存器传递
    s           内存，位于 8(%esp)  拷贝param大小16字节，下一个参数起始24(%esp)
    w           %ymm1寄存器传递
    x           %xmm2寄存器传递
    y           内存，位于 32(%esp)，拷贝y，__m128 需要对齐到16字节边界
    z           内存，位于 64(%esp)，拷贝z，__m256 需要对齐到32字节边界

    栈帧布局
    内容            长度
    z               32个字节
    padding         16个字节
    y               16个字节
    padding         8个字节
    s               16个字节
    i               4个字节
    返回值地址       4个字节 <-- %esp (对齐到32字节边界)

**cdecl调用约定**

综上所述，X86规定的函数调用约定，其实就是C语言的函数调用约定（cdecl）。但与其他编译器不
同的是，这里定义的 GNU 32位编译器要求被调函数负责清理返回值内存地址，其他内容才由调用者
清理。具体地，被调函数需要负责弹出返回值内存地址，并且当函数返回前需要将返回值内存地址保
存到 %eax，最后被调函数的 ret 指令会自动弹出函数的返回地址并返回到调用函数中，然后调用
函数清除栈上其他参数。而标准的 C 语言调用约定，或所有其他32位编译器的 cdecl 调用约定都
是调用函数清除所有的参数。

标准 cdecl 调用约定以及所有64位编译器的调用约定（包括Microsoft、Intel、GNU编译器）都
是由调用者负责清除所有的栈内容。如果清理栈是调用者的责任，并且速度很重要，那么调用者在调
用函数后可以保持栈指针在原位，并通过 mov 指令而不是 push 指令将后续函数调用的参数放在
栈上，可能会更有优势。

调用者负责清理栈内容的好处是，可以处理可变数量个数参数，因为清理是自己处理的想传多少参数
都行，都不会引起函数调用崩溃。对于参数已知并且数量和大小都是固定的函数，也可以使用
stdcall 调用约定，该约定调用者必须提供明确个数和大小的参数，然后由被调函数自动清理所有
的栈内容，这种调用约定可以生成更小的代码。例如微软Windows 32位系统上默认都使用该约定。

调用约定对比： ::

    平台            调用约定        传参顺序        栈清理      备注
    x86 32-bit      cdecl           C（从右至左）   caller
                    gnu cdecl       C              hybrid   栈对齐到16字节
                    stdcall         C              callee
                    gnu stdcall     C              callee   栈对齐到16字节

**fastcall调用约定**

X86 32位平台可以切换到使用 fastcall 调用约定，该约定也不能处理可变数量参数，因为是由被
调函数自动清理栈内容。快速调用约定可以使用 ecx edx 两个寄存器来传递参数，但对于返回值的
处理，微软32位编译器（msc）和 GNU 32位编译器（gnu）的处理不同。虽然它们都使用两个寄存
器 eax edx 传递返回值，但是当返回值的大小超过8个字节需要内存传递时，msc 将返回值的地址
当作第一个栈参数参数，而 gnu 则将返回值的地址当作第一个寄存器参数传递。

调用约定对比： ::

    调用约定                   x86 32-bit C function fastcall
    系统和编译器                Microsoft            Gnu Linux
    传参寄存器                  ecx edx              ecx edx
    栈传参顺序                  C                    C
    栈内容清理                  callee               callee
    传递返回值                  eax edx              eax edx
    返回值地址存储              stack                ecx
    栈对齐                      4-byte              16-byte
    具有返回值地址时的传参      ecx     arg1          ecx    addr***
                              edx     arg2          edx    arg1
                              (esp+0) addr***      (esp+0) arg2
                              (esp+4) arg3         (esp+4) arg3

X64标准约定
-----------

使用AMD64指令集的二进制程序可以编程为32位模型，int、long、指针是32位大小（ILP32），或
64位模型，int是32位大小、long和指针是64位大小（LP64）。这里讨论的覆盖了LP64和ILP32两
种编程模型。除了特别说明之外，AMD64架构ABI遵循Intel386 ABI描述的约定。

函数调用约定仅用于全局函数，本地函数不会被其他编译单元使用可以使用任意约定，但还是推荐
所有函数尽可能都使用标准调用约定。

**类型与对齐** ::

    整数
    _Bool大小1字节1字节对齐，在C++中对应为bool
    size_t定义为unsigned long（LP64），或unsigned int（ILP32）
    enum大小最小4字节4字节对齐，默认类型为unsigned int，可能扩充至long/unsigned long
    int大小4字节4字节对齐，
    指针大小8字节8字节对齐（LP64），或大小4字节4字节对齐（ILP32），指针为无符号类型
    long大小8字节8字节对齐（LP64），或大小4字节4字节对齐（ILP32）
    long long大小8字节8字节对齐***
    __int128大小16字节16字节对齐

    浮点
    _Float16大小2字节2字节对齐                          IEEE-754 16位
    float大小4字节4字节对齐                             IEEE-754单精度
    double大小8字节8字节对齐***                         IEEE-754双精度
    long double大小16字节16字节对齐，仅前10字节有效***   IEEE-754 80位扩展
    __float128大小16字节16字节对齐                      IEEE-754 128位扩展

    复数（Complex float point）
    _Complex float大小8字节4字节对齐                    IEEE-754复数单精度
    _Complex double大小16字节4字节对齐                  IEEE-754复数双精度
    _Complex long double大小16字节或24字节4字节对齐     IEEE-754复数80位扩展精度
    _Complex __float128大小32字节16字节对齐             IEEE-754复数128位扩展精度

    十进制浮点（Decimal floating point）
    _Decimal32大小4字节4字节对齐                        IEEE-754R 32位BID
    _Decimal64大小8字节8字节对齐                        IEEE-754R 64位BID
    _Decimal128大小16字节16字节对齐                     IEEE-754R 128位BID

    打包类型（Packed）
    __m64大小8字节8字节对齐                             MMX/3DNow!
    __m128大小16字节16字节对齐                          SSE/SEE2
    __m256大小32字节32字节对齐                          AVX
    __m512大小64字节64字节对齐                          AVX-512

    char/short/int/long位域与对应的无符号整型位域一样都是无符号数，只有明确指定了
    signed char/short/ing/long 才是有符号数。

仅__m128、__m256、__m512要求强制对齐，其他不对齐仅影响内存读取速度。数组类型根据它的元
素类型对齐，但如果大小至少是16字节或者alloca分配的变长数组需要至少对齐到16字节边界。

**寄存器保护**

由被调函数保护的可跨越函数调用边界的寄存器：%rbx %rbp %rsp %r12 %r13 %r14 %r15。在
Intel386 ABI中，%rdi %rsi 是需要保护的寄存器。状态寄存器%rflags 的 df 位在函数入口处
和返回时必须为 0（即方向向前），即由被调函数保护可以跨越函数。状态寄存器的其他标记在标
准调用约定中没有指定，没有跨函数保护。

CPU 在进入函数之前必须是 x87 模式。因此每个用了 MMX 寄存器的函数，必须在使用完 MMX 寄
存器之后，并在函数返回或调用另一个函数之前必须回到x87模式。所有x87寄存器都不能跨函数，
所有的 xmmN ymmN zmmN 都不能跨函数。但浮点控制寄存器%fcw由被调函数保护，被调函数如果修
改了它的值在返回之前必须进行恢复。媒体控制和状态寄存器mxcsr的控制位，也由被调函数保护可
以跨函数。线程指针（thread pointer）%fs寄存器也需要保护。

X87浮点控制字（Control Word）和状态字（Status Word）寄存器： ::

    Control Word (16-bit) - FSTCW/FNSTCW FLDCW
    [00] IM - Invalid Operation
    [01] DM - Denormal Operand
    [02] ZM - Zero Divide
    [03] OM - Overflow
    [04] UM - Underflow
    [05] PM - Precision
    [06] RSV- Reserved
    [07] RSV- Reserved
    [08] PC - Precision Control bit0
    [09] PC - Precision Control bit1
    [10] RC - Rounding Control bit0
    [11] RC - Rounding Control bit1
    [12] X  - Infinity Control
    [13~15] - Reserved

    fstcw：在执行保存操作之前，fstcw 会检查 FPU 的状态寄存器，查看是否有未处理的异常。
    如果存在未处理的异常，它会触发相应的异常处理程序，程序的执行流程会被打断，跳转到异
    常处理代码处执行。只有当 FPU 没有未处理的异常时，才会将控制字保存到指定内存。

    fstcw 相当于 fwait + fnstcw。wait/fwait 指令是同步指令（实际上它们是同一操作码的
    不同助记符）。这些指令会检查 x87 浮点运算单元（FPU）的状态字，以查看是否存在未屏蔽
    的待处理 x87 FPU 异常。如果发现任何未屏蔽的待处理 x87 FPU 异常，处理器会先处理这
    些异常，然后再继续执行指令流中的指令（整数指令、浮点指令或系统指令）。提供
    wait/fwait 指令是为了实现 x87 FPU 和处理器整数单元之间的指令执行同步。

    当一个浮点异常未被屏蔽且异常条件发生时，x87 浮点运算单元（FPU）会停止进一步执行浮
    点指令，并发出异常事件信号。当指令流中接下来出现浮点指令或 WAIT/FWAIT 指令时，处理
    器会检查 x87 FPU 状态字中的 ES（Error Summary，错误汇总）标志，以查看是否有待处理
    的浮点异常。如果存在待处理的浮点异常，x87 FPU 会隐式调用（陷入）浮点软件异常处理程
    序。然后，异常处理程序可以针对选定的或所有浮点异常执行恢复程序。

    fnstcw：无论 FPU 中是否存在未处理的异常，fnstcw 都会直接将控制字保存到指定内存，
    不会触发异常检查和处理机制。这意味着即使 FPU 处于异常状态，该指令也能正常执行保存操
    作，不会因为异常而中断。

    Status Word (16-bit) - FSTSW/FNSTSW/FXSAVE FLDSW/FXRSTOR
    [00] IE - Invalid Operation
    [01] DE - Denormalized Operand
    [02] ZE - Zero Divide
    [03] OE - Overflow
    [04] UE - Underflow
    [05] PE - Precision
    [06] SF - Stack Fault Exception Flags
    [07] ES - Exception Summary Status
    [08] C0 - Condition Code 0
    [09] C1 - Condition Code 1
    [10] C2 - Condition Code 2
    [11] TOP- Top of Stack Pointer bit0
    [12] TOP- Top of Stack Pointer bit1
    [13] TOP- Top of Stack Pointer bit2
    [14] C3 - Condition Code 3
    [15] B  - FPU Busy

媒体控制和状态寄存器 MXCSR： ::

    The floating point control word and bit 6-15 of the MXCSR register must be
    saved and restored before any call or return by any procedure that needs
    to modify them, except for procedures that have the purpose of changing
    these.

    32-bit MXCSR Control/Status Register - STMXCSR LDMXCSR
    [00] IE - Invalid Operation FLag
    [01] DE - Denormal Flag
    [02] ZE - Divide-by-Zero Flag
    [03] OE - Overflow Flag
    [04] UE - Underflow Flag
    [05] PE - Precision Flag
    [06] DAZ- Denormals Are Zeros, enable the denormals-are-zeros mode
    [07] IM - Invalid Operation Mask, set 1 to mask IE let dont gen IE exception
    [08] DM - Denormal Operation Mask, set 1 to mask out DE
    [09] ZM - Divide-by-Zero Mask, set 1 to mask out ZE
    [10] OM - Overflow Mask, set 1 to mask out OE
    [11] UM - Underflow Mask, set 1 to mask out UE
    [12] PM - Precision Mask, set 1 to mask out PE
    [13] RC - Rounding Control bit0
    [14] RC - Rounding Control bit1
    [15] FTZ- Flush to Zero, enable flush to zero mode
    [16~31] - Reserved

    If a (V)LDMXCSR instruction clears a SIMD floating-point exception mask
    bit and sets the corresponding exception flag bit, a SIMD floating-point
    exception will not be immediately generated. The exception will be
    generated only upon the execution of the next instruction that meets
    both conditions below: • the instruction must operate on an XMM or YMM
    register operand, • the instruction causes that particular SIMD
    floating-point exception to be reported. This instruction’s operation is
    the same in non-64-bit modes and 64-bit mode.

用来传递参数的临时寄存器： ::

    整型寄存器
    %rdi    arg1
    %rsi    arg2
    %rdx    arg3
    %rcx    arg4
    %r8     arg5
    %r9     arg6

    向量寄存器
    %xmm0 ~ %xmm7
    %ymm0 ~ %ymm7
    %zmm0 ~ %zmm7

特殊用途的寄存器： ::

    %rax 除临时寄存器、返回函数值外，%al还传递变参中使用向量寄存器传递的参数个数
    %r10 临时寄存器，还用于传递函数的静态链指针（a function's static chain pointer)
    %r11 临时寄存器，让 PLT 计算转移地址时有足够的寄存器可用，不至于寄存器要溢出到内存
    %r15 被保护的寄存器，还可选用于GOT基指针（GOT base pointer）

    %rax 和 %rdx 传递整型返回值
    %xmm0 和 %xmm1，或 %ymm0 和 %ymm1，或 %zmm0 和 %zmm1，用于传递向量返回值
    %st0 传递 long double 返回值
    %st0 和 %st1 传递 complex long double 返回值

**返回值传递**

返回值的传递按照以下方法判断：

1. 使用下面的分类算法对返回类型进行分类。

2. 如果类别是 MEMORY，则调用者为返回值提供空间，并将此存储地址作为函数的第一个参数传递
   给 %rdi。实际上，这个地址变成了一个“隐藏”的第一个参数。被调函数返回时，%rax 将包含
   调用者在 %rdi 中传递的地址。

   这块用于存储返回值的内存不能和被调函数可以访问的其他数据的内存区域有重叠，如其他显式
   传递的参数、函数内部的局部变量等，否则会导致数据混乱和错误。此存储不应与通过其他名
   称对被调函数可见的任何数据重叠。

3. 如果类别是 INTEGER，则使用 %rax、%rdx 中下一个可用的寄存器返回。

4. 如果类别是 SSE，则使用 %xmm0、%xmm1 中的下一个可用的向量寄存器返回。

5. 如果类别是 SSEUP，则使用最后使用的向量寄存器的下一个 8 字节块。

6. 如果类别是 X87，则值以 80 位 x87 数字的形式在 %st0 上返回。

7. 如果类别是 X87UP，与前一个 X87 值一起在 %st0 中返回。

8. 如果类别是 COMPLEX_X87，则值的实部在 %st0 中返回，虚部在 %st1 中返回。

与 MSC 64位编译器不同的是，GNU 可以使用 %rax 和 %rdx 返回16字节返回值，而 MSC 只能返
回8字节返回值，只能使用 %rax 一个寄存器作为返回值使用。另外由于传递参数的寄存器不同，如
果返回值需要通过内存返回，返回值的内存地址在 MSC 上使用 %rcx 传递，而 GNU 使用 %rdi
传递。

**函数参数类别**

在计算参数值之后，它们会被放入寄存器或推入栈中。参数的传递方式在以下部分中描述。我们首先
定义对参数进行分类的几个类别，这些类别也对应于 AMD64 寄存器的种类：

INTEGER ::

    可以放入通用寄存器的整数类型。参数类型_Bool、char、short、int、long、long long、
    指针类型属于该类。参数类型 __int128，被当成结构体 typedef struct { long l, h; }
    __int128; 也属于INTEGER，只不过它需要使用两个寄存器，而如果作为内存参数传递必须
    对齐到16字节变量。小于等于64比特的_BitInt(N)类型也属于INTEGER。而大于64比特的
    _BitInt(N)类型相当于是一个64比特整数字段的结构体。只要结构体大小不超过64字节，或
    512比特，就可能使用寄存器传递。

    当 _Bool 类型的值在寄存器或栈中返回或传递时，比特0包含真值比特1到7应为零。布尔值在
    存储到内存对象中时，以单字节对象的形式存储，其值始终为 0（假）或 1（真）。当存储在
    整数寄存器中，寄存器的所有 8 个字节都是有意义的；任何非零值都被视为真。

SSE ::

    可以放入向量寄存器的参数类型。参数类型_Float16、float、double、_Decimal32、
    _Decimal64、__m64属于该类。参数类型__float128、_Decimal128、__m128分为两部分，
    低字节部分属于SSE，高字节部分属于SSEUP。参数类型__m256、__m512被分成四部分或八部
    分，低位部分属于SSE，高位部分属于SSEUP。复数complex T，其中T可以是_Float16、
    float、double、__float128，被当成结构体 struct complexT { T real, imag; };
    处理，由于其中的两个字段都属于SSE类别，因此complexT也属于SSE类别。

SSEUP ::

    可以放入向量寄存器并且可以使用其高字节进行传递和返回的参数类型。参数类型
    __float128、_Decimal128、__m128分为两部分，低字节部分属于SSE，高字节部分属于
    SSEUP。参数类型__m256、__m512被分成四部分或八部分，低位部分属于SSE，高位部分属于
    SSEUP。

X87, X87UP ::

    可以通过x87浮点协处理返回的参数类型。参数类型long double（只有10字节有效，高位有6
    个字节的填补），其低8字节的尾数部分属于X87，高位的2字节指数部分和6字节填补部分属于
    X87UP。类别X87、X87UP、COMPLEX_X87的参数都通过内存传递，而返回值通过st(0)以及
    st(1)返回。

COMPLEX_X87 ::

    可以通过x87浮点协处理返回的参数类型。参数类型 complex long double 归类为
    COMPLEX_X87。类别X87、X87UP、COMPLEX_X87的参数都通过内存传递，而返回值通过st(0)
    以及st(1)返回。

NO_CLASS ::

    用于对聚合类型参数的对齐填补，它是聚合类型中每8个字节划分的初始类别。零宽度的位域属
    于 NO_CLASS。

MEMORY ::

    通过栈内存传递和返回的那些参数类型。

对于聚合类型，包括结构体、数组、以及联合体，其分类如下：

1. 如果对象的大小大于64字节（512比特位），或者它包含未对齐的字段，则它属于 MEMORY 类。

2. 根据C++ ABI所指定的，如果C++对象对于调用来说是非平凡的（non-trivial，即有重大意义
   的），它通过不可见的引用传递（对象在参数列表中被替换为一个属于INTEGER类别的指针）。
   对于调用目的来说类型是非平凡的对象不能按值传递，因为这样的对象在调用者和被调用者中必
   须具有相同的地址。在从函数返回对象时，也存在类似的问题。参见C++17 class.temporary
   部分。

3. 如果聚合类型的大小超过8字节，每个8字节分别分类。每个8字节被初始分类为 NO_CLASS。

4. 对象的每个字段递归分类，以便总是考虑两个字段。根据8字节中的每个字段的类别计算结果的
   类别： ::

    (a) 如果两个字段的类别相同，则结果类为该类。
    (b) 如果其中一个类是 NO_CLASS，结果类为另一个类。
    (c) 如果其中一个类是 MEMORY，结果为 MEMORY 类。
    (d) 如果其中一个类是 INTEGER，结果为 INTEGER 类。
    (e) 如果其中一个类是 X87、X87UP 或 COMPLEX_X87 类，则使用 MEMORY 类。
    (f) 否则使用 SSE 类。

5. 然后进行后合清理后处理： ::

    (a) 如果其中一个类是 MEMORY，整个参数通过内存传递。
    (b) 如果 X87UP 的前面不是 X87，则整个参数通过内存传递。
    (c) 如果聚合类型的大小超过两个8字节，并且第一个8字节不是SSE类别或有其他8字节不是
        SSEUP，则整个参数通过内存传递。
    (d) 如果 SSEUP 的前面不是 SSE 或 SSEUP，则将其转换为 SSE。

合并清理的后处理，它确保对于不支持 __m256 类型的处理器，如果对象的大小大于两个8字节，并
且第一个8字节不是SSE或有其他8字节不是SSEUP，它仍然属于MEMORY类。这反过来又确保了对于支
持__m256类型的处理器，如果对象的大小是四个8字节，并且第一个8字节是 SSE，其他所有8字节
都是 SSEUP，它可以被放入寄存器。这同样适用于 __m512 类型。也就是说，对于支持 __m512
类型的处理器，如果对象的大小是八个 8 字节，并且第一个 8 字节是 SSE，其他所有 8 字节都
是 SSEUP，它可以被放入寄存器，否则它将通过内存传递。

**寄存器参数**

一旦参数被分类，按从左到右的参数顺序分配寄存器，用于传递参数：

1. 如果类别是 MEMORY，则在尊重参数对齐的地址上通过栈内存传递参数。

2. 如果类别是 INTEGER，则按顺序使用寄存器序列%rdi、%rsi、%rdx、%rcx、%r8和%r9。%r11
   不需要被保护也不用于传递参数，此寄存器当作临时寄存器使用，可以让PLT中的代码在计算控
   制权转移的目标地址时，不需要溢出任何寄存器（spill any registers）到内存中。

   在计算需要转移控制的地址时，CPU 需要使用寄存器来进行一些中间计算。当程序需要使用更多
   的临时数据存储空间，但寄存器已经不够用时，就需要把寄存器中的数据保存到内存（通常是
   栈）中，这个过程就叫做 “寄存器溢出（spill registers）”。之后，当需要使用这些数据
   时，再从内存中把数据恢复到寄存器中。这里有了%r11这个额外的临时寄存器，PLT的计算就可
   以完全通过寄存器完成，不会发生寄存器溢出。

   寄存器 %r10 可用于传递函数的静态链指针（function's static chain pointer）。在某些
   系统或编译器的实现中，规定使用寄存器 %r10 来传递函数的静态链指针。当一个函数被调用
   时，调用者会将指向自己栈帧的指针（即静态链指针）放入 %r10 寄存器中，然后被调函数就可
   以从 %r10 寄存器中获取这个指针，进而通过这个指针访问到外部函数的栈帧和其中的变量。

   在可变参数函数中，寄存器%al还用于传递函数参数中，使用向量寄存器传递的参数个数。此时
   只有 %al 的内容是定义的，%rax 其余部分的内容未定义。

3. 如果类别是SSE，则使用下一个可用的向量寄存器，按顺序 %xmm0 到 %xmm7 使用。

4. 如果类别是SSEUP，则使用最后使用的向量寄存器的下一个 8 字节块。

5. 如果类别是 X87、X87UP 或 COMPLEX_X87，则通过内存传递。

当调用使用了 varargs 或 stdargs 参数的函数时，即调用无原型的函数或者调用声明中包含省略
号（...）的函数，%al 会作为一个隐藏的参数，以指定变参中可以使用向量寄存器进行传递的参
数个数。%al 的内容不需要与使用的寄存器数量完全匹配，但必须是使用向量寄存器数量的上限，
并且在 0 到 8（包括）范围内。

当将 __m256 或 __m512 参数传递给使用 varargs 或 stdarg 的函数时，必须提供函数原型。
否则，运行时行为是未定义的。

**栈内存参数**

总的来说，函数参数分类中，INTEGER 和 SSE 类型的参数首先尽可能的使用寄存器传递，其他类
别的参数以及剩下的 INTEGER 和 SSE 参数都通过内存传递。也即所有寄存器参数都分配完之后，
剩下的参数都是内存参数，按参数声明的相反顺序（从右到左）以及参数类型的对齐要求推入栈中。

如果参数的任何 8 字节没有可用的寄存器，则整个参数通过栈传递。如果已经为这样的参数的某些
8 字节分配了寄存器，则撤销这些分配。

在栈上采用从右到左的顺序，使得处理接受可变数量参数的函数更为简单。第一个参数的位置总是可
以基于该参数的类型静态计算。如果参数按从左到右的顺序推送，计算第一个参数的地址将很困难。

函数调用进入函数时输入参数区的结束位置必须对齐到16字节边界，此时%rsp指向返回地址，因此
（%rsp+8）必须是16的整数倍。如果栈参数中包含__m256或__m512类型，则传入的参数必须对齐到
32字节或64字节。或者说在执行call指令之前，栈需要对齐到16字节（或32或64字节）边界。

参数不管是通过寄存器还是内存传递，小于字长（8字节）的参数会被零扩展或符号扩展到8字节，
且每个参数的长度会被调整到字长的整数倍。像在内存传递中的每个结构体和联合体参数，其大小必
须至少扩展到字长的整数倍。必须注意的是，多个参数总是单独处理，例如不会将多个参数合并到
一个寄存器中进行传递，但如果一个参数是聚合类型包含多个成员，这些成员只要不超过对应寄存器
的大小且参数总大小不超过64字节（512比特位），多个成员可以合并在一起在一个寄存器中传递。

以下是X64栈帧布局： ::

             栈内容                位置
             memory argument n   | 8n+16(%rbp)
             memory argument 1   |  8+16(%rbp)
             memory argument 0   |    16(%rbp) <-- 该地址至少对齐到16字节边界
             return address      |     8(%rbp)
    %rbp --> previous %rbp value |     0(%rbp)
             unspecified         |    -8(%rbp) local variable 1
             ...                 |   -16(%rbp) local variable 2
             ...                 |        ...  ...
             variable size       |     0(%rsp)
             stack red zone      |    -8(%rsp)
             ...                 |        ...
             ...                 |  -128(%rsp)

%rsp寄存器总是指向当前栈帧的尾部，在不需要处理异常、栈展开、变长局部变量的函数中，可以
优化掉栈帧指针%rbp的使用（FPO，Frame Pointer Omission），省略使用%rbp可以减少指令或
者可以作其他额外用途。

%rsp指向的位置之后的128字节区域被认为是保留的，不应被信号或中断处理程序修改。因此，函数
可以使用这个区域来存储不需要跨越函数调用边界的临时数据，因为一旦继续调用函数这一区域就可
能被函数破坏。特别是，叶子函数可以使用这个区域作为它们的整个栈帧，而不是在函数序言和尾声
代码中调整栈指针。这个区域被称为栈红区。保留128个字节大小是因为128可以用一个字节表示地
址偏移。

Linux内核可能会将输入参数区域的结尾对齐到8字节，而不是16字节边界。它不遵循红区规则，因
此这个区域不允许被内核代码使用。内核代码应该使用GCC的-mno-red-zone选项进行编译。

一个参数传递示例： ::

    typedef struct { int a, b; double d; } param;
    param s;
    int e, f, g, h, i, j, k;
    long double ld;
    double m, n;
    __m256 y;
    __m512 z;
    extern void func(int e, f, param s, int g, h, long double ld, double m,
        __m256 y, __m512 z, double n, int i, int j, int k);
    func(e, f, s, g, h, ld, m, y, z, n, i, j, k);

    参数分配：

    通用寄存器          向量寄存器          函数调用前的栈帧偏移
    %rdi    e           %xmm0   s.d         0   ld
    %rsi    f           %xmm1   m           16  j   // 每个参数大小至少是8字节
    %rdx    s.a s.b     %ymm2   y           24  k   // 每个参数大小至少是8字节
    %rcx    g           %zmm3   z
    %r8     h           %xmm4   n
    %r9     i

**可变参数列表**

可移植的 C 程序依赖于参数传递方案，隐式假设所有参数都通过栈传递，并且参数在栈上以递增顺
序出现。这些假设使得程序从未真正可移植，但它们在许多实现上能正常工作。然而在 AMD64 架构
上，它们无法工作，因为某些参数是通过寄存器传递的。可移植的C程序必须使用头文 <stdarg.h>
来处理可变参数列表。

当调用接受可变参数的函数时，%al 必须设置为变参中可以使用向量寄存器传递的参数个数。当
__m256 或 __m512 作为可变参数传递时，它们应始终通过栈传递。只有命名的 __m256 和
__m512 参数可以按寄存器传递，即那些命名的正常参数。下面是一个存在可变参数的情况下，函数
参数传递的例子： ::

    int a, b;
    long double ld;
    double m, n;
    __m256 u, y;
    __m512 v, z;
    extern void func (int a, double m, __m256 u, __m512 v, ...);
    func(a, m, u, v, b, ld, y, z, n); // 可变参数中最多有3个参数通过向量寄存器传递

    参数分配：

    通用寄存器              向量寄存器                  栈帧偏移
    %rdi    a               %xmm0   m                   0   ld  16-byte
    %rsi    b               %ymm1   u                  32   y   32-byte // 对齐
    %rax    3               %zmm2   v                  64   z   64-byte // 对齐
                            %xmm3   n

接受可变参数列表的函数的开始代码，一般都是使用一个可变参数列表调用宏函数 va_start，它会
将参数寄存器的值保持到对应的寄存器保存区域（register save area）。每个参数寄存器在寄存
器保存区域中都有一个固定的偏移量，如下图： ::

    寄存器保持区域
    寄存器      偏移
    %rdi        0
    %rsi        8
    %rdx        16
    %rcx        24
    %r8         32
    %r9         40
    %xmm0       48
    %xmm1       64
    ...
    %xmm7       160

只有可能用于传递参数的寄存器需要被保存。其他寄存器不被访问，可以用于其他目的。如果一个函
数被认为永远不会接受通过寄存器传递的参数，寄存器保存区域可以完全省略。这一事实可以通过探
索 va_arg 宏所使用的类型来确定，或者因为命名参数已经完全耗尽了参数寄存器。

函数开始代码应使用 %al 以避免保存不必要的 XMM 寄存器。这对于仅使用整数的程序尤其重要，
以防止初始化 XMM 单元。va_list 类型是一个数组，包含一个结构体元素，包含实现 va_arg 宏
所需的信息。va_list 类型的 C 定义如下： ::

    typedef struct {
        unsigned int gp_offset;        // 通用寄存器偏移
        unsigned int fp_offset;        // 浮点寄存器偏移
        void *overflow_arg_area;       // 溢出参数区域
        void *reg_save_area;           // 寄存器保存区域
    } va_list[1];

va_start 宏初始化结构如下：

1. reg_save_area：该元素指向寄存器保存区域的起始位置。

2. overflow_arg_area：该指针用于获取通过栈传递的参数。如果有参数，它会初始化为栈上第一
   个参数的地址，然后始终更新为指向栈上下一个参数的起始位置。

3. gp_offset：该元素保存从 reg_save_area 到下一个可用的通用参数寄存器保存位置的字节偏
   移。如果所有参数寄存器都已用尽，它的值被设置为 48。

4. fp_offset：该元素保存从 reg_save_area 到下一个可用的浮点参数寄存器保存位置的字节偏
   移。如果所有参数寄存器都已用尽，它的值被设置为 176（48 + 8*16）。

va_arg(vl, type) 的通用实现算法定义如下：

1. 确定 type 是否可以通过寄存器传递。如果不能，转到第 7 步。

2. 计算num_gp保存传递type所需的通用寄存器数量，计算num_fp保存所需向量寄存器数量。

3. 验证参数是否适合寄存器，如果条件满足转到第 7 步： ::

    l->gp_offset > 48 - num_gp * 8 或者
    l->fp_offset > 176 - num_fp * 16

4. 从 l->reg_save_area 中以 l->gp_offset 和 l->fp_offset 的偏移量提取 type。这可能
   需要将参数复制到临时位置，以防参数在不同的寄存器类中传递，或对于通用寄存器参数需要有
   比8字节更大的对齐，对于XMM寄存器参数需要比16字节更大的对齐。

5. 设置： ::

    l->gp_offset = l->gp_offset + num_gp * 8
    l->fp_offset = l->fp_offset + num_fp * 16

6. 返回提取的类型。

7. 如果type所需的对齐超过8字节边界，则将l->overflow_arg_area向上对齐到16字节边界。

8. 从 l->overflow_arg_area 中提取 type。

9. 将 l->overflow_arg_area 设置为：l->overflow_arg_area + sizeof(type)

10. 将 l->overflow_arg_area 向上对齐到 8 字节边界。

11. 返回提取的类型。

va_arg 通常实现为编译器内置函数，并为每种特定类型展开为简化形式。以下是
va_arg(vl, int) 的示例实现： ::

        movl l->gp_offset, %eax
        cmpl $48, %eax                  // 寄存器可用吗？
        jae stack                       // 如果不可用，从栈中找参数
        leal $8(%rax), %edx             // 下一个可用寄存器
        addq l->reg_save_area, %rax     // 保存寄存器的地址
        movl %edx, l->gp_offset         // 更新 gp_offset
        jmp fetch
    stack:
        movq l->overflow_arg_area, %rax // 栈槽的地址
        leaq 8(%rax), %rdx              // 下一个可用栈槽
        movq %rdx, l->overflow_arg_area // 更新
    fetch:
        movl (%rax), %eax               // 加载参数

寄存器参数缺点
--------------

使用寄存器传递参数，可以获得一定的运行时效率。替代方案是将参数推入堆栈，执行速度会较慢。
对于X86，通常用于参数的寄存器包括 EAX、EBX、ECX 和 EDX，较少使用 EDI 和 ESI。不幸的
是，这些相同的寄存器也用于保存诸如循环计数器和计算中的操作数等数据值。因此，任何用作参数
的寄存器在过程调用之前如果已作为他用，必须先压入堆栈，分配过程参数的值，并在过程返回后恢
复它们的原始值。所有额外的压栈和出栈操作不仅会造成代码混乱，而且往往会抵消我们希望通过使
用寄存器参数获得的性能优势。

此外，程序员必须非常小心，确保每个寄存器的 PUSH 都与其适当的 POP 匹配，即使在代码中有
多个执行路径。例如，在以下代码中，如果第 8 行的 EAX 等于 1，则程序将不会在第 17 行返回
给其调用者，因为有三个寄存器值还留在栈上，ret 出栈的不是真正的返回地址。 ::

     1:     push    ebx                 ; save register values
     2:     push    ecx
     3:     push    esi
     4:     mov     esi,OFFSET array    ; starting OFFSET
     5:     mov     ecx,LENGTHOF array  ; size, in units
     6:     mov     ebx,TYPE array      ; doubleword format
     7:     call    dump                ; display memory
     8:     cmp     eax,1               ; error flag set?
     9:     je      error_exit          ; exit with flag set
    10: 
    11:     pop     esi                 ; restore register values
    12:     pop     ecx
    13:     pop     ebx
    14:     ret
    15: error_exit:
    16:     mov     edx,offset error_msg
    17:     ret

而使用栈传递参数，提供了不需要寄存器的一种灵活方法，在调用过程前只需简单的将参数压入栈
中，不需要考虑寄存器。

优化栈帧指针
------------

* http://www.nynaeve.net/?p=97
* http://www.nynaeve.net/?page_id=67

在调试程序的过程中，你可能遇到过“FPO”这个术语。FPO（Frame Pointer Omission）指的是一
类特定的编译器优化，在 x86 架构中，涉及编译器如何访问局部变量和基于栈的参数，对应于 GCC 
的 -fomit-frame-pointer 优化选项。对于使用局部变量或基于栈的参数的函数，编译器需要一
种机制来引用这些值在栈上的位置。通常，这可以通过两种方式实现：

1. 直接从栈指针（esp）访问局部变量。如果启用了 FPO 优化，则会采用这种行为。虽然这不需
   要使用单独的寄存器来跟踪局部变量和参数的位置，但它使生成的代码稍微复杂一些。特别是，
   由于函数调用或其他修改栈的指令，局部变量和参数相对于 esp 的偏移量在函数执行过程中实
   际上会发生变化。因此，编译器必须在函数中每次引用基于栈的值时跟踪当前 esp 值的实际偏
   移量。对于编译器来说，这通常不是一个大问题，但在手写汇编时，这可能会变得有些棘手。

2. 专用寄存器指向栈上相对于局部变量和基于栈的参数的固定位置，并使用该寄存器访问局部变量
   和参数。如果禁用 FPO 优化，则会采用这种行为。约定是使用 ebp 寄存器来访问局部变量和
   栈参数。ebp 通常设置为在 [ebp+08] 处找到函数的第一个栈参数，而局部变量通常位于 ebp
   的负偏移量处。

禁用 FPO 优化的典型函数序言代码如下所示： ::

    push   ebp               ; 保存旧的 ebp（非易失性）
    mov    ebp, esp          ; 将 ebp 加载为栈指针
    sub    esp, sizeoflocals ; 为局部变量保留空间
    ...                      ; 函数的其余部分

主要概念是，当禁用 FPO 优化时，函数会立即保存旧的 ebp，然后将 ebp 加载为当前的栈指针。
这设置了相对于 ebp 的栈布局，如下所示： ::

    [ebp-01]   第一个局部变量的一个字节
    [ebp+00]   旧的 ebp 值
    [ebp+04]   返回地址
    [ebp+08]   第一个参数...

此后，函数将始终使用 ebp 来访问局部变量和基于栈的参数。函数的序言代码可能会有所不同，特
别是使用变体 __SEH_prolog 来设置初始 SEH 框架的函数，但最终结果在相对于 ebp 的栈布局
方面始终相同。

如前所述，这使得 ebp 寄存器不再可用于寄存器分配器的其他用途。然而，相对于启用 FPO 优化
的函数，这种性能损失通常不足以引起较大关注。此外，有许多条件要求函数使用帧指针，你可能会
遇到：

1. 任何使用 SEH 的函数必须使用帧指针，因为当异常发生时，无法知道局部变量与 esp 值（栈
   指针）之间的偏移量。异常可能发生在任何地方，而诸如函数调用或为函数调用设置栈参数等操
   作会修改 esp 的值。

2. 任何使用具有析构函数的自动 C++ 对象的函数必须使用 SEH 以支持编译器的展开。这意味着
   大多数 C++ 函数最终会禁用 FPO 优化。可以更改编译器对 SEH 异常和 C++ 展开的假设，但
   默认（推荐）设置是在发生 SEH 异常时展开对象。

3. 任何使用 _alloca 在栈上动态分配内存的函数必须使用帧指针，因为局部变量和参数相对于
   esp 的偏移量在运行时可能会变化，并且在编译时生成代码时编译器无法知道。

这里进一步探讨，当需要调试程序时，启用（或禁用）FPO 优化会带来哪些影响。为了说明问题，
考虑以下示例程序，其中包含几个无实际功能的函数，这些函数在栈上传递参数并相互调用。为了便
于说明，禁用了全局优化和函数内联。 ::

    __declspec(noinline) void f3(int* c, char* b, int a) {
        *c = a * 3 + (int)strlen(b);
        __debugbreak();
    }
    __declspec(noinline) int f2(char* b, int a) {
        int c;
        f3(&c, b + 1, a - 3);
        return c;
    }
    __declspec(noinline) int f1(int a, char* b) {
        int c;
        c = f2(b, a + 10);
        c ^= (int)rand();
        return c + 2 * a;
    }
    int __cdecl wmain(int ac, wchar_t** av) {
        int c;
        c = f1((int)rand(), "test");
        printf("%d\n", c);
        return 0;
    }

如果运行程序并在硬编码的断点处中断到调试器中，并且加载了符号，一切看起来都符合预期： ::

    0:000> k
    ChildEBP RetAddr  
    0012ff3c 010015ef TestApp!f3+0x19
    0012ff4c 010015fe TestApp!f2+0x15
    0012ff54 0100161b TestApp!f1+0x9
    0012ff5c 01001896 TestApp!wmain+0xe
    0012ffa0 77573833 TestApp!__tmainCRTStartup+0x10f
    0012ffac 7740a9bd kernel32!BaseThreadInitThunk+0xe
    0012ffec 00000000 ntdll!_RtlUserThreadStart+0x23

无论是否启用 FPO 优化，只要加载了符号，我们都能得到合理的调用栈。然而，如果没有加载符
号，情况就大不相同了。对于启用了 FPO 优化且未加载符号的程序，请求调用栈会是一团糟： ::

    0:000> k
    ChildEBP RetAddr  
    WARNING: Stack unwind information not available.
    Following frames may be wrong.
    0012ff4c 010015fe TestApp+0x15d8
    0012ffa0 77573833 TestApp+0x15fe
    0012ffac 7740a9bd kernel32!BaseThreadInitThunk+0xe
    0012ffec 00000000 ntdll!_RtlUserThreadStart+0x23

比较两个调用栈，我们在输出中丢失了三个调用帧。我们能得到稍微合理的结果的唯一原因是
WinDbg 的调用栈跟踪机制有一些智能启发式方法，用于猜测使用帧指针的调用帧的位置。

回顾一下使用帧指针时调用栈的设置方式，在没有符号的情况下在 x86 上进行调用栈遍历的程序将
把栈视为一种调用帧的链表。回想一下我之前提到的使用帧指针时栈的布局： ::

    [ebp-01]   最后一个局部变量的最后一个字节
    [ebp+00]   旧的 ebp 值
    [ebp+04]   返回地址
    [ebp+08]   第一个参数...

这意味着，如果我们试图在没有符号的情况下进行调用栈遍历，方法是假设 ebp 指向一个类似于以
下的结构体： ::

    typedef struct _CALL_FRAME {
        struct _CALL_FRAME* Next; // 指向前一个栈帧
        void* ReturnAddress;
    } CALL_FRAME, * PCALL_FRAME;

请注意，这与我之前描述的相对于 ebp 的栈布局相对应。一个设计用于遍历使用了帧指针的帧非常
简单，如下所示。使用 _AddressOfReturnAddress 内在函数查找 ebp，假设旧的 ebp 位于返回
地址地址之前的 4 个字节处： ::

    LONG StackwalkExceptionHandler(PEXCEPTION_POINTERS ExceptionPointers) {
        if (ExceptionPointers->ExceptionRecord->ExceptionCode
            == EXCEPTION_ACCESS_VIOLATION) return EXCEPTION_EXECUTE_HANDLER;
        return EXCEPTION_CONTINUE_SEARCH;
    }
    void stackwalk(void* ebp) {
        PCALL_FRAME frame = (PCALL_FRAME)ebp;
        printf("Trying ebp %p\n", ebp);
        __try {
            for (unsigned i = 0; i < 100; i++) {
                if ((ULONG_PTR)frame & 0x3) {
                    printf("Misaligned frame\\n");
                    break;
                }
                printf("#%02lu %p  [@ %p]\n", i, frame, frame->ReturnAddress);
                frame = frame->Next;
            }
        } __except(StackwalkExceptionHandler(GetExceptionInformation())) {
            printf("Caught exception\\n");
        }
    }
    #pragma optimize("y", off)
    __declspec(noinline) void printstack() {
        void* ebp = (ULONG*)_AddressOfReturnAddress() - 1;
        stackwalk(ebp);
    }
    #pragma optimize("", on)

如果我们重新编译程序，禁用 FPO 优化，并在 f3 函数中插入对 printstack 的调用，控制台输
出如下。换句话说，在没有使用任何符号的情况下，我们成功地在 x86 上进行了调用栈遍历。 ::

    #00 0012FEB0  [@ 0100185C]
    #01 0012FED0  [@ 010018B4]
    #02 0012FEF8  [@ 0100190B]
    #03 0012FF2C  [@ 01001965]
    #04 0012FF5C  [@ 01001E5D]
    #05 0012FFA0  [@ 77573833]
    #06 0012FFAC  [@ 7740A9BD]
    #07 0012FFEC  [@ 00000000]
    Caught exception

然而，当调用栈中的某个函数没有使用帧指针（即启用了 FPO 优化）时，这一切都会崩溃。在这种
情况下，假设 ebp 指向一个 CALL_FRAME 结构的假设不再有效，调用栈要么被截断，要么完全错
误（特别是如果该函数将 ebp 用于除帧指针以外的其他用途）。尽管可以使用启发式方法尝试猜测
结构上真正的调用和返回的地址记录，但这实际上只是一个有根据的猜测，通常至少会有一些错误并
且通常会完全丢失一个或多个帧）。

你可能会好奇，为什么需要在没有符号的情况下进行调用栈遍历。毕竟，你有 Microsoft 二进制
文件（如 kernel32）的符号，这些符号可以从 Microsoft 符号服务器获得，而且你（假设）有与
你自己的程序对应的私有符号，用于调试问题。然而答案是，在正常的调试过程中，你会遇到各种问
题，需要在没有符号的情况下记录调用栈。原因是 NTDLL（和 NTOSKRNL）中内置了大量支持，用
于调试一类特别棘手的问题：句柄泄漏（以及其他问题，如错误的句柄值在某处被关闭，你需要找出
原因）、内存泄漏和堆损坏。

这些（非常有用的）调试功能提供了选项允许你进行配置，让系统在每次堆分配、堆释放或每次打开
或关闭句柄时记录调用栈。这些功能的工作方式是，在堆操作或句柄操作发生时实时捕获调用栈，但
不是尝试中断到调试器以显示此输出结果（这在许多情况下是不可取的），而是将当前的调用栈副本
保存在内存中，然后正常继续执行。要显示这些保存的调用栈，可以使用 !htrace、!heap -p 和
!avrf 命令，这些命令具有定位内存中保存的调用栈并将其打印出来的功能。

然而，NTDLL/NTOSKRNL 需要一种方法来首先创建这些调用栈，以便可以为后续检查保存它们。这
里有几个要求：

1. 捕获调用栈的功能不能依赖于 NTDLL 或 NTOSKRNL 之上的任何东西。这意味着任何复杂的操
   作，如通过 DbgHelp 下载和加载符号，立刻被排除在外，因为这些函数层次结构远高于
   NTDLL/NTOSKRNL（实际上，它们必须调用同样会记录调用栈的函数）。

2. 该功能必须在调用栈上没有可用符号的情况下工作。例如，这些功能必须能够在客户计算机上部
   署，而不以某种方式让该计算机访问你的私有符号。因此，即使有好的方法来定位符号，调用栈
   被捕获时也无法找到符号。

3. 该功能必须在内核模式下工作（用于保存句柄跟踪），因为句柄跟踪部分由内核本身管理，而不
   仅仅是 NTDLL。

4. 该功能必须使用最少的内存来存储每个调用栈，因为堆分配、堆释放、句柄创建和句柄关闭等操
   作在进程的生命周期中是非常频繁的操作。因此，像在符号可用时保存整个线程栈这样的选项无
   法使用，因为这在每个保存的调用栈上会占用过多的内存。

考虑到所有这些限制，负责保存调用栈的代码需要在没有符号的情况下运行，并且必须能够以非常简
洁的方式保存调用栈（而不需要为每个调用栈使用大量内存）。

因此，在 x86 上，NTDLL 和 NTOSKRNL 中的调用栈保存代码假设调用帧中的所有函数都使用帧指
针。这是保存没有符号的调用栈的唯一现实选择，因为每个单独编译的二进制文件中没有足够的信息
来可靠地执行调用栈遍历，只有靠使用帧指针。（Windows 支持的 64 位平台通过使用广泛的展开
元数据解决了这个问题。）

如果你确保在所有代码中禁用 FPO 优化，那么你将能够使用像 pageheap 在堆操作上的调用栈跟
踪、UMDH（用户模式堆调试器）、以及句柄跟踪等工具来追踪堆相关问题和句柄相关问题。这些功
能的最佳部分是，你甚至可以在客户现场部署它们，而无需安装完整的调试器（或在调试器下运行你
的程序），只需稍后获取进程的迷你转储以便在实验室中检查。所有这些功能都依赖于 FPO 优化被
禁用（至少在 x86 上），因此请记得在发布构建中关闭 FPO 优化，以提高这些难以发现的问题的
可调试性。
