函数调用约定
============

优化栈帧指针
------------

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

参考链接：

* http://www.nynaeve.net/?p=97
* http://www.nynaeve.net/?page_id=67
