GDB调试器参考
=============

调试器的目的，是让你能够运行另一个程序，并查看该程序 “内部” 的运行情况，或者了解程序崩
溃时正在执行的操作。gdb 主要能做四类事情以及相关的辅助性事情，帮助你在程序运行过程中捕
捉错误：

1. 启动你的程序，并指定可能影响其行为的任何事情
2. 使程序在指定条件下停止运行
3. 当程序停止时，检查发生了什么
4. 修改程序内容，尝试修正某个错误，并继续查看其他错误

你可以使用 gdb 调试用 C 和 C++ 编写的程序。gdb 对 D 语言的支持是部分性的。对Modula-2
语言的支持也是部分性的。对 OpenCL C 的支持同样是部分性的。目前，gdb 无法调试使用集合、
子界、文件变量或嵌套函数的 Pascal 程序。gdb 不支持使用 Pascal 语法输入表达式、打印值
或类似功能。gdb 可用于调试用 Fortran 编写的程序，不过可能需要在某些变量名后加上下划线
来引用它们。gdb 还能使用 Apple/NeXT 或 GNU 的 Objective-C 运行时系统，调试用
Objective-C 编写的程序。

一个简单的gdb调试会话： ::

    指定需要调试的可执行程序
    $ gdb executable_program
    (gdb)

    指定需要调试的可执行程序并传递参数
    $ gdb --args executable_program arg1 arg2 ...
    (gdb)

    调试一个正在执行的进程
    $ gdb -p pid
    (gdb)

    使用命令关联一个进程或一个远端进程
    (gdb) attach pid
    (gdb) target remote ip:port

    使用file命令指定可执行程序
    $ gdb
    (gdb) file executable_program

    在run之前使用set args清除参数
    (gdb) set args

    在run之前使用set args设置参数
    (gdb) set args arg1 arg2 ...

    设置显示宽度
    (gdb) set width 70

    在进入函数时设置程序断点，b表示breakpoint
    (gdb) b main

    在对应代码行设置程序断点
    (gdb) b LINENUM

    当变量内容发生改变时进行断点
    (gdb) watch *ADDRESS

    显示所有的断点
    (gdb) info break

    删除断点，d表示delete
    (gdb) d BREAK_NUM ...

    运行程序，直到某个设置的断点暂停，r表示run
    (gdb) r

    执行到当前函数的下一行，n表示next
    (gdb) n

    单步调试执行到下一行，会进入到子函数的内部，s表示step
    (gdb) s

    打印当前的栈帧信息，bt表示backtrace
    (gdb) bt

    打印变量的值，p表示print，p可以打印任何表达式的值并产生效果，包括函数调用和赋值
    (gdb) p variable_name
    (gdb) p len = strlen(str)

    查看上下附近10行源代码，l表示list
    (gdb) l

    调试命令帮助
    (gdb) help l

    继续执行程序到下一个断点或执行完毕，c表示continue
    (gdb) c

    使用quit或使用ctrl-d输出EOF进行退出，q表示quit
    (gdb) q

源程序显示： ::

    以行号或函数起始行为中心显示10行代码
    (gdb) l LINENUM
    (gdb) l FUNCTION

    以源代码指令地址为中心显示10行代码
    (gdb) l *ADDRESS

    显示两个行号之间的内容
    (gdb) l 100,200

    显示包含起始行在内的之后10行内容
    (gdb) l 100,

    显示包含结束行在内的之前10行内容
    (gdb) l ,200

    当前的代码显示行数，默认是显示10行
    (gdb) show listsize

    设置代码代码显示行数
    (gdb) set listsize 20

    如果程序编译时去除或替换了源代码目录前缀，添加源代码根目录以正常显示源代码，命令
    directory dir1 dir2 ... 将目录添加到源代码搜索目录列表的前面，如果对应的目录已
    经存在，会将对应目录移动到最前面，dir表示directory
    (gdb) dir src_root_dir

    显示当前的源代码搜索目录列表
    (gdb) show directories

    显示当前指令指针所在函数的汇编源代码，disas表示disassemble
    (gdb) disas

    显示指定函数或对应地址所在函数的汇编源代码
    (gdb) disas FUNCTION
    (gdb) disas ADDRESS

    显示两个地址之间的汇编源代码
    (gdb) disas start,end

    显示对应地址开始指定长度的汇编源代码
    (gdb) disas start,+length

    不仅显示汇编助记符，还显示二进制机器码
    (gdb) disas /r

    在x86平台上，可以将汇编代码风格设置为intel，默认风格是att
    (gdb) set disassembly-flavor intel

    查看当前的汇编代码风格
    (gdb) show disassembly-flavor

    单步调式时，不仅显示下一行源代码，还显示下一行源代码的汇编代码
    (gdb) set disassemble-next-line on

    查看显示下一行汇编源代码的选项是否打开
    (gdb) show disassemble-next-line

    启用TUI（Text User Interface）模式
    (gdb) tui enable

    退出TUI模式
    (gdb) tui disable

    显示源代码分屏
    (gdb) layout src

    显示汇编代码分屏
    (gdb) layout asm

    同时显示源代码和汇编代码两个分屏
    (gdb) layout split

    显示寄存器内容分屏，如果还指定了layout src或asm，会自动显示两个分屏
    (gdb) layout regs

单步调式命令： ::

    执行到当前函数的下一行，n表示next
    (gdb) n

    执行当前指令地址处的一条指令，ni表示nexti
    (gdb) ni

    单步调试执行到下一行，会进入到子函数的内部，s表示step
    (gdb) s

    执行当前指令地址处的一条指令，如果遇到函数调用指令会进入到子函数内部，si表示stepi
    (gdb) si

    当使用ni和si执行当前指令时，可以使用disass显示对应指令地址附近的汇编源代码
    (gdb) disas $rip,+80

    结束当前函数栈帧，返回到调用者，fin表示finish
    (gdb) fin

打印程序信息： ::

    打印所有寄存器的内容
    (gdb) info registers

    打印函数参数
    (gdb) info args

    打印函数局部变量
    (gdb) info locals

    打印所有的全局变量
    (gdb) info variables

    打印单个寄存器值
    (gdb) p $rax
    (gdb) p *(long long *)($rsp + 8)

    打印前面打印过的值
    (gdb) $             上一次打印过的值，相当于$$0
    (gdb) $$            上上次打印过的值，相当于$$1
    (gdb) $$2           上第3次打印过的值

    给表达式的值取一个方便记忆的名字，不能与寄存器的名字重名，否则是给寄存器赋值
    (gdb) $name = expr

    打印一个以foo开始的10个长度的数组
    (gdb) p foo@10

    打印对应地址处指定类型的数据的值
    (gdb) p {type}addr

    以十六进制进行打印
    (gdb) p/x expr

    打印结构体成员信息，o表示offset
    (gdb) ptype /o type

    检查内存地址中的内容，x表示examine
    c表示计数，多少个对应大小的对象，默认打印1个
    f表示格式，c(char) s(string) f(float)，默认是前一次使用过的格式
              t(binary) o(octal) d(decimal) u(unsigned decimal)
              x(hex) z(hex zero padded on the left) a(address) i(instruction)
    s对象大小，默认是前一次使用过的大小
              b(byte) h(halfword 2-byte) w(word 4-byte) g(giant 8-byte)
    addr是一个表示内存地址的表达式，默认是前一个 x 或 p 命令执行完之后的后一个地址
    (gdb) x/cfs addr

    每次执行之后，自动显示指定的信息
    (gdb) display $rsp

    查看当前所有的自动显示
    (gdb) info display

    查看当前所有自动显示信息以及对应的值
    (gdb) display

    根据编号删除对应的自动显示
    (gdb) undisplay DNUM ...

程序断点
--------

断点的作用是，每当程序运行到某个特定位置时，使程序停止。对于每个断点，你可以添加条件，
以便更精细地控制程序是否停止。你可以使用break命令及其变体来设置断点，通过行号、函数名
或程序中的精确地址来指定程序应该停止的位置。在某些系统中，你可以在可执行文件运行之前，
在共享库中设置断点。

观察点是一种特殊的断点，当一个表达式的值发生变化时，它会使程序停止。该表达式可以是一个
变量的值，也可以是由运算符组合的一个或多个变量的值，例如a + b。这有时也被称为数据断
点。你必须使用不同的命令来设置观察点，但除此之外，你可以像管理其他断点一样管理观察点：
使用相同的命令来启用、禁用和删除断点和观察点。

你可以设置，每当 GDB 在断点处停止时，自动显示程序中的值。

捕获点是另一种特殊的断点，当发生某种特定事件时，例如抛出 C++ 异常或加载库，它会使程序
停止。与观察点一样，你需要使用不同的命令来设置捕获点，但除此之外，你可以像管理其他断点
一样管理捕获点。要在程序接收到信号时停止，可使用handle命令。

当你创建断点、观察点或捕获点时，GDB 会给它们各自分配一个编号；这些编号是从 1 开始的连
续整数。在许多用于控制断点各种特性的命令中，你可以使用断点编号来指明要修改的是哪个断
点。每个断点都可以被启用或禁用；如果被禁用，在你再次启用它之前，它对程序不会产生任何影
响。

一些 GDB 命令接受以空格分隔的断点列表作为操作对象。列表中的元素可以是单个断点编号，
如5，也可以是一个编号范围，如5-7。当将一个断点列表提供给某个命令时，该列表中的所有断
点都会被操作。

自动显示
--------

如果你发现自己想要频繁打印某个表达式的值（以查看其变化情况），你可以将其添加到自动显示
列表中，这样每当程序停止时，GDB 就会打印该表达式的值。添加到列表中的每个表达式都会被赋
予一个编号以便识别；若要从列表中移除某个表达式，只需指定该编号即可。自动显示的内容类似
如下形式： ::

    2: foo = 38
    3: bar[5] = (struct hack *) 0x3804

这种显示方式展示了条目编号、表达式及其当前值。就像你手动使用x或print命令进行显示一样，
你可以指定自己偏好的输出格式；实际上，display命令会根据你指定的格式来决定使用print还
是x命令。如果你指定了'i'或's'格式，或者指定了单位大小，它就会使用x命令；否则，就使用
print命令。

*display expr* ::

    将表达式expr添加到每次程序停止时要显示的表达式列表中。使用该命令后，若再次按下回车
    键，该命令不会重复执行。

*display/fmt expr* ::

    当fmt仅指定显示格式而不指定大小或数量时，将表达式expr添加到自动显示列表中，并安排
    每次都以指定的格式fmt显示该表达式。

*display/fmt addr* ::

    当fmt为'i'或's'，或者包含单位大小或单位数量时，将表达式addr作为内存地址添加到列表
    中，以便每次程序停止时对其进行检查。检查实际上相当于执行'x/fmt addr'命令。例如，
    display/i $pc会很有用，它能让你在每次程序执行停止时查看即将执行的机器指令。

*undisplay dnums... delete display dnums...* ::

    从要显示的表达式列表中移除条目。通过命令参数dnums指定你想要操作的显示条目的编号。
    它可以是单个显示编号，即info display显示结果中第一列所显示的编号之一；也可以是一
    个显示编号范围，如2 - 4。使用undisplay命令后，若按下回车键，该命令不会重复执行。

*disable display dnums...* ::

    禁用编号为dnums的显示条目。被禁用的显示条目不会自动打印，但不会被遗忘，之后你可以
    再次启用它。通过命令参数dnums指定你想要操作的显示条目的编号。它可以是单个显示编
    号，即info display显示结果中第一列所显示的编号之一；也可以是一个显示编号范围，如
    2 - 4。

*enable display dnums...* ::

    启用编号为dnums的显示条目。启用后，该表达式将再次在自动显示中生效，直到你另行指
    定。通过命令参数dnums指定你想要操作的显示条目的编号。它可以是单个显示编号，即
    info display显示结果中第一列所显示的编号之一；也可以是一个显示编号范围，如
    2 - 4。

*display* ::

    显示列表中表达式的当前值，就像程序停止时那样。

*info display* ::

    打印之前设置为自动显示的表达式列表，每个表达式都带有其条目编号，但不显示其值。这包
    括已禁用的表达式，这些表达式会被标记出来。此外，还包括那些由于引用了当前不可用的自
    动变量而此时不会显示的表达式。

如果一个显示表达式引用了局部变量，那么在其设置时的词法上下文之外，该表达式就没有意义
了。当程序执行进入某个上下文，其中该表达式所引用的变量未定义时，该表达式会被禁用。例
如，如果你在一个带有参数last_char的函数内部使用display last_char命令，那么在程序继
续在该函数内部停止时，GDB 会显示这个参数的值。当程序在其他地方停止（即没有last_char
变量的地方）时，该显示会自动被禁用。下次程序在last_char有意义的地方停止时，你可以再
次启用该显示表达式。
