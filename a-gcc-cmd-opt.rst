编译器命令行选项
================

1. `GCC 简介`_
2. `环境变量`_
3. `通用选项`_
4. `C 特定选项`_
5. `C++ 特定选项`_
6. `预处理选项`_
7. `预编译头文件`_
8. `目录选项`_
9. `诊断消息格式`_
10. `警告选项`_
11. `静态分析选项`_
12. `调试选项`_
13. `优化选项`_
14. `程序指令选项`_
15. `代码生成选项`_
16. `开发者选项`_
17. `汇编选项`_
18. `链接选项`_
19. `特定平台选项`_

GCC 简介
--------

GCC 代表 “GNU Compiler Collection”。GCC 是一个集成的编译器发行版，支持多种主要编程语
言。这些语言目前包括 C、C++、Objective-C、Objective-C++、Fortran、Ada、D 和 Go。GCC
这个缩写在常见用法中有多种含义。当前的官方含义是 “GNU Compiler Collection”，它泛指完
整的工具套件。历史上，GCC 代表 “GNU C Compiler”，当重点是编译 C 程序时，这种用法仍然
很常见。最后，当谈论 GCC 的语言无关组件时，也会使用这个名字：所有支持的语言的编译器共享
的代码。GCC 的语言无关组件包括大多数优化器，以及为各种处理器生成机器代码的 “后端”。编译
器中特定于某种语言的部分称为 “前端”。除了作为 GCC 集成组件的前端外，还有几个其他前端是
单独维护的。这些支持 Mercury 和 COBOL 等语言。要使用这些语言，它们必须通过合适的方式与
GCC一起进行构建。除了 C 语言之外，大多数其他语言的编译器都有自己的名称。C++ 编译器是
G++，Ada 编译器是 GNAT，等等。当我们谈论编译这些语言之一时，我们可以用它自己的名称或
GCC 来指代该编译器。两者都是正确的。历史上，包括 C++ 和 Fortran 在内的许多语言的编译器
都是作为 “预处理器” 实现的，这些预处理器会生成另一种高级语言，如 C。GCC 中包含的编译器
没有一个是以这种方式实现的，它们都直接生成机器代码。这种预处理器不应与 C 预处理器混淆，
C 预处理器是 C、C++、Objective-C 和 Objective-C++ 语言的一个重要特性。

**C 语言支持**

对于每种由 GCC 编译且有标准的语言，GCC 试图遵循该语言的一个或多个版本的标准，可能有一些
例外，也可能有一些扩展。

原始 ANSI C 标准（X3.159-1989）于 1989 年获得批准，并于 1990 年发布。该标准于 1990
年晚些时候被批准为 ISO 标准（ISO/IEC 9899:1990）。尽管 ANSI 标准的章节被重新编号并成
为 ISO 标准的条款，但这两个版本在技术上没有区别。ANSI 标准（而非 ISO 标准）还附带了一
份原理说明文件（Rationale document）。这个标准，无论是哪种形式，通常被称为 C89，偶尔
也被称为 C90，这是根据批准的日期。要在 GCC 中选择此标准，请使用 -ansi、-std=c90 或
-std=iso9899:1990 中的一个选项。若要获得标准所要求的所有诊断信息，应指定 -pedantic
（或者如果希望它们是错误而不是警告，则指定 -pedantic-errors）。参见 C 语言特定选项部
分。1990 年 ISO C 标准中的错误在 1994 年和 1996 年发布的两个技术勘误中得到了修正。GCC
不支持未经修正的版本。1990 年标准的一项修正案于 1995 年发布。该修正案向语言中添加了双
字母代替符（digraphs）和 __STDC_VERSION__，但除此之外主要涉及库。该修正案通常被称为
AMD1；经过修正的标准有时被称为 C94 或 C95。要在 GCC 中选择此标准，请使用
-std=iso9899:199409 选项（与其他标准版本一样，使用 -pedantic 以获得所有所需的诊断信
息）。

ISO C 标准的新版本于 1999 年发布，编号为 ISO/IEC 9899:1999，通常被称为 C99。在开发过
程中，该标准版本的草案被称为 C9X。GCC 对这一标准版本的支持基本完整；详情请参阅
https://gcc.gnu.org/c99status.html。要选择此标准，请使用 -std=c99 或
-std=iso9899:1999。1999 年 ISO C 标准中的错误在 2001 年、2004 年和 2007 年发布的三
个技术勘误中得到了修正。GCC不支持未经修正的版本。

C 语言标准的第四个版本，被称为 C11，于 2011 年发布，编号为 ISO/IEC 9899:2011。在开发
过程中，该标准版本的草案被称为 C1X。GCC 对这一标准的支持基本完整，可通过 -std=c11 或
-std=iso9899:2011 启用。一个集成了修正的版本于 2017 年准备就绪，并于 2018 年发布，编
号为 ISO/IEC 9899:2018，它被称为 C17，可通过 -std=c17 或 -std=iso9899:2017 启用；
这些修正也适用于 -std=c11，这些选项之间唯一的区别是 __STDC_VERSION__ 的值。

C 语言标准的另一个版本，被称为 C23，正在开发中，预计将于 2024 年发布，编号为 ISO/IEC
9899:2024。在开发过程中，该标准版本的草案被称为 C2X。通过 -std=c23 或
-std=iso9899:2024 启用对此的实验性和不完整支持。

默认情况下，GCC 为 C 语言提供了一些扩展，这些扩展在极少数情况下会与 C 标准冲突。参见 C
语言家族扩展部分。C99 标准中的一些特性在 C90 模式下被接受为扩展，C11 标准中的一些特性
在 C90 和 C99 模式下被接受为扩展。使用上述列出的 -std 选项可以禁用与所选 C 标准版本冲
突的这些扩展。您还可以通过 -std=gnu90（带有 GNU 扩展的 C90）、-std=gnu99（带有 GNU
扩展的 C99）或 -std=gnu11（带有 GNU 扩展的 C11）显式选择 C 语言的扩展版本。如果没有指
定任何 C 语言特定选项，则默认为 -std=gnu17。

ISO C 标准在第 4 条款中定义了两种符合标准的实现类别。一种是符合标准的托管实现（hosted
implementation），它支持整个标准，包括所有的库功能；另一种是符合标准的独立实现
（freestanding implementation），它只需要提供某些库功能：即 <float.h>、<limits.h>、
<stdarg.h> 和 <stddef.h> 中定义的功能；自 AMD1 起，还包括 <iso646.h> 中的功能；自
C99 起，还包括 <stdbool.h> 和 <stdint.h> 中的功能；自 C11 起，还包括 <stdalign.h>
和 <stdnoreturn.h> 中的功能。此外，C99 中新增的复数类型对于独立实现并非强制要求。

该标准还定义了两种程序运行环境：一种是独立环境，所有实现都必须提供这种环境，它可能不提供
超出独立实现所要求的库功能，在这种环境中，程序的启动和终止方式由实现定义；另一种是托管环
境，它并非强制要求，它提供所有的库功能，并通过 ``int main(void)`` 或
``int main(int, char*[])`` 函数启动。操作系统内核是运行在独立环境中的程序的一个例子；
使用操作系统功能的程序是运行在托管环境中的程序的一个例子。

GCC 旨在既可以作为符合标准的独立实现使用，也可以作为符合标准的托管实现的编译器使用。默
认情况下，它作为托管实现的编译器运行，将 __STDC_HOSTED__ 定义为 1，并假定当使用 ISO C 
函数的名称时，它们具有标准中定义的语义。若要使其作为独立环境中的符合标准的独立实现运行，
请使用 -ffreestanding 选项；它随后将 __STDC_HOSTED__ 定义为 0，并不对标准库中函数名
称的含义做出假设，但下面会提到一些例外情况。若要构建操作系统内核，你可能仍需自行安排链接
和启动事宜。参见 C 语言特定选项部分。

GCC 并不提供仅托管实现所需的库功能，也尚未在所有平台上为独立实现提供 C99 所需的所有功
能。若要使用托管环境的设施，你需要在其他地方找到它们（例如在 GNU C 库中），参见标准库部
分。GCC 所使用的大多数编译器支持例程都包含在 libgcc 中，但也有一些例外。GCC 要求独立环
境提供 memcpy、memmove、memset 和 memcmp。与涵盖 memcpy 的标准相反，GCC 期望源和目标
完全重叠的情况能够正常工作，而不是引发未定义行为。最后，如果使用了 __builtin_trap，并
且目标未实现陷阱模式，那么 GCC 将发出对 abort 的调用。关于技术勘误、原理文件以及在线可
用的 C 语言历史信息的参考，可参考链接 https://gcc.gnu.org/readings.html。

**C++ 语言支持**

GCC 支持 1998 年发布的原始 ISO C++ 标准，以及 2011 年、2014 年、2017 年和大部分 2020
年的修订版。原始 ISO C++ 标准作为 ISO 标准（ISO/IEC 14882:1998）发布，并通过 2003 年
发布的技术勘误（ISO/IEC 14882:2003）进行了修订。这些标准分别被称为 C++98 和 C++03。
GCC 实现了 C++98 的大部分内容（export 是一个显著的例外）以及 C++03 中的大多数变更。要
在 GCC 中选择此标准，请使用 -ansi、-std=c++98 或 -std=c++03 中的一个选项；若要获得标
准所要求的所有诊断信息，您还应指定 -pedantic（或者如果您希望它们是错误而不是警告，则指
定 -pedantic-errors）。

2011 年发布的修订版 ISO C++ 标准为 ISO/IEC 14882:2011，被称为 C++11；在发布之前，它
通常被称为 C++0x。C++11 对 C++ 语言进行了若干变更，所有这些变更都已在 GCC 中实现。详
情请参阅 https://gcc.gnu.org/projects/cxx-status.html#cxx11。要在 GCC 中选择此标
准，请使用 -std=c++11 选项。

2014 年发布的另一版修订版 ISO C++ 标准为 ISO/IEC 14882:2014，被称为 C++14；在发布之
前，它有时被称为 C++1y。C++14 对 C++ 语言进行了若干进一步的变更，所有这些变更都已在
GCC 中实现。详情请参阅 https://gcc.gnu.org/projects/cxx-status.html#cxx14。要在
GCC 中选择此标准，请使用 -std=c++14 选项。

C++ 语言在 2017 年进一步进行了修订，并发布了 ISO/IEC 14882:2017。这被称为 C++17，在
发布之前通常被称为 C++1z。GCC 支持该规范中的所有变更。更多详情请参阅
https://gcc.gnu.org/projects/cxx-status.html#cxx17。使用 -std=c++17 选项可以选择
此 C++ 变体。

2020 年发布的另一版修订版 ISO C++ 标准为 ISO/IEC 14882:2020，被称为 C++20；在发布之
前，它有时被称为 C++2a。GCC 支持新规范中的大部分变更。更多详情请参阅
https://gcc.gnu.org/projects/cxx-status.html#cxx20。要在 GCC 中选择此标准，请使用
-std=c++20 选项。

关于 C++ 标准的更多信息可在 ISO C++ 委员会的网站
https://www.open-std.org/jtc1/sc22/wg21/ 上找到。若要获得上述任何标准版本所要求的所
有诊断信息，您应指定 -pedantic 或 -pedantic-errors，否则 GCC 将允许一些非 ISO C++
特性作为扩展。参见警告选项部分。

默认情况下，GCC 还为 C++ 语言提供了一些额外的扩展，这些扩展在极少数情况下会与 C++ 标准
冲突。参见 C++ 特定选项部分。使用上述列出的 -std 选项会禁用那些与所选 C++ 标准版本冲突
的扩展。您还可以通过 -std=gnu++98（带有 GNU 扩展的 C++98）、-std=gnu++11（带有 GNU
扩展的 C++11）、-std=gnu++14（带有 GNU 扩展的 C++14）、-std=gnu++17（带有 GNU 扩展
的 C++17）或 -std=gnu++20（带有 GNU 扩展的 C++20）显式选择 C++ 语言的扩展版本。如果
没有指定任何 C++ 语言特定选项，则默认为 -std=gnu++17。

**GCC 命令行**

当你调用 GCC 时，它通常会执行预处理、编译、汇编和链接。“总体选项” 或通用选项允许你在中
间阶段停止这一过程。例如，-c 选项表示不运行链接器。那么输出就由汇编器输出的目标文件组
成。其他选项会被传递到一个或多个处理阶段。有些选项控制预处理器，有些控制编译器本身。还有
一些选项控制汇编器和链接器，由于你很少需要使用它们中的任何一个，因此这里没有对它们进行说
明，可另外参考 GNU as 汇编器和 GNU ld 链接器的文档。

大多数可以与 GCC 一起使用的命令行选项对 C 程序很有用；当某个选项只对另一种语言（通常是
C++）有用时，解释中会明确说明。如果某个特定选项的描述没有提到源语言，那么你可以用该选项
来处理所有支持的语言。

运行 GCC 的通常方式是运行名为 gcc 的可执行文件，或者在交叉编译时运行 machine-gcc，或
者运行 machine-gcc-version 以运行 GCC 的特定版本。当你编译 C++ 程序时，你应该使用
g++ 而不是 gcc，参见编译 C++ 程序部分了解 gcc 和 g++ 在编译 C++ 程序时行为差异的信
息。

gcc 程序接受选项和文件名作为操作数。许多选项的名称由多个字母组成，因此多个单字母选项不
能组合在一起，-dv 与 ‘-d -v’ 有很大区别。你可以将选项和其他参数一起混合使用，在大多数
情况下，你使用的顺序并不重要。当你使用同一类别的多个选项时，顺序是重要的；例如，如果你多
次指定 -L，目录将按指定的顺序被搜索，此外 -l 选项的位置也很重要。许多选项的名称以 ‘-f’
或 ‘-W’ 开头，例如 -fmoveloop-invariants、-Wformat 等。这些选项大多有正负两种形式，
-ffoo 的负形式是 -fno-foo。本手册只记录这两种形式中的一种，即非默认的那种。

有些选项需要一个或多个参数，这些参数通常用空格或等号（‘=’）与选项名称分隔。除非另有说
明，参数可以是数字或字符串。数字参数通常是较小的无符号十进制或十六进制整数。十六进制参数
必须以 ‘0x’ 前缀开头。指定某种大小阈值的选项的参数可以是任意大的十进制或十六进制整数，
后面跟着一个字节大小后缀，表示字节的倍数，如 kB 和 KiB 分别表示 kilobyte（1000-byte）
和 kibibyte（1024-byte），MB 和 MiB 分别表示 megabyte 和 mebibyte，GB 和 GiB 分别
表示 gigabyte 和 gigibyte，等等。有关二进制和十进制字节大小前缀的完整列表和解释，请参
阅 NIST、IEC 以及其他相关的国家和国际标准。

环境变量
--------

以下是影响 GCC 运行的一些环境变量，其中一些通过指定目录或前缀来添加搜索相关文件时的路
径，另外一些则配置编译环境的其他方面。请注意，你还可以使用 -B、-I 和 -L 等选项来指定搜
索路径。这些选项优先于通过环境变量指定的值，而环境变量指定的值又优先于 GCC 的配置。

**LC_CTYPE LC_MESSAGES LC_ALL** ::

    这些环境变量控制 GCC 使用本地化信息的方式，这使得 GCC 能够适应不同的国家习惯。如果
    GCC 进行了配置，它会检查 LC_CTYPE 和 LC_MESSAGES 这些本地化类别。这些本地化类别
    可以设置成所安装支持的任何值，一个典型的值是 en_GB.UTF-8，表示用于英国的英语，编码
    为 UTF-8。LC_CTYPE 环境变量指定字符分类，GCC 使用它来确定字符串中字符的边界，这在
    某些包含引号和转义字符的多字节编码中是必需的，这些字符在其他情况下会被解释为字符串
    结束或转义。LC_MESSAGES 环境变量指定在诊断消息中使用的语言。如果设置了 LC_ALL 环
    境变量，它将覆盖 LC_CTYPE 和 LC_MESSAGES 的值，否则 LC_CTYPE 和 LC_MESSAGES 默
    认为 LANG 环境变量的值。如果这些变量都没有设置，GCC 默认使用传统 C 英语行为。

**TMPDIR** ::

    如果设置了 TMPDIR，它指定用于临时文件的目录。GCC 使用临时文件来保存一个编译阶段的
    输出，这些输出将被用作下一个阶段的输入，例如预处理器的输出是编译器本身的输入。

**GCC_COMPARE_DEBUG** ::

    设置 GCC_COMPARE_DEBUG 几乎等同于向编译器驱动程序传递 -fcompare-debug。有关此选
    项的更多详细信息，参阅开发者选项部分。

**GCC_EXEC_PREFIX** ::

    如果设置了 GCC_EXEC_PREFIX，它指定编译器执行的子程序名称使用的前缀。这个前缀与子程
    序名称组合时，不会添加斜杠，但如果你愿意，可以指定一个以斜杠结尾的前缀。如果没有设
    置 GCC_EXEC_PREFIX，GCC 会尝试根据它被调用的路径来确定一个合适的前缀。如果 GCC 无
    法使用指定的前缀找到子程序，它会尝试在通常的位置寻找子程序。GCC_EXEC_PREFIX 的默认
    值是 prefix/lib/gcc/，其中 prefix 是安装编译器的前缀。在许多情况下，prefix 是你
    运行 configure 脚本时的 prefix 值。

    使用 -B 指定的其他前缀优先于该前缀。这个前缀也用于查找用于链接的文件比如 crt0.o。
    此外，该前缀在查找头文件目录时以一种不寻常的方式使用。对于每个通常以
    ‘/usr/local/lib/gcc’（更准确地说是 GCC_INCLUDE_DIR 的值）开头的标准目录，GCC
    会尝试使用指定的前缀进行替换，以产生一个替代的目录名称。例如使用 -Bfoo/，GCC 会在
    搜索标准目录 /usr/local/lib/bar 之前搜索 foo/bar。如果一个标准目录以配置的前缀开
    头，那么在查找头文件时，prefix 的值将被 GCC_EXEC_PREFIX 替换。

**COMPILER_PATH** ::

    COMPILER_PATH 的值是一个由冒号分隔的目录列表，类似于 PATH。如果 GCC 无法使用
    GCC_EXEC_PREFIX 找到子程序，它会尝试使用该变量指定的目录来搜索子程序。

**LIBRARY_PATH** ::

    LIBRARY_PATH 的值是一个由冒号分隔的目录列表，类似于 PATH。当配置为本地编译器时，
    GCC 会尝试使用该变量指定的目录来搜索特殊的链接器文件，如果它无法使用
    GCC_EXEC_PREFIX 找到它们。使用 GCC 进行链接时，这些目录也会被用于搜索 -l 选项指定
    的普通库（但使用 -L 指定的目录会优先）。

**LANG** ::

    此变量用于将本地化信息传递给编译器。使用此信息的一种方式是确定在 C 和 C++ 中解析字
    符字面量、字符串字面量和注释时使用的字符集。当编译器被配置为允许多字节字符时，以下
    是 LANG 可识别的值：‘C-JIS’ 识别 JIS 字符，‘C-SJIS’ 识别 SJIS 字符，‘C-EUCJP’
    识别 EUCJP 字符。如果 LANG 未定义，或者它有其他值，那么编译器将使用默认本地化定义
    的 mblen 和 mbtowc 来识别和转换多字节字符。

**GCC_EXTRA_DIAGNOSTIC_OUTPUT** ::

    如果设置了 GCC_EXTRA_DIAGNOSTIC_OUTPUT 其值为以下之一，则在生成修复提示时，会向
    stderr 输出额外的文本。-fdiagnostics-parseable-fixits 和
    -fno-diagnostics-parseable-fixits 优先于此环境变量。
    
    ‘fixits-v1’ 输出可解析的修复提示，等同于 -fdiagnostics-parseable-fixits。特别
    是，列号表示的是从第一个字节开始的字节数。‘fixits-v2’ 与 fixits-v1 类似，但列号表
    示显示列数，与 -fdiagnostics-column-unit=display 一致。

下面这些额外的环境变量影响预处理器的行为：

**CPATH C_INCLUDE_PATH CPLUS_INCLUDE_PATH OBJC_INCLUDE_PATH** ::

    每个变量的值是一个由特殊字符分隔的目录列表，类似于 PATH，用于查找头文件。特殊字符
    PATH_SEPARATOR 是目标平台依赖的，并在 GCC 构建时确定。对于基于 Microsoft
    Windows 平台它是分号，对于几乎所有其他平台它是冒号。CPATH 指定一个目录列表，这些目
    录将被搜索，就像用 -I 指定的一样，但是在命令行上用 -I 选项给出的路径之后。无论预处
    理哪种语言都会使用这个环境变量。
    
    其余环境变量仅适用于预处理特定语言。每个指定的目录列表都会被搜索，就像是使用
    -isystem 选项指定的一样，但是在命令行上用 -isystem 选项给出的路径之后。在所有这些
    变量中，空元素指示编译器搜索其当前工作目录。空元素可以出现在路径的开头或结尾。例如
    如果 CPATH 的值是 :/special/include，那么它与 ‘-I. -I/special/include’ 具有相
    同的效果。

**DEPENDENCIES_OUTPUT** ::

    如果设置了此变量，其值指定如何基于编译器处理的非系统头文件为 Make 输出依赖关系。系
    统头文件在依赖输出中被忽略。DEPENDENCIES_OUTPUT 的值可以只是一个文件名，这种情况
    下，Make 规则被写入该文件，并从源文件名猜测目标名。或者值可以是 ‘file target’ 的
    形式，这种情况下规则被写入文件 file，并使用 target 作为目标名。换句话说，这个环境
    变量等同于结合使用 -MM 和 -MF 选项，并且还可以选择性地使用 -MT 选项。

**SUNPRO_DEPENDENCIES** ::

    此变量与 DEPENDENCIES_OUTPUT 相同，只是系统头文件不被忽略，因此它意味着 -M 而不
    是 -MM。然而，对主输入文件的依赖被省略了。

**SOURCE_DATE_EPOCH** ::

    如果设置了此变量，其值指定一个 UNIX 时间戳，用于替换 __DATE__ 和 __TIME__ 宏中的
    当前日期和时间，从而使嵌入的时间戳变得可重现。SOURCE_DATE_EPOCH 的值必须是一个
    UNIX 时间戳，定义为自 1970 年 1 月 1 日 00:00:00（不包括闰秒）以来的秒数，以
    ASCII 形式表示；与 GNU/Linux 和其他支持 date 命令中 %s 扩展的系统上的 date +%s
    输出相同。该值应该是已知的时间戳，例如源文件或软件包的最后修改时间，并且应该由构建
    过程设置。

通用选项
---------

编译过程最多可以分为四个阶段：预处理（将未预处理的源文件加工成预处理完的源文件）、编译
（将源代码编译成汇编代码）、汇编（将汇编代码汇编成机器码形成目标文件）、和链接（链接多个
由机器码组成目标文件），顺序始终是这样。GCC 能够对多个文件进行预处理和编译，生成多个汇
编输入文件，或者生成一个汇编输入文件；然后每个汇编输入文件生成一个目标文件，链接将所有目
标文件（新编译的以及作为输入指定的）组合成一个可执行文件。对于任何给定的输入文件，文件名
后缀决定了要执行的编译类型。 ::

    .c      C 源文件需要预处理
    .i      C 源文件不需要预处理
    .ii     C++ 源文件不需要预处理
    .m      Object-C 源文件需要预处理
    .mi     Object-C 源文件不需要预处理
    .mm .M  Object-C++ 源文件需要预处理
    .mii    Object-C++ 源文件不需要预处理
    .h      C/C++/Object-C/Object-C++ 头文件
    .cc .cp .cxx .cpp .CPP .c++ .C
            C++ 源文件需要预处理
    .hh .H .hp .hxx .hpp .HPP .h++ .tcc
            C++ 头文件
    .f .for .ftn .fi
            固定形式的 Fortran 源代码文件不需要预处理
    .f90 .f95 .f03 .f08 .fii
            自由形式的 Fortran 源代码文件不需要预处理
    .F .FOR .fpp .FPP .FTN
            固定形式的 Fortran 源代码文件需要预处理
    .F90 .F95 .F03 .F08
            自由形式的 Fortran 源代码需要预处理
    .go     Go 语言源文件
    .d      D 语言源文件
    .di     D 语言接口文件
    .dd     D 语言文档代码文件（Ddoc）
    .ads    Ada 包含库单元声明的源代码文件（library unit declaration）
    .adb    Ada 包含库单元实现的源代码文件（library unit body）
    .s      汇编代码
    .S .sx  汇编代码需要预处理
    other   直接当作目标文件喂给链接器，任何没有被识别后缀的文件都按这种方式处理

你可以使用 -x language 选项显式指定输入语言，也可以使用 -x none 关闭任何语言的指定，
以便后续文件根据它们的文件后缀名进行处理，就像没有使用过 -x 选项一样。如果你只需要编译
过程中的某些阶段，你可以使用 -x 或文件名后缀告诉 gcc 从哪里开始，并使用 -c、-S 或 -E
中的一个选项来指定 gcc 在哪里停止。注意某些组合例如 -x cpp-output -E 会指示 gcc 什么
也不做。通用选项汇总： ::

    -x language
    -E -S -c -o file
    -v -### --version
    --help --target-help --help={class|[^]qualifier}[,...]
    @file -pass-exit-codes -pipe -specs=file -wrapper
    -ffile-prefix-map=old=new -fcanon-prefix-map
    -fplugin=file -fplugin-arg-name=arg
    -dumpbase dumpbase -dumpbase-ext auxdropsuf -dumpdir dumppfx
    -fdump-ada-spec[-slim] -fada-spec-parent=unit -fdump-go-spec=file

**-x language|none** ::

    手动指定后续的输入文件类型，不让编译器默认根据文件后缀判定。这个选项应用到后面的所
    有文件，直到下一个 -x 选项。该选项值 lanuage 可以是：

    c c-header cpp-output 源文件 头文件 上次预处理输出的文件
    c++ c++-header c++-system-header c++-user-header c++-cpp-output
    objective-c objective-c-header objective-c-cpp-output
    objective-c++ objective-c++-header objective-c++-cpp-output
    f77 f77-cpp-input f95 f95-cpp-input
    assembler assembler-with-cpp
    ada d go

    或使用 -x none 关掉前面 -x 选项的设置，重新让编译器根据后缀名决定文件的语言类型。

**-E -S -c -o file** ::

    -E 预处理阶段之后就停止，不进行编译，预处理完的源代码会输出到标准输出。如果输入的文
    件是不需要进行预处理的文件，则会忽略掉该文件。

    -S 只进行编译包括预处理不进行汇编，输出源文件的汇编代码。默认汇编代码文件名称是用
    .s 代替 .c、.i 等源文件的后缀名。会忽略不需要进行编译的输入文件。

    -c 对源文件进行预处理、编译、和汇编，但不进行链接，最终的输出是每个源文件对应的目标
    文件。默认目标文件的名称是用 .o 替换源文件的后缀名比如 .c，.i，.s 等等。对于不能识
    别的输入文件类型，或者不需要进行编译或汇编的输入文件，都会被忽略。

    如果不指定 -E -S 和 -c，那么会执行整个预处理、编译、汇编、链接的过程。

    -o file 默认不指定 -o 选项的情况下，可执行文件会输出到 a.out，目标文件会输出到
    .o，汇编文件会输出到 .s，预编译后的头文件 source.suffix 会输出到
    source.suffix.gch，所有预处理后的 C 源文件会输出到标准输出。添加 -o 选项之后，所
    有的输出到输出到指定的文件中，它还会影响辅助输出（auxiliary outputs）和转储输出
    （dump outputs）的输出路径。请参见下面的例子。

    除非被覆盖，否则辅助输出和转储输出都放在与主要输出相同的目录中。在辅助输出中，输入
    文件的后缀被替换为辅助输出文件类型的后缀；在转储输出中，转储文件的后缀附加到输入文
    件的后缀上。在编译命令中，辅助输出和转储输出的基本名称是主要输出的基本名称；在编译
    和链接命令中，是主要输出名称去掉可执行文件后缀之后与输入文件名组合的名称。如果它们
    共享相同的基本名称（忽略后缀），组合的结果就是那个基本名称，否则，它们被连接起来用
    短划线分隔。

    gcc -c foo.c ... 将使用 foo.o 作为主要输出，并将辅助输出和转储文件输出到相同目
    录，例如，对于 -gsplit-dwarf，辅助文件是 foo.dwo，对于 -fdump-rtl-final，转储文
    件是 foo.c.???r.final。

    如果明确指定了非链接器输出文件，辅助文件和转储文件默认使用相同的基本名称：
    gcc -c foo.c -o dir/foobar.o ... 辅助输出为 dir/foobar.，转储输出为
    dir/foobar.c.。

    链接器输出的情况：
    gcc foo.c bar.c -o dir/foobar ... 通常会将辅助输出命名为 dir/foobar-foo. 和
    dir/foobar-bar.，将转储输出命名为 dir/foobar-foo.c. 和 dir/foobar-bar.c.。上述
    情况的唯一例外是，当可执行文件与单个输入共享基本名称时：gcc foo.c -o dir/foo ...
    在这种情况下，辅助输出被命名为 dir/foo.，转储输出被命名为 dir/foo.c.。

    辅助输出和转储输出的位置和名称可以通过选项 -dumpbase、-dumpbase-ext、-dumpdir、
    -save-temps=cwd 和 -save-temps=obj 进行调整。

**-v -### --version** ::

    -v 在标准错误输出上打印编译阶段会执行的命令行。还会打印编译器驱动程序、预处理器和编
    译器本身的版本号。

    -### 与 -v 类似，但不执行命令，并且除非参数只包含字母数字字符或 ./-_，否则会对参数
    进行引用。这对于 shell 脚本捕获驱动程序生成的命令行很有用。

    --version 显示所调用的 GCC 的版本号和版权声明。

**--help --target-help --help={class|[^]qualifier}[,...]** ::

    --help 在标准输出上打印 gcc 理解的命令行选项的描述。如果同时指定了 -v 选项，那么
    --help 也会传递给 gcc 调用的各个阶段的进程，以便它们可以显示它们接受的命令行选项。
    如果还指定了-Wextra 选项（在 --help 选项之前），那么没有文档说明的命令行选项也会显
    示出来。

    --target-help 在标准输出上打印每个工具的特定目标平台相关的命令行选项的描述。对于某
    些目标平台，还可能打印额外的目标平台特定信息。

    --help={class|[^]qualifier}[,...] 在标准输出上打印编译器理解的符合所有指定类别和
    限定符的命令行选项的描述。支持的类别：
    ‘optimizers’
            显示编译器支持的所有优化选项。
    ‘warnings’
            显示控制编译器产生的警告消息的所有选项。
    ‘target’
            显示目标特定选项。然而与 --target-help 选项不同，链接器和汇编器的目标特定
            选项不会显示，因为这些工具目前不支持扩展的 --help= 语法。
    ‘params’
            显示 --param 选项识别的值。
    language
            显示特定语言支持的选项，其中 language 是此版本 GCC 支持的语言之一的名称。
            如果一个选项被所有语言支持，可以使用 ‘common’ 来显示。
    ‘common’
            显示所有语言共有的选项。
    支持的限定符：
    ‘undocumented’
            只显示未加文档说明的选项。
    ‘joined’
            显示那些在同一个连续文本片段中用等号分隔参数的选项，例如 --help=target。
    ‘separate’
            显示那些参数作为单独分隔的单词出现在选项之后的那些选项，例如
            -o output-file。

    因此，例如要显示编译器支持的所有未加文档说明的目标平台相关的选项，可以使用：
    --help=target,undocumented。限定符的含义可以通过在其前面加上 ^ 字符来反转，例如
    要显示所有有文档描述的那些要么开要么关且不带参数的选项，可以使用：
    --help=warnings,^joined,^undocumented。--help= 的参数不应仅由反转的限定符组成。
    组合多个类别是可以的，尽管这通常会限制输出，以至于没有什么可以显示的。然而，有一个
    情况是有效的，那就是其中一个类别是 target。例如要显示所有特定目标平台的优化选项，
    可以使用：--help=target,optimizers。--help= 选项可以在命令行上重复使用。每次连续
    使用都会显示其请求的选项类别，跳过那些已经显示过的。如果在命令行上的任何地方也指定
    了 --help，那么它将优先于任何 --help= 选项。

    如果在 --help= 选项之前在命令行上出现了 -Q 选项，那么 --help= 显示的描述性文本将
    被改变，它不会打印选项的描述信息，而是打印选项的启用、禁用、设值情况，假设编译器在
    对应 --help= 选项指定的位置知道这些信息。以下是从 ARM 版本的 gcc 截取的一个例子：
    % gcc -Q -mabi=2 --help=target -c
    The following options are target specific:
    -mabi=                      2
    -mabort-on-noreturn         [disabled]
    -mapcs                      [disabled]

    输出结果对 --help 选项前面提供的选项敏感，例如可以通过使用以下命令找出 -O2 具体启
    用了哪些优化：-Q -O2 --help=optimizers，或者你可以通过以下命令找出 -O3 启用的二
    进制优化：
    gcc -c -Q -O3 --help=optimizers > /tmp/O3-opts
    gcc -c -Q -O2 --help=optimizers > /tmp/O2-opts
    diff /tmp/O2-opts /tmp/O3-opts | grep enabled

**@file** ::

    从文件中读取命令行选项。读取的选项将替换原始的 @file 选项。如果文件不存在或不能读
    取，那么 @file 这个选项将被当作普通的文字处理，不会被移除。文件中的选项由空白字符
    分隔。可以通过将整个选项用单引号或双引号括起来，以便在选项中包含空白字符。通过在要
    包含的字符前面加上反斜杠，可以包含任何字符（包括反斜杠）。文件本身也可以包含额外的
    @file 选项；任何这样的选项都将被递归处理。

**-pass-exit-codes** ::

    通常，gcc 程序会在编译器任何阶段收到一个不成功的返回代码时，会以错误代码 1 作为退
    出码退出程序。如果指定了 -pass-exit-codes，gcc 程序将改为返回任何返回错误指示的阶
    段产生的数值最高的错误。C、C++ 和 Fortran 前端在遇到内部编译器错误时会返回 4。

**-pipe** ::

    在编译的各个阶段之间使用管道而不是临时文件进行通信。这在某些系统上无法工作，因为汇
    编器无法从管道中读取；但 GNU 汇编器没有这个问题。

**-specs=file** ::

    在编译器读取标准 specs 文件之后处理 file，以便覆盖 gcc 驱动程序的默认设定，这些设
    定用来确定需要将哪些选项传递给 cc1、cc1plus、as、ld 等各阶段的工具。可以在命令行上
    指定多个 -specs=file，并且它们将从左到右依次处理。有关文件格式的信息，参阅 Spec文
    件部分。spec 文件可以指定子进程及其传递的选项开关，gcc 是一个驱动程序，它通过调用
    一系列其他程序来完成编译、汇编和链接的工作。GCC 解释其命令行参数，并利用这些参数推
    断出它应该调用哪些程序，以及应该在它们的命令行上放置哪些命令行选项。这种行为由 spec
    字符串控制。在大多数情况下，GCC 可以调用的每个程序都有一个 spec 字符串，但少数程序
    有多个 spec 字符串来控制它们的行为。可以通过使用 -specs= 命令行开关指定一个 spec
    文件来覆盖 GCC 内置的 spec 字符串。Spec 文件是用于构建 spec 字符串的纯文本文件。
    它们由一系列由空白行分隔的指令组成，行首的第一个非空白字符决定了指令的类型。

**-wrapper** ::

    在包装程序下调用所有子命令。包装程序的名称及其参数作为逗号分隔的列表传递。例如：
    gcc -c t.c -wrapper gdb,--args 将在 ‘gdb --args’ 下调用 gcc 的所有子程序，因此
    cc1 通过 ‘gdb --args cc1 ...’ 的形式进行调用。

**-ffile-prefix-map=old=new** ::

    当编译目录 old 中文件时，将编译结果中任何引用这些文件的记录，修改为这些文件相当于是
    位于 new 目录中。指定此选项相当于指定了所有单独的 -f*-prefix-map 选项。这可以用于
    创建与位置无关的可重现构建。但是预处理指令（directives）中引用的目录不受这些选项的
    影响。还可以参阅 -fmacro-prefix-map、-fdebug-prefix-map、-fprofile-prefix-map
    和 -fcanon-prefix-map 选项。

**-fcanon-prefix-map** ::

    对于 -f*-prefix-map 选项，通常通过旧目录前缀与编译结果中引用的文件名来进行比较，或
    者在不区分大小写的文件系统上忽略字符大小写，并在基于 DOS 的文件系统上将斜杠和反斜杠
    视为相等。-fcanon-prefix-map 导致这样的 old目录和引用文件名的比较在规范路径上进
    行。

    规范化路径是指将一个文件或目录的路径转换为一种标准、唯一且明确的表示形式的结果。在
    不同的操作系统和文件系统中，同一个文件或目录可能有多种不同的路径表示方式，而规范化
    路径可以消除这些差异，确保每个文件或目录只有一个标准的路径表示。例如去除去除冗余的
    “.” 和 “..”，合并连续的斜杠 /home//user/pictures，解析符号链接等等。

**-fplugin=name.so** ::

    加载共享文件 name.so 中的插件代码，假定它是一个要被编译器通过 dlopen 加载的共享对
    象。共享对象文件的基本名称用于识别插件，以便匹配 -fplugin-arg-name-key=value 选
    项中为该插件提供的参数。每个插件都应该定义插件 API 中指定的回调函数。

**-fplugin-arg-<name>-<key>=value** ::

    为名为 name 的插件定义一个名为 key 值为 value 的参数。

**-dumpbase dumpbase** ::

    该选项设置辅助输出和转储输出文件的基本名称，它不影响主输出文件的名称。中间输出（如
    果保留）不被视为主输出，而是辅助输出：gcc -save-temps -S foo.c 将预处理后的临时文
    件保存为 foo.i，然后编译到隐含的输出文件 foo.s，而：gcc -save-temps
    -dumpbase save-foo -c foo.c 将预处理到 save-foo.i，编译到 save-foo.s（现在是中
    间的，因此是辅助输出），然后组装到隐含的输出文件 foo.o。

    如果没有此选项，转储和辅助文件的名称将从输入文件中获取，或者从显式指定的（非链接
    器）输出文件中获取：转储输出文件（例如通过 -fdump-* 选项请求的）带有输入名称后缀，
    而辅助输出文件（由其他非转储选项请求的，例如 -save-temps、-gsplit-dwarf、
    -fcallgraph-info）则没有。通过同时指定 -dumpbase-ext .suf 和 -dumpbase
    basename.suf，实现转储和辅助输出使用类似的后缀名。如果 dumpbase 显式指定了任何目
    录组件，则任何 dumppfx 的指定（例如通过 -dumpdir 或 -save-temps=*）将被忽略，
    dumpbase 完全覆盖它：gcc foo.c -c -o dir/foo.o -dumpbase alt/foo
    -dumpdir pfx- -save-temps=cwd ... 将创建名为 alt/foo.* 的辅助和转储输出，忽略
    -o 中的 dir/、-save-temps=cwd 暗示的 ./ 前缀以及 -dumpdir 中的 pfx-。

    当在编译多个输入文件或编译后链接的命令中指定 -dumpbase 时，它可以与 dumppfx 结合
    使用，如 -dumpdir 下所述。然后，每个输入文件都使用组合的 dumppfx 编译，并为每个输
    入文件计算默认的 dumpbase 和 auxdropsuf 值：
    gcc foo.c bar.c -c -dumpbase main ... 将创建主输出文件 foo.o 和 bar.o，并通过
    使用 dumpbase 作为前缀来避免覆盖辅助和转储输出，创建名为 main-foo.* 和
    main-bar.* 的辅助和转储输出。

    指定为空字符串的 dumpbase 可以避免在编译期间命名辅助和转储输出时受输出基本名称的影
    响，计算默认值：gcc -c foo.c -o dir/foobar.o -dumpbase '' ... 将命名辅助输出为
    dir/foo.*，转储输出为 dir/foo.c.*。请注意，它们的基本名称是从输入名称中获取的，但
    目录仍然默认为输出的目录。

    空字符串 dumpbase 不会阻止在链接期间为输出使用输出基本名称：
    gcc foo.c bar.c -o dir/foobar -dumpbase '' -flto ... 源文件的编译将命名辅助输
    出为 dir/foo.* 和 dir/bar.*，转储输出为 dir/foo.c.* 和 dir/bar.c.*。链接期间的
    LTO 重新编译将使用 dir/foobar. 作为转储和辅助文件的前缀。

**-dumpbase-ext auxdropsuf** ::

    在形成辅助输出文件（但不是转储输出文件）的名称时，从 dumpbase 中删除尾部的
    auxdropsuf，然后再追加任何后缀。如果没有指定此选项，则默认为默认 dumpbase 的后
    缀，即当命令行中未指定 -dumpbase 或 dumpbase 与 dumppfx 结合时，输入文件的后缀。

    gcc foo.c -c -o dir/foo.o -dumpbase x-foo.c -dumpbase-ext .c ... 将创建主输出
    文件 dir/foo.o，并生成名为 dir/x-foo.* 的辅助输出，取主输出的位置，并从 dumpbase
    中删除 .c 后缀。转储输出保留后缀：dir/x-foo.c.*。

    如果此选项与指定的 dumpbase 的后缀不匹配，则忽略此选项，但以下情况除外：作为将链接
    器输出基本名称追加到 dumppfx 的可执行文件后缀的替代方案，如下所述：
    gcc foo.c bar.c -o main.out -dumpbase-ext .out ...
    将创建主输出文件 main.out，并通过使用可执行文件名称减去 auxdropsuf 作为前缀来避免
    覆盖辅助和转储输出，创建名为 main-foo.* 和 main-bar.* 的辅助输出，以及名为
    main-foo.c.* 和 main-bar.c.* 的转储输出。

**-dumpdir dumppfx** ::

    在形成辅助或转储输出文件的名称时，使用 dumppfx 作为前缀：
    gcc -dumpdir pfx- -c foo.c ... 将创建主输出文件 foo.o，并创建名为 pfx-foo.* 的
    辅助输出，将给定的 dumppfx 与从输入名称派生的默认 dumpbase 结合，后者又从输入名称
    派生。转储输出也带有输入名称后缀：pfx-foo.c.*。如果 dumppfx 要用作目录名称，则必
    须以目录分隔符结尾：

    gcc -dumpdir dir/ -c foo.c -o obj/bar.o ... 将创建主输出文件 obj/bar.o，并创建
    名为 dir/bar.* 的辅助输出，将给定的 dumppfx 与从主输出名称派生的默认 dumpbase 结
    合。转储输出也带有输入名称后缀：dir/bar.c.*。它默认为输出文件的位置，除非输出文件
    是像 /dev/null 这样的特殊文件。选项 -save-temps=cwd 和 -save-temps=obj 覆盖此默
    认值，就像显式的 -dumpdir 选项一样。如果给出了多个这样的选项，则最后一个选项生效：

    gcc -dumpdir pfx- -c foo.c -save-temps=obj ... 输出 foo.o，辅助输出名为
    foo.*，因为 -save-temps=* 覆盖了前面 -dumpdir 给出的 dumppfx。这并不重要，因为
    -save-temps 的默认值是 =obj，并且输出目录隐式为当前目录。转储输出名为 foo.c.*。
    当从多个输入文件编译时，如果指定了 -dumpbase，则从 dumpbase 中减去 auxdropsuf 后
    缀和一个短横线，然后追加到（或覆盖，如果包含任何目录组件）显式或默认的 dumppfx，以
    便每个编译都有不同名称的辅助和转储输出。

    gcc foo.c bar.c -c -dumpdir dir/pfx- -dumpbase main ... 将辅助转储输出到
    dir/pfx-main-foo.* 和 dir/pfx-main-bar.*，将 dumpbase- 追加到 dumppfx。转储输
    出保留输入文件后缀：dir/pfx-main-foo.c.* 和 dir/pfx-main-bar.c.*。与单输入编译
    形成对比：

    gcc foo.c -c -dumpdir dir/pfx- -dumpbase main ... 将 dumpbase 应用于单个源文
    件，不会为每个输入文件计算和追加单独的 dumpbase。其辅助和转储输出位于
    dir/pfx-main.*。当从多个输入文件编译然后链接时，如果未指定 -dumpdir 或
    -dumpbase，则默认或显式指定的 dumppfx 也会经过上述 dumpbase- 转换（例如上面的
    foo.c 和 bar.c 的编译，但没有 -c）。如果未指定 -dumpdir 或 -dumpbase，则从链接器
    输出基本名称中减去 auxdropsuf（如果指定），或者否则为可执行文件后缀，然后加上一个
    短横线，追加到默认 dumppfx。注意，然而，与早期的链接案例不同：

    gcc foo.c bar.c -dumpdir dir/pfx- -o main ... 不会将输出名称 main 追加到
    dumppfx，因为 -dumpdir 是显式指定的。目标是显式指定的 dumppfx 可以包含作为前缀的
    一部分的指定输出名称（如果需要）；只有显式指定的 -dumpbase 会与它结合，以避免简单
    地丢弃一个有意义的选项。

    当从单个输入文件编译然后链接时，只有当链接器输出基本名称不与单个输入文件名称共享基
    本名称时，链接器输出基本名称才会像上面一样追加到默认 dumppfx。这在上面的单输入链接
    案例中已经涵盖，但没有显式的 -dumpdir，即使被 -save-temps=* 覆盖，也会抑制组合：
    gcc foo.c -dumpdir alt/pfx- -o dir/main.exe -save-temps=cwd ... 辅助输出名为
    foo.*，转储输出名为 foo.c.*，在当前工作目录中，最终由 -save-temps=cwd 请求。

    总结一下，虽然有些不精确，但为了直观起见：主输出名称被分解为目录部分和基本名称部
    分；dumppfx 被设置为前者，除非被 -dumpdir 或 -save-temps=* 覆盖，dumpbase 被设
    置为后者，除非被 -dumpbase 覆盖。如果有多个输入或链接，这个 dumpbase 可能会与
    dumppfx 结合，并从每个输入文件中获取。每个输入的辅助输出名称是通过组合 dumppfx、去
    掉后缀的 dumpbase 和辅助输出后缀形成的；转储输出名称只是在保留 dumpbase 的后缀方
    面有所不同。

    当涉及到 LTO 重新编译期间创建的辅助和转储输出时，无论是否在其他情况下使用这种组合，
    都会将一个组合的 dumppfx 和 dumpbase（如给定或从链接器输出名称派生的，而不是从输
    入派生的）传递给 lto-wrapper，作为 -dumpdir 选项，尾部的编译器添加的短横线（如果
    有）被替换为句点；由于涉及链接，该程序通常不会得到任何 -dumpbase 和
    -dumpbase-ext，并且会忽略它们。当运行子编译器时，lto-wrapper 将 LTO 阶段名称追加
    到接收到的 dumppfx，确保它包含一个目录组件，以便它覆盖任何 -dumpdir，并将该内容作
    为 -dumpbase 传递给子编译器。

C 特定选项
-----------

特定于 C 语言的选项汇总： ::

    -ansi -std=standard -aux-info filename
    -fno-asm -fno-builtin -fno-builtin-function
    -ffreestanding -fhosted
    -fcond-mismatch -fgimple -fgnu-tm -fgnu89-inline
    -flax-vector-conversions -fms-extensions -fplan9-extensions
    -fpermitted-flt-eval-methods=standard
    -fdeps-file=file -fdeps-format=format -fdeps-target=file
    -fsigned-bitfields -funsigned-bitfields
    -fsigned-char -funsigned-char -fstrict-flex-arrays[=n]
    -fsso-struct=endianness
    -foffload=arg -foffload-options=arg
    -fopenacc -fopenacc-dim=geom
    -fopenmp -fopenmp-simd -fopenmp-target-simd-clone[=device-type]

这些选项编译器接受的特定于 C 语言部分的选项，这包括从 C 语言派生的语言例如 C++、
Object-C、Object-C++ 所共用的选项。

**-ansi** ::

    在 C 模式下，这相当于 -std=c90。在 C++ 模式下，它相当于 -std=c++98。此选项关闭了
    GCC 中与 ISO C90（编译C代码时）或标准 C++（编译C++代码时）不兼容的某些特性，例如
    asm 和 typeof 关键字，以及标识你所使用的系统类型的预定义宏，如 unix 和 vax。它还
    启用了不受欢迎且很少使用的 ISO 三字符替代功能。对于 C 编译器，它禁用了对 C++ 风格
    的 ‘//’ 注释以及 inline 关键字的识别。

    备用关键字 __asm__、 __extension__、 __inline__ 和 __typeof__ 尽管有 -ansi 选
    项，但仍然可以使用。当然你不会在 ISO C 标准程序中使用它们，但在可能使用 -ansi 选项
    进行编译的头文件中包含这些关键字还是很有用的。备用预定义宏，如 __unix__ 和 __vax，
    无论是否指定 -ansi 都可以使用。-ansi 选项不会无端拒绝非 ISO 程序。为此，除了
    -ansi 之外，还需要 -Wpedantic，参见警告选项部分。

    使用 -ansi 选项时，会预定义 __STRICT_ANSI__ 宏。某些头文件可能会注意到这个宏，并
    避免声明某些 ISO 标准未要求的函数或定义某些宏，以免干扰可能将这些名称用作其他用途的
    程序。通常作为内置函数但其语义未被 ISO C 定义的函数（如 alloca 和 ffs）在使用
    -ansi时不是内置函数（builtin functions）。有关受影响函数的详细信息，参阅由 GCC
    提供的其他内置函数部分。

**-std=standard** ::

    确定语言标准，此选项目前仅当编译 C 或 C++ 时使用。编译器可以接受几种基础标准，如
    c90 或 c++98，以及这些标准的 GNU 版本，如 gnu90 或 gnu++98。当指定了一个基础标准
    时，编译器接受遵循该标准的所有程序以及那些不与之冲突的 GNU 扩展。例如 -std=c90 关
    闭了与 ISO C90 不兼容的 GCC 的某些特性，如 asm 和 typeof 关键字，但没有关闭在
    ISO C90 中没有意义的其他 GNU 扩展，如省略 ?: 表达式的中间项。另一方面，当指定了一
    个标准的 GNU 版本时，即使这些特性改变了基础标准的含义，编译器支持的所有特性都被启
    用。因此，一些严格符合标准的程序可能会被拒绝。特定的标准被 -Wpedantic 用来识别在该
    标准版本下哪些特性是 GNU 扩展。例如 -std=gnu90 -Wpedantic 会警告 C++ 风格的 //
    注释，而 -std=gnu99 -Wpedantic 则不会。此选项必须提供一个值，可以使用的值包括：

    c90 c89 iso9899:1990
        支持所有 ISO C90 程序（某些与 ISO C90 冲突的 GNU 扩展被禁用）。对于 C 代码与
        -ansi 相同。
    iso9899:199409
        包含 ISO C90 在修正案 1 中的修改。
    c99 c9x iso9899:1999 iso9899:199x
        ISO C99，此标准基本完全支持，更多信息请参阅
        https://gcc.gnu.org/c99status.html。c9x 和 iso9899:199x 的名称已被弃用。
    c11 c1x iso9899:2011
        ISO C11，ISO C 标准的 2011 年修订版。c1x 的名称已被弃用。
    c17 c18 iso9899:2017 iso9899:2018
        ISO C17，ISO C 标准的 2017 年修订版（2018年发布）。此标准与 C11 相同，除了修
        正了缺陷（所有这些修正也应用于 -std=c11）和一个新的 __STDC_VERSION__ 值，因
        此支持程度与 C11 相同。
    c23 c2x iso9899:2024
        ISO C23，ISO C 标准的 2023 年修订版（预计2024年发布）。对这个版本的支持是实
        验性的且不完整的。c2x 的名称已被弃用。
    gnu90 gnu89
        ISO C90 的 GNU 版本（包括一些 C99 特性）。
    gnu99 gnu9x
        ISO C99 的 GNU 版本。gnu9x 的名称已被弃用。
    gnu11 gnu1x
        ISO C11 的 GNU 版本。gnu1x 的名称已被弃用。
    gnu17 gnu18
        ISO C17 的 GNU 版本。这是 C 代码的默认值。
    gnu23 gnu2x
        ISO C 标准的下一个版本，仍在开发中，加上 GNU 扩展。对这个版本的支持是实验性的
        且不完整的。gnu2x 的名称已被弃用。
    c++98 c++03
        1998 年 ISO C++ 标准加上 2003 年技术勘误和一些额外的缺陷报告。对于 C++ 代
        码，与 -ansi 相同。
    gnu++98 gnu++03
        -std=c++98 的 GNU 版本。
    c++11 c++0x
        2011 年 ISO C++ 标准加上修正案。c++0x 的名称已被弃用。
    gnu++11 gnu++0x
        -std=c++11 的 GNU 版本。gnu++0x 的名称已被弃用。
    c++14 c++1y
        2014 年 ISO C++ 标准加上修正案。c++1y 的名称已被弃用。
    gnu++14 gnu++1y
        -std=c++14 的 GNU 版本。gnu++1y 的名称已被弃用。
    c++17 c++1z
        2017 年 ISO C++ 标准加上修正案。c++1z 的名称已被弃用。
    gnu++17 gnu++1z
        -std=c++17 的 GNU 版本。这是 C++ 代码的默认值。gnu++1z 的名称已被弃用。
    c++20 c++2a
        2020 年 ISO C++ 标准加上修正案。支持是实验性的，未来版本可能会以不兼容的方式
        变化。c++2a 的名称已被弃用。
    gnu++20 gnu++2a
        -std=c++20 的 GNU 版本。支持是实验性的，未来版本可能会以不兼容的方式变化。
        gnu++2a 的名称已被弃用。
    c++23 c++2b
        ISO C++ 标准的下一个修订版，计划于 2023 年发布。支持是非常实验性的，未来版本
        几乎肯定会以不兼容的方式变化。
    gnu++23 gnu++2b 
        -std=c++2b 的 GNU 版本。支持是非常实验性的，未来版本几乎肯定会以不兼容的方式
        变化。
    c++26 c++2c 
        ISO C++ 标准的下一个修订版，计划于 2026 年发布。支持是非常实验性的，未来版本
        几乎肯定会以不兼容的方式变化。
    gnu++26 gnu++2c 
        -std=c++2c 的 GNU 版本。支持是非常实验性的，未来版本几乎肯定会以不兼容的方式
        变化。

**-aux-info filename** ::

    将一个编译单元中声明以及定义的所有函数的原型输出到给定文件名中，包括头文件中的函
    数。一个编译单元就是把源文件和它所包含的头文件内容合并在一起后形成的一个完整的文本
    块。此选项在 C 语言以外的任何语言中都会被静默忽略。除了声明之外，文件还会在注释中指
    示每个声明的来源（源文件和行号），声明是隐式的、有原型的还是无原型的（在行号和冒号
    后的第一个字符分别用 ‘I’、‘N’ 新的或 ‘O’ 旧的来表示），以及它来自声明还是定义（在
    随后的字符中分别用 ‘C’ 或 ‘F’ 表示）。对于函数定义，还会在注释中提供一个 K&R 风格
    的参数列表及其声明，紧跟在声明之后。

**-fno-asm** ::

    不把 asm、inline 或 typeof 识别为关键字，以便代码可以将这些词用作标识符。你可以改
    用关键字 __asm__、 __inline__ 和 __typeof__。在 C 语言中，-ansi 隐含了
    -fno-asm 选项。在 C++ 中，inline 是一个标准关键字，不受此开关的影响。你可能想改用
    -fno-gnu-keywords 标志，它禁用了 typeof，但不禁用 asm 和 inline。在 C99 模式
    （-std=c99 或 -std=gnu99）下，此开关只影响 asm 和 typeof 关键字，因为 inline 是
    ISO C99 中的标准关键字。在 C23 模式（-std=c23 或 -std=gnu23）下，此开关只影响
    asm 关键字，因为 typeof 是 ISO C23 中的标准关键字。

**-fno-builtin -fno-builtin-<function>** ::

    不识别不以 __builtin_ 为前缀的内置函数（builtin functions）。关于受影响的函数的
    详细信息，参阅由 GCC 提供的其他内置函数部分，包括那些当使用 -ansi 或 -std 选项以
    严格遵循 ISO C 标准时不是内置函数的函数，因为它们在 ISO 标准下没有意义。GCC 通常会
    生成特殊的代码，以更高效地处理某些内置函数；例如对 alloca 的调用可能会变成直接调整
    堆栈的单条指令，对 memcpy 的调用可能会变成内联复制循环。生成的代码通常既小又快，但
    由于函数调用不再以调用的形式出现，因此你无法在这些调用上设置断点，也无法通过链接不
    同的库来改变这些函数的行为。此外，当一个函数被识别为内置函数时，GCC 可能会使用有关
    该函数的信息来警告对该函数的调用问题，或者生成更高效的代码，即使生成的代码仍然包含
    对该函数的调用。例如，当 printf 是内置函数且 strlen 已知不会修改全局内存时，会为对
    printf 的不良调用发出警告（使用-Wformat）。使用 -fno-builtin-function 选项时，
    仅禁用名为 function 的内置函数。function 不能以 ‘__builtin_’ 开头。如果命名的函
    数在 GCC 的这个版本中不是内置函数，则忽略此选项。没有对应的 -fbuiltin-function 选
    项；如果你想在使用 -fno-builtin 或 -ffreestanding 时选择性地启用内置函数，你可以
    定义宏，例如：

    #define abs(n) __builtin_abs ((n))
    #define strcpy(d, s) __builtin_strcpy ((d), (s))

**-ffreestanding** ::

    声明编译目标是一个独立环境，该选项暗含了 -fno-builtin。独立环境是一个标准库可能不
    存在，程序启动不一定在 main 的环境。最明显的例子是操作系统内核。这相当于
    -fno-hosted。

**-fhosted** ::

    声明编译目标是一个托管环境。该选项暗含了 -fbuiltin。托管环境是一个提供整个标准库，
    且 main 的返回类型为 int 的环境。除了内核之外，几乎所有环境都是托管环境。这相当于
    -fno-freestanding。

**-fcond-mismatch** ::

    允许条件表达式的第二个和第三个参数类型不匹配。这种表达式的值是 void。此选项不支持
    C++。

**-fgimple** ::

    启用对标记为 __GIMPLE 的函数定义的解析。这是一个实验性功能，允许对 GIMPLE 传递进
    行单元测试。

**-fgnu-tm** ::

    当指定 -fgnu-tm 选项时，编译器会为 Intel 当前事务内存 ABI 规范文档（修订版 1.1，
    2009 年 5 月 6 日）的 Linux 变体生成代码。这是一个实验性功能，其接口可能会随着
    GCC 未来版本的变化而变化，因为官方规范可能会发生变化。请注意，并非所有架构都支持此
    功能。有关 GCC 对事务内存的支持的更多信息，请参阅 GNU 事务内存库部分。请注意，事务
    内存功能不支持非调用异常 -fnon-call-exceptions。

**-fgnu89-inline** ::

    -fgnu89-inline 选项告诉 GCC 在 C99 模式下使用传统的 GNU 语义处理内联函数。参见内
    联函数与宏一样快部分。使用此选项大致相当于为所有内联函数添加 gnu_inline 函数属性。
    -fno-gnu89-inline 选项明确告诉 GCC 在 C99 或 gnu99 模式下使用 C99 语义处理内联
    函数（默认行为）。此选项在 -std=c90 或 -std=gnu90 模式下不支持。可以使用预处理器
    宏 __GNUC_GNU_INLINE__ 和 __GNUC_STDC_INLINE__ 来检查当前生效的内联函数语义。参
    见常用预定义宏部分。

**-flax-vector-conversions** ::

    允许在元素数量不同以及元素类型不兼容的向量之间进行隐式转换。不应在新代码中使用此选
    项。

**-fms-extensions** ::

    接受 Microsoft 头文件中使用的一些非标准构造。在 C++ 代码中，这允许结构体中的成员名
    称与其类型名称相同：
    typedef int UOW;
    struct ABC {
        UOW UOW;
    };
    某些情况下，结构体和联合体中的未命名字段只有在此选项下才会被接受。详细信息请参阅结
    构体和联合体中的未命名结构体和联合体字段部分。请注意，此选项仅在使用 ms-abi 的 x86 
    目标平台上默认启用，其他目标平台默认关闭。

**-fplan9-extensions** ::

    接受 Plan 9 代码中使用的一些非标准构造。该选项启用了 -fms-extensions，允许将具有
    匿名字段的结构体的指针传递给期望该字段类型元素指针的函数，并允许引用使用 typedef
    声明的匿名字段。有关详细信息，请参阅结构体和联合体中的未命名结构体和联合体字段部
    分。此选项仅支持 C，不支持 C++。

**-fpermitted-flt-eval-methods=style** ::

    ISO/IEC TS 18661-3 定义了 FLT_EVAL_METHOD 的新允许值，这些值表明具有交换格式或扩
    展格式的语义类型的运算和常量应按该类型的精度和范围进行评估。这些新值是 C99/C11 允
    许值的超集，C99/C11 没有指定 FLT_EVAL_METHOD 中其他正数值的含义。因此，符合 C11
    标准的代码可能没有预料到新值的可能性。

    -fpermitted-flt-eval-methods 指定编译器是否只允许 C99/C11 中指定的
    FLT_EVAL_METHOD 值，还是允许 ISO/IEC TS 18661-3 中指定的扩展值集。style 可以是
    c11 或 ts-18661-3，视情况而定。在符合标准的模式下（-std=c11 或类似），默认值是
    -fpermitted-flt-eval-methods=c11。在 GNU 版本模式下（-std=gnu11 或类似），默认
    值是 -fpermitted-flt-eval-methods=ts-18661-3。

**-fdeps-file=file -fdeps-format=format -fdeps-target=file** ::

    -fdeps-* 选项用于提取源文件的结构化依赖信息。这包括确定编译源文件提供的资源，以及
    其所需的其他源文件提供的资源。这些信息仅根据源文件的内容就可以给对应的独立源文件的
    编译规则添加所需的依赖关系，而无需将这些信息同时反映到构建工具中。-fdeps-file=file
    将结构化依赖信息写到何处。-fdeps-format=format 用于指定结构化依赖信息的格式。目前
    唯一支持的格式是 p1689r5。

    请注意，当指定此参数时，-MF 的输出会剥离一些信息，特别是 C++ 模块（modules），以便
    它不会使用大多数工具无法理解的扩展 makefile 语法。

    -fdeps-target=file 类似于 -MT，但用于结构化依赖信息。这表明目标最终需要的任何所需
    资源，并提供从源文件中提取的可能被其他源文件所需的资源。

**-f[un]signed-bitfields -fno-[un]signed-bitfields** ::

    这些选项控制在声明中未使用 signed 或 unsigned 时，位域是有符号的还是无符号的。默
    认情况下，这样的位域字段是有符号的，因为这是一致的：基本整数类型（如 int）是有符号
    类型。

**-fsigned-char -funsigned-char** ::

    -fsigned-char 让 char 类型是有符号的，就像 signed char 一样。注意，这相当于
    -fno-unsigned-char，即 -funsigned-char 的否定形式。同样，选项 -fno-signed-char
    相当于 -funsigned-char。-funsigned-char 让 char 类型是无符号的，就像 unsigned
    char 一样。

    每种机器都有 char 的默认类型。它默认是像 unsigned char 一样，或者像 signed char
    一样。理想情况下，可移植程序在依赖对象的符号时，应始终使用 signed char 或
    unsigned char。但许多程序只使用普通 char 类型，并期望它是有符号的或者根据它们所在
    的机器期望它是无符号的。此选项及其反向选项，可以让这样的程序与其相反的默认值一起工
    作。

    char 类型始终是与 signed char 或 unsigned char 类型是不同的类型，尽管它的行为总
    是与这两个类型中的一个相同。

**-fstrict-flex-arrays -fstrict-flex-arrays=level（仅限 C 和 C++）** ::

    控制何时将结构体的尾随数组（trailing array）视为灵活数组成员（flexible array），
    以便访问此类数组的元素。level 的值控制严格性级别。-fstrict-flex-arrays 相当于
    -fstrict-flex-arrays=3，这是最严格的，所有结构体的尾随数组都被视为灵活数组成员。
    否定形式 -fno-strict-flex-arrays 相当于 -fstrict-flex-arrays=0，这是最不严格
    的。在这种情况下，只有当尾随数组被声明为 C99 标准及以后的灵活数组成员时，才将其视为
    灵活数组成员。level 的可能值与 strict_flex_array 属性的值相同。你可以通过使用变量
    属性 strict_flex_array 来控制结构体的特定尾随数组字段的这种行为。
    -fstrict-flex-arrays 选项与 -Wstrict-flex-arrays 选项相互作用。更多信息请参阅警
    告选项部分。

**-fsso-struct=endianness** ::

    将结构体和联合体的默认标量存储顺序（scalar storage order）设置为指定的字节序。可
    接受的值为 ‘big-endian’、‘little-endian’ 和 ‘native’（目标的本地字节序，默认
    值）。此选项不支持 C++。警告：如果指定的字节序不是目标平台本地字节序，-fsso-struct
    选项会导致 GCC 生成的代码与没有该选项时生成的代码不二进制兼容。

C++ 特定选项
------------

特定于 C++ 语言的选项汇总： ::

    -stdlib=libstdc++,libc++
    -fabi-version=n -fno-access-control
    -faligned-new=n -fargs-in-order=n -fchar8_t -fcheck-new
    -fconstexpr-depth=n -fconstexpr-cache-depth=n
    -fconstexpr-loop-limit=n -fconstexpr-ops-limit=n
    -fno-elide-constructors
    -fno-enforce-eh-specs
    -fno-gnu-keywords
    -fno-immediate-escalation
    -fno-implicit-templates
    -fno-implicit-inline-templates
    -fno-implement-inlines
    -fmodule-header[=kind] -fmodule-only -fmodules-ts
    -fmodule-implicit-inline
    -fno-module-lazy
    -fmodule-mapper=specification
    -fmodule-version-ignore
    -fms-extensions
    -fnew-inheriting-ctors
    -fnew-ttp-matching
    -fno-nonansi-builtins -fnothrow-opt -fno-operator-names
    -fno-optional-diags
    -fno-pretty-templates
    -fno-rtti -fsized-deallocation
    -ftemplate-backtrace-limit=n
    -ftemplate-depth=n
    -fno-threadsafe-statics -fuse-cxa-atexit
    -fno-weak -nostdinc++
    -fvisibility-inlines-hidden
    -fvisibility-ms-compat
    -fext-numeric-literals
    -flang-info-include-translate[=header]
    -flang-info-include-translate-not
    -flang-info-module-cmi[=module]
    -Wabi-tag -Wcatch-value -Wcatch-value=n
    -Wno-class-conversion -Wclass-memaccess
    -Wcomma-subscript -Wconditionally-supported
    -Wno-conversion-null -Wctad-maybe-unsupported
    -Wctor-dtor-privacy -Wdangling-reference
    -Wno-delete-incomplete
    -Wdelete-non-virtual-dtor -Wno-deprecated-array-compare
    -Wdeprecated-copy -Wdeprecated-copy-dtor
    -Wno-deprecated-enum-enum-conversion -Wno-deprecated-enum-float-conversion
    -Weffc++ -Wno-elaborated-enum-base
    -Wno-exceptions -Wextra-semi -Wno-global-module -Wno-inaccessible-base
    -Wno-inherited-variadic-ctor -Wno-init-list-lifetime
    -Winvalid-constexpr -Winvalid-imported-macros
    -Wno-invalid-offsetof -Wno-literal-suffix
    -Wmismatched-new-delete -Wmismatched-tags
    -Wmultiple-inheritance -Wnamespaces -Wnarrowing
    -Wnoexcept -Wnoexcept-type -Wnon-virtual-dtor
    -Wpessimizing-move -Wno-placement-new -Wplacement-new=n
    -Wrange-loop-construct -Wredundant-move -Wredundant-tags
    -Wreorder -Wregister
    -Wstrict-null-sentinel -Wno-subobject-linkage -Wtemplates
    -Wno-non-template-friend -Wold-style-cast
    -Woverloaded-virtual -Wno-pmf-conversions -Wself-move -Wsign-promo
    -Wsized-deallocation -Wsuggest-final-methods
    -Wsuggest-final-types -Wsuggest-override -Wno-template-id-cdtor
    -Wno-terminate -Wno-vexing-parse -Wvirtual-inheritance
    -Wno-virtual-move-assign -Wvolatile -Wzero-as-null-pointer-constant

**编译 C++ 程序**

C++ 源文件通常使用以下后缀之一：.C、.cc、.cpp、.CPP、.c++、.cp 或 .cxx；C++ 头文件通
常使用 .hh、.hpp、.H，或者（对于共享模板代码）.tcc；预处理后的 C++文件使用 .ii 后缀。
即使你以编译 C 程序的方式调用编译器（通常使用gcc名称），GCC 也能自动识别这些文件的后缀
名并将它们作为 C++ 程序进行编译。

然而，使用 gcc 并不会添加 C++ 库。g++ 是一个调用 GCC 并自动指定链接 C++ 库的程序。它
将 .c、.h 和 .i 文件视为 C++ 源文件，而不是 C 源文件，除非使用了 -x 选项。g++ 也可以
用来在 C++ 的编译过程中预编译带有 .h 扩展名的 C 头文件。在许多系统上，g++ 还以 c++ 的
名称进行了安装。

当你编译 C++ 程序时，你可以指定许多与编译任何语言程序时相同的命令行选项；或者对 C 和相
关语言有意义的命令行选项；或者仅对 C++ 程序有意义的选项。一些用于编译 C 程序的选项，如
-std，也适用于 C++ 程序。关于与 C 语言相关的选项的解释，参考前面的 C 特定选项部分。

这里描述了仅用于编译 C++ 语言程序的选项。

**-stdlib=libstdc++,libc++** ::

    当 g++ 配置了支持此选项时，它允许指定替代的 C++ 运行时库。有两种选项可供选择：
    libstdc++（g++ 默认、原生 C++ 运行时）和 libc++（在某些操作系统上安装的 C++ 运行
    时，例如从 Darwin11 开始的 Darwin 版本）。此选项使 g++ 切换到使用指定库的头文件，
    并在链接需要的 C++ 运行时库时分别使用 -lstdc++ 或 -lc++。

预处理选项
----------

预处理选项汇总： ::

    -Dmacro[=defn] -Umacro -undef -pthread
    -imacros file -include file
    -M -MD -MM -MMD -MF -MG -MP -MT -MQ -Mno-modules
    -dD -dI -dM -dN -dU
    -C -CC -H -P -Aquestion=answer -A-question[=answer]
    -traditional -traditional-cpp -trigraphs
    -Wp,option -Xpreprocessor option
    -no-integrated-cpp -remap
    -fdebug-cpp -fdirectives-only -fdollars-in-identifiers
    -fexec-charset=charset -fextended-identifiers
    -finput-charset=charset -flarge-source-files
    -fmacro-prefix-map=old=new -fmax-include-depth=depth
    -fno-canonical-system-headers -fpch-deps -fpch-preprocess
    -fpreprocessed -ftabstop=width -ftrack-macro-expansion
    -fwide-exec-charset=charset -fworking-directory

这些选项控制 C 预处理器，它在实际编译之前对每个 C 源文件进行处理。如果你使用了 -E 选
项，除了预处理之外它不会执行任何操作。这些选项中的一些只有与 -E 一起使用才有意义，因为
它们会使预处理器的输出不适用于实际编译阶段。

**-D name -D name=definition** ::

    -D name 定义一个值为 1 的宏。-D name=definition 定义一个值为 definition 的宏，
    如果你是从 shell 或类似 shell 的程序中调用预处理器，你可能需要使用 shell 的引号语
    法来保护在 shell 语法中有意义的字符，比如空格。如果你想在命令行上定义一个类似函数
    的宏，应该在等号之前，用括号将它的参数列表包起来。括号对大多数 shell 来说都有意
    义，所以你应该对选项进行引用。在 sh 和 csh 中，-D'name(args...)=definition' 是
    有效的。

    -D 和 -U 选项按照它们在命令行上出现的顺序进行处理。所有 -imacros file 和
    -include file 选项都在所有 -D 和 -U 选项之后处理。而所有的 -include file 都在所
    有 -imacros file 之后处理。

**-U name** ::

    取消 name 的任何先前定义，无论是内置的还是通过 -D 选项提供的。

**-undef** ::

    不要预定义任何系统特定或 GCC 特定的宏。标准预定义的宏仍然被定义。

**-pthread** ::

    定义使用 POSIX 线程库所需的额外宏。你应该在编译和链接时一致地使用这个选项。这个选
    项在 GNU/Linux 目标平台、大多数其他 Unix 衍生系统以及 x86 Cygwin 和 MinGW 目标平
    台上得到支持。

**-imacros file** ::

    与 -include 完全相同，只是扫描 file 产生的任何输出都被丢弃。它定义的宏仍然保持定
    义。这允许你获取一个头文件中的所有宏，而无需处理它的声明。所有由 -imacros 指定的文
    件都在由 -include 指定的所有文件之前处理。

**-include file** ::

    处理 file，就像在主源文件的第一行出现了 #include "file" 一样。然而，搜索 file 的
    第一个目录是预处理器的工作目录，而不是包含主源文件的目录。如果在那里没有找到，它将
    按照正常的 #include "..." 搜索链在其余目录中搜索。如果给出了多个 -include 选项，
    文件将按照它们在命令行上出现的顺序被包含。

**-M** ::

    不输出预处理的结果，而是输出一个适合 make 的规则，描述主源文件的依赖关系。预处理器
    输出一个 make 规则，包含该源文件的对象文件名、一个冒号以及所有被包含文件的名称，包
    括那些来自命令行选项 -include 或 -imacros 的文件。

    除非明确指定（使用 -MT 或 -MQ ），对象文件名由源文件名组成，将任何后缀替换为对象文
    件后缀，并移除任何前导目录部分。如果包含的文件很多，使用‘\’-换行符将规则分割成多
    行，规则只包含依赖文件没有命令。

    此选项不会抑制预处理器的调试输出，例如 -dM。为了避免将这样的调试输出与依赖规则混
    合，你应该明确指定依赖输出文件（使用 -MF），或者使用环境变量，如
    DEPENDENCIES_OUTPUT，参见环境变量部分。调试输出仍然像往常一样发送到常规输出流。

    将 -M 传递给驱动程序意味着隐含 -E 选项，并通过隐含的 -w 选项抑制警告。

**-MM** ::

    与 -M 类似，但不提及系统头文件目录中找到的头文件，也不提及直接或间接从这样的头文件
    中包含的头文件。这意味着在 ‘#include’ 指令中选择尖括号或双引号本身不会决定该头文件
    是否出现在-MM依赖输出中。

**-MD** ::

    -MD 等同于 -M -MF file，但不隐含 -E。驱动程序根据是否指定了 -o 选项来确定 file。
    如果指定了，驱动程序使用它的参数，但将其后缀改为 .d；否则，它取输入文件的名称，移除
    任何目录组件和后缀，并应用一个 .d 后缀。如果 -MD 与 -E 一起使用，任何 -o 选项都被
    理解为指定依赖输出文件，但如果在没有 -E 的情况下使用，每个 -o 都被理解为指定一个目
    标对象文件。由于没有隐含 -E，-MD 可以用来在编译过程中附带生成一个依赖输出文件。

**-MMD** ::

    与 -MD 类似，但只提及用户头文件，不提及系统头文件。

**-MF file** ::

    当与 -M 或 -MM 一起使用时，指定一个文件来写入依赖关系。如果没有给出 -MF 开关，预处
    理器将规则发送到它将发送预处理输出的同一位置。当与驱动程序选项 -MD 或 -MMD 一起使
    用时，-MF 覆盖默认的依赖输出文件。如果 file 是 -，则依赖关系被写入 stdout。

**-MG** ::

    与请求生成依赖关系的选项（如 -M）一起使用时，-MG 假设缺失的头文件是生成的文件，并
    将它们添加到依赖列表中，而不引发错误。依赖文件名直接从 #include 指令中取出，不添加
    任何路径。-MG 还抑制预处理输出，因为缺失的头文件使预处理输出变得无用。此功能用于
    makefile 的自动更新。

**-MP** ::

    此选项指示 CPP 为每个依赖项（除了主文件）添加一个伪目标（phony target），使每个依
    赖项依赖空文件。这些假规则可以解决如果你移除头文件而没有同步更新 Makefile 导致
    make 报错的问题。例如上次 test.c 文件中包含了 test.h 文件，但在当前的这次修改中将
    包含的 test.h 文件移除了，当再次编译时不应该报 test.h 找不到的错误，而是应该继续编
    译并生成新的依赖文件：

    test.o: test.c test.h
    test.h:

**-MT target** ::

    更改依赖关系规则的目标名称。默认情况下，CPP 取主输入文件的名称，删除任何目录组件和
    任何文件后缀（如 ‘.c’），并追加平台通常的对象文件后缀，结果就是目标。-MT 选项将目
    标设置为你指定的确切字符串。如果你想有多个目标，你可以将它们作为单个参数传递给
    -MT，或者使用多个 -MT 选项。例如，-MT '$(objpfx)foo.o' 可能会输出：
    $(objpfx)foo.o: foo.c

**-MQ target** ::

    与 -MT 相同，但它会引用对 Make 来说特殊的任何字符。例如 -MQ '$(objpfx)foo.o' 输
    出：$$(objpfx)foo.o: foo.c。默认目标会自动被引用，就像它是用 -MQ 给出的一样。

**-Mno-modules** ::

    禁止为已经编译的模块接口（compiled module interfaces）生成依赖关系文件。

**-dletters** ::

    指定在编译期间根据 letters 来生成 debugging dump 信息。这里记录的是与预处理器相关
    的标志，其他字母由编译器本身解释，或为 GCC 的未来版本保留，因此会被静默忽略。如果指
    定的 letters 行为冲突，结果是未定义的。更多信息参阅开发者选项部分。

    -dM 不输出正常内容，而是为预处理器执行过程中定义的所有宏（包括预定义宏）生成
    ‘#define’ 指令列表。这为你提供了一种找出预处理器版本中预定义内容的方法。假设你没有
    文件 foo.h，命令 touch foo.h; cpp -dM foo.h 会显示所有预定义的宏。如果在没有 -E
    选项的情况下使用 -dM，-dM 被解释为 -fdump-rtl-mach 的同义词，参见开发者选项部分。

    -dD 与 -dM 类似，但它输出 ‘#define’ 指令以及预处理的结果。这两种输出都发送到标准
    输出文件。

    -dN 与 -dD 类似，但只输出宏名称，而不是它们的展开内容。

    -dI 除了预处理的结果外，还输出 ‘#include’ 指令。

    -dU 与 -dD 类似，但只有被展开或在预处理器指令中测试其定义性的宏才会被输出；输出会
    延迟到宏的使用或测试时；并且对于在测试时未定义的宏，也会输出 ‘#undef’ 指令。

**-C** ::

    不丢弃注释。所有注释都会被传递到输出文件中，除了已经处理过的预处理指令行之外，这些
    注释会连同指令一起被删除。使用 -C 时，你需要应对它的副作用，它会导致预处理器将注释
    视为正常的语法标记，而不是像原理的语义一样将注释当作空白移除。例如，出现在预处理指
    令行开头的注释会使该行变成普通源代码行，因为行的第一个标记不再是 ‘#’。例如：
    // 保留的注释
    #include <stdio.h> // 该行处理完之后是将 stdio.h 包含进来，这个注释也随之移除
    /* 注释不被当作空白了，导致影响对预处理指令的正常解析 */ #define SIZE 16

**-CC** ::

    不丢弃注释，包括在宏展开期间。这与 -C 类似，但宏中的注释也会被传递到宏展开的输出文
    件中。除了 -C 选项的副作用外，-CC 选项还会将宏中所有的 C++ 风格注释转换为 C 风格注
    释。这是为了防止后来使用该宏时不慎注释掉后面的源代码行。-CC 选项通常用于支持 lint
    注释。

**-H** ::

    除了其他正常活动外，打印每个使用的头文件名称。每个名称都会缩进以显示它在
    ‘#include’ 堆栈中的深度。预编译的头文件也会被打印出来，即使它们被发现是无效的；无
    效的预编译头文件会用 ‘...x’ 打印，有效的则用 ‘...!’ 打印。

**-P** ::

    禁止在预处理的输出文件中生成行标记，当在非 C 代码上运行预处理时可能有用，因为这些行
    标记可能给对应的程序代码造成混淆。

**-A predicate=answer -A -predicate[=answer]** ::

    使用谓词 predicate 和答案 answer 进行断言。因为它不使用 shell 特殊字符，这种形式
    优于旧形式 -A predicate(answer)，但后者仍然被支持。-A -predicate=answer 取消使
    用谓词 predicate 和答案 answer 的断言。可以在代码中使用
    __has_predicate(predicate)、__predicate(predicate) == 1 等形式运用这些断言。

**-traditional -traditional-cpp** ::

    尝试模仿非标准 C 预处理器的行为，而不是 ISO C 预处理器的行为。详细信息请参阅 GNU
    CPP 手册。注意 GCC 并不试图模拟非标准 C 编译器的其他行为，这些选项仅在使用 -E 时或
    显式调用 CPP 时支持。

**-trigraphs** ::

    支持 ISO C 三字符替代。这些是以 ‘??’ 开头的三个字符序列，ISO C 定义它们代表单个字
    符。例如，‘??/’ 代表 ‘\’，因此 '??/n' 是一个换行符的字符常量。九个三字符替代及其
    替换定义如下：
    Trigraph:     ??(   ??)   ??<   ??>   ??=   ??/   ??'   ??!   ??-
    Replacement:  [     ]     {     }     #     \     ^     |     ~
    默认情况下，GCC 忽略三字符替代，但在符合标准的模式下会转换它们。参见 -std 和 -ansi
    选项。

**-Wp,option** ::

    你可以使用 -Wp,option 绕过编译器驱动程序，直接将 option 传递给预处理器。如果
    option 包含逗号，它将在逗号处分割成多个选项。许多选项在传递给预处理器之前会被编译
    器驱动程序修改、翻译或解释，而 -Wp 强制绕过这个阶段。预处理器的直接接口未被记录，可
    能会发生变化，因此在可能的情况下应该避免使用 -Wp，而是让驱动程序处理选项。

**-Xpreprocessor option** ::

    将 option 作为选项传递给预处理器。你可以使用此选项提供 GCC 不识别的系统特定预处理
    器选项。如果你想传递一个需要参数的选项，你必须使用 -Xpreprocessor 两次，一次用于选
    项，一次用于参数。

**-no-integrated-cpp** ::

    在编译之前将预处理作为一个独立的步骤执行。默认情况下，GCC 将预处理作为输入标记化和
    解析的一个集成部分执行。如果提供了此选项，则相应的语言前端（对于 C、C++ 和
    Objective-C 分别是 cc1、cc1plus 或 cc1obj）将被调用两次，一次仅用于预处理，一次
    用于实际编译预处理后的输入。此选项可以与 -B 或 -wrapper 选项一起使用，以指定一个替
    代的预处理器或在正常预处理和编译之间对程序源代码进行额外处理。

**-remap** ::

    启用特殊代码，以应对只允许非常短文件名的文件系统，例如 MS-DOS。

**-fdebug-cpp** ::

    此选项仅用于调试 GCC。从 CPP 或与 -E 一起使用时，它会转储有关位置映射（location
    maps）的调试信息。输出的每个标记（token）之前都会转储（dump）其位置所属的映射。从
    GCC 中不带 -E 使用时，此选项无效。

**-fpreprocessed** ::

    向预处理器表明输入文件已经被预处理过。这会抑制诸如宏展开、三字符替代转换、转义换行
    符拼接以及大多数指令的处理等操作。预处理器仍然会识别并移除注释，因此你可以将使用 -C
    预处理过的文件传递给编译器而不会出现问题。在这种模式下，集成预处理器几乎只是一个为
    前端提供标记的工具。

    如果输入文件的扩展名是 ‘.i’、‘.ii’ 或 ‘.mi’，则 -fpreprocessed 是隐含的。这些是
    GCC 使用 -save-temps 创建的预处理文件的扩展名。

**-fdirectives-only** ::

    在预处理时，处理指令但不展开宏。此选项的行为取决于 -E 和 -fpreprocessed 选项。与
    -E 一起使用时，预处理仅限于处理诸如 #define、#ifdef 和 #error 之类的指令。其他预
    处理器操作，如宏展开和三字符替代转换，不会执行。此外，-dD 选项会被隐式启用。与
    -fpreprocessed 一起使用时，预定义的命令行宏和大多数内置宏会被禁用，像 __LINE__ 这
    样依赖上下文的宏将正常处理。这使得可以编译已经使用 -E -fdirectives-only 预处理过
    的文件。同时使用 -E 和 -fpreprocessed 时，-fpreprocessed 的规则优先。这使得可以
    完全预处理之前已经使用 -E -fdirectives-only 预处理过的文件。

**-fdollars-in-identifiers** ::

    接受标识符中的 ‘$’ 字符。

**-fextended-identifiers** ::

    接受标识符中的通用字符名称和扩展字符。此选项默认在 C99（以及后续 C 标准版本）和
    C++ 中启用。

**-fno-canonical-system-headers** ::

    在预处理时，不通过规范化缩短系统头文件路径。

**-fmax-include-depth=depth** ::

    设置嵌套 #include 的最大深度，默认值为200。

**-ftabstop=width** ::

    设置制表符之间的距离。这有助于预处理器即使行中出现了制表符的情况下，能在警告或错误
    中报告正确的列号。如果值小于 1 或大于 100，则忽略该选项，默认值为 8。

**-ftrack-macro-expansion[=level]** ::

    跟踪宏展开过程中标记（token）的位置。这使得编译器在宏展开中发生编译错误时能够发出关
    于当前宏展开堆栈的诊断信息。使用此选项会使预处理器和编译器消耗更多内存。level 参数
    可用于选择标记位置跟踪的精度级别，从而在必要时减少内存消耗。level 的值为 ‘0’ 时禁
    用此选项。值为 ‘1’ 时以最小内存开销的降级模式跟踪标记位置。在这种模式下，函数式宏参
    数展开后得到的所有标记都具有相同的位置。值为 ‘2’ 时完全跟踪标记位置，这是最消耗内存
    的级别。如果不带参数使用此选项，默认参数值为‘2’。请注意，默认情况下
    -ftrack-macro-expansion=2 是启用的。

**-fmacro-prefix-map=old=new** ::

    在预处理位于 old 目录中的文件时，将 __FILE__ 和 __BASE_FILE__ 宏展开为文件位于目
    录 new 中。可以通过使用 . 作为 new 将绝对路径更改为相对路径，从而实现更具可重现性
    和位置无关性的构建，此选项还影响编译期间的 __builtin_FILE()。另参见
    -ffile-prefix-map 和 -fcanon-prefix-map。

**-fexec-charset=charset** ::

    设置执行字符集，用于字符串和字符常量，默认值为 UTF-8。charset 可以是系统 iconv 库
    例程支持的任何编码。

**-fwide-exec-charset=charset** ::

    设置宽执行字符集，用于宽字符串和字符常量。默认值为 UTF-32BE、UTF-32LE、UTF-16BE
    或 UTF-16LE 之一，具体取决于 wchar_t 的宽度以及代码生成所使用的大小端字节序。与
    -fexec-charset 类似，charset 可以是系统 iconv 库例程支持的任何编码；然而如果编码
    不能完全适应 wchar_t，你可能会遇到问题。

**-finput-charset=charset** ::

    设置输入字符集，用于从输入文件的字符集转换为 GCC 使用的源字符集。如果区域设置未指
    定，或者 GCC 无法从区域设置中获取此信息，则默认值为 UTF-8。此值可以通过区域设置或
    此命令行选项覆盖。目前，如果存在冲突，命令行选项优先。charset 可以是系统 iconv库例
    程支持的任何编码。

**-fpch-deps** ::

    当使用预编译头文件（参见预编译头文件部分）时，当输出文件的依赖关系规则时，该选项还
    会列出预编译头文件中的依赖文件。如果未指定，仅列出预编译头文件本身，不会列出哪些用
    于创建它的文件，因为当使用预编译头文件时，不会咨询这些文件。

**-fpch-preprocess** ::

    此选项允许将预编译头文件与 -E 一起使用。它在输出中插入一个特殊的 #pragma：
    #pragma GCC pch_preprocess "filename" 来标记找到的预编译头文件的位置及其文件
    名。当使用 -fpreprocessed 时，GCC 会识别这个 #pragma 并加载 PCH。默认情况下此选
    项是关闭的，因为生成的预处理输出只适合作为 GCC 的输入。它可以通过 -save-temps 启
    用。你不应在自己的代码中编写这个 #pragma，但如果 PCH 文件位于不同的位置，编辑文件
    名是安全的。文件名可以是绝对的，也可以是相对于 GCC 当前目录的相对路径。

**-fworking-directory** ::

    在预处理器输出中生成行标记，以便编译器知道预处理时的当前工作目录。启用此选项时，预
    处理器会在初始行标记之后，生成第二个行标记它带有当前工作目录并跟两个斜杠。当预处理
    输入中存在此目录时，GCC 会使用这个目录，将其作为某些调试信息格式中生成的当前工作目
    录。如果调试信息已启用，则此选项会隐式启用，但可以通过否定形式
    -fno-working-directory 来禁止。如果命令行中存在 -P 标志，则此选项无效，因为根本
    不会生成 #line 指令。

**-flarge-source-files** ::

    调整 GCC 以期望大型源文件，以较慢的编译速度和更高的内存使用为代价。具体来说，GCC
    通常会在源文件中跟踪行号和列号，并且通常在诊断信息中打印这两个数字。然而，一旦它处
    理了一定数量的源代码行，它就会停止跟踪列号，仅跟踪行号。这意味着后续行的诊断信息中
    不会包含列号。这也意味着像 -Wmisleading-indentation 这样的选项在这一点上将不再起
    作用，尽管编译器会打印一条说明。使用 -flarge-source-files 会显著增加 GCC 在停止
    跟踪列号之前可以处理的源代码行数。

预编译头文件
------------

大型项目通常有许多头文件，这些头文件在每个源文件中都会被包含。编译器处理这些头文件所花费
的时间几乎可以占据构建项目所需时间的全部。为了加快构建速度，GCC 允许你预编译一个头文
件。要创建一个预编译头文件，只需像编译其他文件一样编译它，如果需要可以使用 -x 选项使驱
动程序将其视为 C 或 C++ 头文件。你可能希望使用像 make 这样的工具来保持预编译头文件在其
包含的头文件发生更改时是最新的。

在编译时只要遇到 #include 就会搜索预编译头文件。在搜索包含的文件时，编译器会在预处理器
确定的搜索路径中查找包含文件之前，首先在每个目录中查找预编译头文件。搜索的名称是指定在
#include 中的名称，后面附加 ‘.gch’。如果无法使用预编译头文件则会忽略预编译头文件。例
如，如果你有 #include "all.h"，并且在与 all.h 相同的目录中有 all.h.gch，则会尽可能使
用预编译头文件，否则使用原始头文件。

另外，你也可以选择将预编译头文件放在一个目录中，并使用 -I 确保该目录在包含原始头文件的
目录之前被搜索。然后，如果你想检查预编译头文件是否始终被使用，可以在该目录中放置一个与原
始头文件同名的文件，其中包含一个 #error 指令。

这也适用于 -include。因此另一种使用预编译头文件的方法，是简单地将项目所使用的大多数头文
件用另一个头文件包含，预编译该头文件，并 -include 给预编译头文件。如果头文件有防止多次
包含的保护则会跳过它们，因为它们已经在预编译头文件中被包含。

如果你需要为不同的语言、目标或编译器选项预编译相同的头文件，可以创建一个名为 all.h.gch
的目录，并将每个预编译头文件放入该目录中，可能使用 -o。文件在目录中的名称无关紧要，目录
中的每个预编译头文件都会被考虑。编译期间对这些头文件的搜索没有特定的顺序，在目录中找到的
第一个有效的预编译文件就会被拿来使用。

还有许多其他可能性，仅受你的想象力、常识和构建系统的限制。预编译头文件仅在满足以下条件时
可以使用：

* 在特定编译中只能使用一个预编译头文件。

* 一旦看到第一个 C 标记（token），就不能再使用预编译头文件。在预编译头文件之前可以有预
  处理指令，但不能在另一个头文件中包含预编译头文件。

* 预编译头文件必须为当前编译生成相同的语言，你不能将 C 预编译头文件用于 C++ 编译。

* 预编译头文件必须由当前编译所使用的相同编译器二进制文件生成。

* 在包含预编译头文件之前定义的任何宏必须以与生成预编译头文件时相同的方式定义，或者不能
  影响预编译头文件，这通常意味着它们根本不出现在预编译头文件中。

  -D 选项是定义宏的一种方式，在包含的预编译头文件之前；使用 #define 也可以做到这一点。
  还有一些选项会隐式定义宏，例如 -O 和 -Wdeprecated，以这种方式定义的宏也适用相同的规
  则。

* 如果在使用预编译头文件时输出调试信息，使用 -g 或类似选项时，生成预编译头文件时必须输
  出相同类型的调试信息。然而使用 -g 生成的预编译头文件可以使用在不输出调试信息的编译
  中。

* 在生成和使用预编译头文件时，通常必须使用相同的 -m 选项。有关此规则放宽的情况，请参见
  第子模型选项部分。

* 在生成和使用预编译头文件时，以下每个选项必须相同：-fexceptions。

* 一些以 -f、-p 或 -O 开头的其他命令行选项必须以与生成预编译头文件时相同的方式定义。目
  前，尚不清楚哪些选项可以安全更改，哪些不可以；最安全的选择是在生成和使用预编译头文件
  时使用完全相同的选项。已知以下选项是安全的： ::

    -fmessage-length=
    -fpreprocessed
    -fsched-interblock
    -fsched-spec
    -fsched-spec-load
    -fsched-spec-load-dangerous
    -fsched-verbose=number
    -fschedule-insns
    -fvisibility=
    -pedantic-errors

* 地址空间布局随机化（ASLR）可能导致预编译头文件不具有二进制兼容性。如果你依赖于稳定的
  PCH 文件内容，请在生成 PCH 文件时禁用 ASLR。

对于上述除了最后一条之外的所有条件，如果条件不满足编译器会自动忽略预编译头文件。如果你发
现某个选项组合不起作用但没有忽略预编译头文件，请考虑提交错误报告。如果在生成和使用预编译
头文件时使用不同的选项，实际行为将是这些选项行为的混合。例如，如果你在生成预编译头文件时
使用 -g，但在编译时不使用，你可能会或可能不会获得预编译头文件中例程的调试信息。

目录选项
---------

这些选项指定用于头文件、库文件以及编译器自身目录的搜索： ::

    -Idir -iquote dir -isystem dir -idirafter dir
    -I- -iprefix file -iwithprefixbefore dir -iwithprefix dir
    --sysroot=dir -isysroot dir
    -nostdinc -nostdinc++ -imultilib dir
    -Ldir -Bprefix -iplugindir=dir
    -no-canonical-prefixes --no-sysroot-suffix

**-I dir -iquote dir -isystem dir -idirafter dir** ::

    将目录 dir 添加到预处理期间搜索头文件的目录列表中。如果 dir 以 ‘=’ 或 $SYSROOT 开
    头，则 ‘=’ 或 $SYSROOT 将被替换为 sysroot 前缀，参见 --sysroot 和 -isysroot。使
    用 -iquote 指定的目录仅适用于预处理指令的引号形式 #include "file"。使用 -I、
    -isystem 或 -idirafter 指定的目录适用于 #include "file" 和 #include <file> 的
    查找。你可以在命令行上指定任意数量或组合的这些选项，以在多个目录中搜索头文件。查找
    顺序如下：

    1. 对于包含指令的引号形式，首先搜索当前文件的目录。
    2. 对于包含指令的引号形式，按命令行上出现的顺序从左到右搜索由 -iquote 选项指定的目
       录。
    3. 按从左到右的顺序扫描使用 -I 选项指定的目录。
    4. 按从左到右的顺序扫描使用 -isystem 选项指定的目录。
    5. 扫描标准系统目录。
    6. 按从左到右的顺序扫描使用 -idirafter 选项指定的目录。

    你可以使用 -I 覆盖系统头文件，用你自己的版本，因为这些目录在标准系统头文件目录之前
    搜索。但是不应该使用此选项添加包含供应商提供的系统头文件的目录，而应该使用
    -isystem。-isystem 和 -idirafter 选项还将目录标记为系统目录，因此它会得到与标准
    系统目录相同的特殊处理。如果一个标准系统包含目录或使用 -isystem 指定的目录也使用
    -I 指定，则忽略 -I 选项。该目录仍然被搜索，但如果是系统目录会在它系统包含链中的正
    常位置。这是为了确保 GCC 修复有缺陷的系统头文件的程序以及 #include_next 指令的顺
    序不会意外改变。如果你确实需要改变系统目录的搜索顺序，可以使用和 -isystem 以及
    -nostdinc 选项。

**-I-** ::

    分割包含路径，此选项已被弃用。请改用 -iquote 代替 -I- 之前的 -I 目录并移除 -I- 选
    项。在 -I- 之前使用 -I 选项指定的任何目录仅用于请求 #include "file" 头文件，它们
    不会被 #include <file> 搜索。如果在 -I- 之后使用 -I 选项指定了额外的目录，那么这
    些目录将被所有 ‘#include’ 指令搜索。此外，-I- 会阻止使用当前文件目录作为
    #include "file" 的第先的搜索目录，没有方法可以覆盖 -I- 的这一效果。

**-iprefix prefix** ::

    指定 prefix 作为后续 -iwithprefix 选项的前缀。如果前缀是目录应该包含尾部的 ‘/’。

**-iwithprefix dir -iwithprefixbefore dir** ::

    将 dir 附加到之前使用 -iprefix 指定的前缀之后，并将结果目录添加到包含搜索路径中。
    -iwithprefixbefore 将目录放在 -I 会放置的位置，-iwithprefix 将目录放在
    -idirafter 会放置的位置。

**--sysroot=dir** ::

    使用 dir 作为头文件和库文件的逻辑根目录。例如编译器通常在 /usr/include 中搜索头文
    件，在 /usr/lib 中搜索库文件，此选项将改为在 dir/usr/include 和 dir/usr/lib 中
    搜索。如果你同时使用了此选项和 -isysroot 选项，那么 --sysroot 选项适用于库文件，
    而 -isysroot 选项适用于头文件。GNU 链接器（从2.16版本开始）支持此选项。如果你的链
    接器不支持此选项，--sysroot 的头文件搜索仍然有效，但库文件搜索则无效。

**-isysroot dir** ::

    此选项类似于 --sysroot 选项，但仅适用于头文件（除了 Darwin 目标，它适用于头文件和
    库文件）。

**-nostdinc** ::

    不要在标准系统目录中搜索头文件。只有明确使用 -I、-iquote、-isystem -idirafter 选
    项指定的目录（以及当前文件的目录，如果适用）会被搜索。

**-nostdinc++** ::

    不要在 C++ 特定的标准目录中搜索头文件，但仍然搜索其他标准目录。此选项在构建 C++ 库
    时使用。

**-imultilib dir** ::

    使用 dir 作为包含特定目标平台 C++ 头文件的目录的子目录。

**-Ldir** ::

    将目录 dir 添加到要搜索 -l 的目录列表中。

**-Bprefix** ::

    此选项指定在哪里找到编译器自身的可执行文件、库文件、头文件和数据文件。编译器驱动程
    序运行一个或多个子程序 cpp、cc1、as 和 ld。它尝试将 prefix 作为它尝试运行的每个程
    序的前缀，无论是否带有对应的目标机器和编译器版本 ‘machine/version/’。

    对于要运行的每个子程序，编译器驱动程序首先尝试 -B 前缀（如果有）。如果找不到该名
    称，或者没有指定 -B，驱动程序会尝试两个标准前缀，/usr/lib/gcc/ 和 
    /usr/local/lib/gcc/。如果上述两个都没有找到文件名，将使用 PATH 环境变量中指定的目
    录搜索未修改的程序名称。

    编译器检查 -B 提供的路径是否为目录，并在必要时在路径末尾添加目录分隔符。-B 前缀指
    定的目录名称，也适用于链接器中的库文件，因为编译器将这些选项转换为链接器的 -L 选
    项。它们也适用于预处理器中的头文件，因为编译器将这些选项转换为预处理器的 -isystem
    选项。在这种情况下，编译器将 ‘include’ 附加到前缀中。

    如果需要，运行时支持文件 libgcc.a 也可以使用 -B 前缀进行搜索。如果在那里找不到，将
    尝试上述两个标准前缀。如果通过这些方法找不到文件，则将其从链接中排除。

    另一种指定类似于 -B 前缀的方法是使用环境变量 GCC_EXEC_PREFIX。作为一个特殊的权宜
    之计，如果 -B 提供的路径是 [dir/]stageN/，其中 N 是 0 到 9 之间的数字，则它将被
    替换为 [dir/]include，这是为了帮助引导编译器。

**-iplugindir=dir** ::

    设置目录用于搜索插件，当该插件是以 -fplugin=name 的形式而不是
    -fplugin=path/name.so 形式指定其名称时。此选项不是供用户使用的，仅由驱动程序使
    用传递该选项。

**-no-canonical-prefixes** ::

    在生成相对前缀时，不要展开任何符号链接、解析对 ‘/../’ 或 ‘/./’ 的引用，或将路径变
    为绝对路径。

**--no-sysroot-suffix** ::

    对于某些目标，根据使用的其他选项，会在 --sysroot 指定的根目录上添加一个后缀，以便
    头文件可以在 dir/suffix/usr/include 而不是 dir/usr/include 中找到。此选项禁用此
    类后缀的添加。

诊断消息格式
------------

诊断消息格式选项汇总： ::

    -fmessage-length=n
    -fdiagnostics-plain-output
    -fdiagnostics-show-location=[once|every-line]
    -fdiagnostics-color=[auto|never|always]
    -fdiagnostics-urls=[auto|never|always]
    -fdiagnostics-format=[text|sarif-stderr|sarif-file|
        json|json-stderr|json-file]
    -fno-diagnostics-json-formatting
    -fno-diagnostics-show-option -fno-diagnostics-show-caret
    -fno-diagnostics-show-labels -fno-diagnostics-show-line-numbers
    -fno-diagnostics-show-cwe
    -fno-diagnostics-show-rule
    -fdiagnostics-minimum-margin-width=width
    -fdiagnostics-parseable-fixits -fdiagnostics-generate-patch
    -fdiagnostics-show-template-tree -fno-elide-type
    -fdiagnostics-path-format=[none|separate-events|inline-events]
    -fdiagnostics-show-path-depths
    -fno-show-column
    -fdiagnostics-column-unit=[display|byte]
    -fdiagnostics-column-origin=origin
    -fdiagnostics-escape-format=[unicode|bytes]
    -fdiagnostics-text-art-charset=[none|ascii|unicode|emoji]

警告选项
---------

警告选项汇总： ::

    -fsyntax-only -fmax-errors=n
    -w -Werror -Werror= -Wno-error= -Wfatal-errors
    -Wpedantic -pedantic -pedantic-errors -fpermissive
    -Wall -Wextra -Wabi=n

    -Wall 控制的选项：

    -Waddress
    -Waligned-new (C++ and Objective-C++ only)
    -Warray-bounds=1 (only with -O2)
    -Warray-compare
    -Warray-parameter=2
    -Wbool-compare
    -Wbool-operation
    -Wc++11-compat -Wc++14-compat -Wc++17compat -Wc++20compat
    -Wcatch-value (C++ and Objective-C++ only)
    -Wchar-subscripts
    -Wclass-memaccess (C++ and Objective-C++ only)
    -Wcomment
    -Wdangling-else
    -Wdangling-pointer=2
    -Wdelete-non-virtual-dtor (C++ and Objective-C++ only)
    -Wduplicate-decl-specifier (C and Objective-C only)
    -Wenum-compare (in C/ObjC; this is on by default in C++)
    -Wenum-int-mismatch (C and Objective-C only)
    -Wformat=1
    -Wformat-contains-nul
    -Wformat-diag
    -Wformat-extra-args
    -Wformat-overflow=1
    -Wformat-truncation=1
    -Wformat-zero-length
    -Wframe-address
    -Wimplicit (C and Objective-C only)
    -Wimplicit-function-declaration (C and Objective-C only)
    -Wimplicit-int (C and Objective-C only)
    -Winfinite-recursion
    -Winit-self (C++ and Objective-C++ only)
    -Wint-in-bool-context
    -Wlogical-not-parentheses
    -Wmain (only for C/ObjC and unless -ffreestanding)
    -Wmaybe-uninitialized
    -Wmemset-elt-size
    -Wmemset-transposed-args
    -Wmisleading-indentation (only for C/C++)
    -Wmismatched-dealloc
    -Wmismatched-new-delete (C++ and Objective-C++ only)
    -Wmissing-attributes
    -Wmissing-braces (only for C/ObjC)
    -Wmultistatement-macros
    -Wnarrowing (C++ and Objective-C++ only)
    -Wnonnull
    -Wnonnull-compare
    -Wopenmp-simd (C and C++ only)
    -Woverloaded-virtual=1 (C++ and Objective-C++ only)
    -Wpacked-not-aligned
    -Wparentheses
    -Wpessimizing-move (C++ and Objective-C++ only)
    -Wpointer-sign (only for C/ObjC)
    -Wrange-loop-construct (C++ and Objective-C++ only)
    -Wreorder (C++ and Objective-C++ only)
    -Wrestrict
    -Wreturn-type
    -Wself-move (C++ and Objective-C++ only)
    -Wsequence-point
    -Wsign-compare (C++ and Objective-C++ only)
    -Wsizeof-array-div
    -Wsizeof-pointer-div
    -Wsizeof-pointer-memaccess
    -Wstrict-aliasing
    -Wstrict-overflow=1
    -Wswitch
    -Wtautological-compare
    -Wtrigraphs
    -Wuninitialized
    -Wunknown-pragmas
    -Wunused
    -Wunused-but-set-variable
    -Wunused-const-variable=1 (only for C/ObjC)
    -Wunused-function
    -Wunused-label
    -Wunused-local-typedefs
    -Wunused-value
    -Wunused-variable
    -Wuse-after-free=2
    -Wvla-parameter
    -Wvolatile-register-var
    -Wzero-length-bounds

    -Wextra 控制的选项：

    -Wabsolute-value (only for C/ObjC)
    -Walloc-size
    -Wcalloc-transposed-args
    -Wcast-function-type
    -Wclobbered
    -Wdeprecated-copy (C++ and Objective-C++ only)
    -Wempty-body
    -Wenum-conversion (only for C/ObjC)
    -Wexpansion-to-defined
    -Wignored-qualifiers (only for C/C++)
    -Wimplicit-fallthrough=3
    -Wmaybe-uninitialized
    -Wmissing-field-initializers
    -Wmissing-parameter-type (C/ObjC only)
    -Wold-style-declaration (C/ObjC only)
    -Woverride-init (C/ObjC only)
    -Wredundant-move (C++ and Objective-C++ only)
    -Wshift-negative-value (in C++11 to C++17 and in C99 and newer)
    -Wsign-compare (C++ and Objective-C++ only)
    -Wsized-deallocation (C++ and Objective-C++ only)
    -Wstring-compare
    -Wtype-limits
    -Wuninitialized
    -Wunused-parameter (only with -Wunused or -Wall)
    -Wunused-but-set-parameter (only with -Wunused or -Wall)

    仅用于 C 和 Objective-C 的警告选项：

    -Wbad-function-cast -Wmissing-declarations
    -Wmissing-parameter-type -Wdeclaration-missing-parameter-type
    -Wmissing-prototypes -Wmissing-variable-declarations
    -Wnested-externs -Wold-style-declaration -Wold-style-definition
    -Wstrict-prototypes -Wtraditional -Wtraditional-conversion
    -Wdeclaration-after-statement -Wpointer-sign

警告是诊断消息，它们报告那些并非本质上错误但存在风险或暗示可能存在错误的构造。以下与语言
无关的选项并不启用特定的警告，而是控制 GCC 产生的诊断信息的类型：

**-fsyntax-only** ::

    检查代码的语法错误，不进行除此之外的其他检查。

**-fmax-errors=n** ::

    将错误消息的最大数量限制为 n，一旦达到该数量 GCC 将退出，而不是尝试继续处理源代
    码。如果 n 为 0（默认值），则不限制错误消息的数量。如果同时指定了
    -Wfatal-errors，则 -Wfatal-errors 优先于此选项。

**-w** ::

    抑制所有警告消息。

**-Werror** ::

    将所有警告视为错误。

**-Werror=** ::

    将指定的警告视为错误。警告的标识符附加在后面，例如 -Werror=switch 将 -Wswitch 控
    制的警告视为错误。此开关也有否定形式，用于否定 -Werror 对特定警告的影响，例如
    -Wno-error=switch 即使在 -Werror 生效时，也不会将 -Wswitch 警告视为错误。每个可
    控警告的警告消息中都包含了控制该警告的选项标识符，它们可以当作 -Werror= 和
    -Wno-error= 的参数使用。使用 -fno-diagnostics-show-option 标志可以禁用在警告消
    息中打印选项。注意，指定 -Werror=foo 自动隐含 -Wfoo，而 -Wno-error=foo 不隐含其
    他东西。

**-Wfatal-errors** ::

    此选项导致编译器在遇到第一个错误时停止编译，而不是尝试继续并打印更多的错误消息。

你可以使用以 ‘-W’ 开头的许多特定选项来请求特定的警告，例如 -Wimplicit 用于请求隐式声明
的警告。这些特定的警告选项也有以 ‘-Wno-’ 开头的否定形式用于关闭警告，例如
-Wno-implicit。本手册仅列出其中一种形式，即非默认形式。有关进一步的语言特定选项，参考
特定语言选项部分。通过启用静态分析器可以产生的额外警告，参见静态分析选项部分。

一些选项，如 -Wall 和 -Wextra，会启用其他选项，例如 -Wunused，而 -Wunused 又可能启用
进一步的选项，例如 -Wunused-value。而正负形式的组合效果是，更具体的选项优先于不太具体
的选项，与它们在命令行中的位置无关。对于相同具体性的选项，最后一个生效。通过编译器
pragma 在代码中启用或禁用的选项，就像它们出现在命令行的末尾一样生效。

当请求了一个未识别的警告选项（例如 -Wunknown-warning）时，GCC 会发出一个诊断消息，指
出该选项未被识别。然而，如果使用了 -Wno- 形式，行为略有不同：除非编译产生了其他诊断消
息，否则不会为 -Wno-unknown-warning 产生诊断消息。这允许使用新的 -Wno- 选项与旧编译器
一起使用，但如果出现了编译问题，编译器才会警告存在未识别的选项。

一些警告的有效性取决于是否启用了优化。例如 -Wsuggest-final-types 在启用链接优化时更有
效，并且其他一些警告可能只有在启用优化时才会生成。虽然优化总体上提高了控制流和数据流敏感
警告的有效性，但在某些情况下，它也可能导致误报。

**-Wpedantic -pedantic** ::

    产生严格 ISO C 和 ISO C++ 要求的所有警告；诊断所有程序是否使用禁止的扩展，以及一些
    程序是否没有遵循 ISO C 和 ISO C++ 标准。遵循的标准由 -std 选项指定的 ISO C 或
    C++ 标准版本决定。有效的 ISO C 和 ISO C++ 程序应该在启用或不启用此选项的情况下都
    能正确编译。

    在没有指定此选项的情况下，某些 GNU 扩展和传统的 C 和 C++ 特性也得到支持。使用此选
    项时，它们会被诊断产生警告消息，或使用 -pedantic-errors 产生错误消息。-Wpedantic
    不会为使用以 ‘__’ 开头和结尾的备用关键字发出警告。这种备用格式也可以用来禁用非 ISO
    ‘__intN’ 类型的警告，即 __intN__。在 __extension__ 后面的表达式中也会禁用
    pedantic 警告。然而只有系统头文件才应该使用这些规避方式，应用程序不应该使用。参见
    语言扩展备用关键字部分。

    一些关于不符合标准的程序警告由 -Wpedantic 以外的选项控制；在许多情况下它们被
    -Wpedantic 隐含，但可以使用它们的特定选项单独禁用，例如 -Wpedantic
    -Wno-pointer-sign。当使用 -std 指定的标准是 GNU 扩展的 C 方言时，例如 ‘gnu90’
    或 ‘gnu99’，存在一个对应的基础标准，即 GNU 扩展基于的那个 ISO C 版本。-Wpedantic
    的警告仅基于基础标准产生。

**-pedantic-errors** ::

    每当基础标准要求产生诊断消息时，将其当作错误进行处理。该选项并不等同于
    -Werror=pedantic，后者不那么有用因为它只将 -Wpedantic 控制的诊断视为错误，存在一
    些不符合标准的警告不被 -Wpedantic 控制，而此选项还影响始终启用的或由 -Wpedantic
    控制之外的必需的诊断。-pedantic-errors 选项会让 GCC 严格遵循 ISO C 和 ISO C++
    标准。

    如果你想将默认情况下是警告的必需诊断改为错误，但又不想启用 -Wpedantic 诊断，你可以
    指定 -pedantic-errors -Wno-pedantic。或者 -pedantic-errors
    -Wno-error=pedantic 以启用它们，但仅作为警告。一些必需的诊断默认是错误，但可以使
    用 -fpermissive 或它们的特定警告选项降级为警告，例如 -Wno-error=narrowing。

    一些非标准的诊断由特定的警告选项控制，而不是 -Wpedantic，但会被 -pedantic-errors
    视为错误。例如：

    -Wattributes（标准属性）
    -Wchanges-meaning（C++）
    -Wcomma-subscript（C++23 或更高版本）
    -Wdeclaration-after-statement（C90 或更早版本）
    -Welaborated-enum-base（C++11 或更高版本）
    -Wimplicit-int（C99 或更高版本）
    -Wimplicit-function-declaration（C99 或更高版本）
    -Wincompatible-pointer-types
    -Wint-conversion
    -Wlong-long（C90 或更早版本）
    -Wmain
    -Wnarrowing（C++11 或更高版本）
    -Wpointer-arith
    -Wpointer-sign
    -Wincompatible-pointer-types
    -Wregister（C++17 或更高版本）
    -Wvla（C90 或更早版本）
    -Wwrite-strings（C++11 或更高版本）

**-fpermissive** ::

    将一些不符合标准的必需诊断从错误降级为警告。因此使用 -fpermissive 允许一些不符合标
    准的代码编译。一些 C++ 诊断仅由这个标志控制，但它也降级了一些 C 和 C++ 诊断：

    -Wdeclaration-missing-parameter-type（仅限C和Objective-C）
    -Wimplicit-function-declaration（仅限C和Objective-C）
    -Wimplicit-int（仅限C和Objective-C）
    -Wincompatible-pointer-types（仅限C和Objective-C）
    -Wint-conversion（仅限C和Objective-C）
    -Wnarrowing（仅限C++和Objective-C++）
    -Wreturn-mismatch（仅限C和Objective-C）

    -fpermissive 选项是历史 C 语言模式（-std=c89、-std=gnu89、-std=c90、
    -std=gnu90）的默认值。

**-Wall** ::

    启用所有关于某些用户认为可疑且易于避免和修改的警告，即使与宏结合使用也是如此。它还
    启用了特定于语言的警告。请注意，有些警告标志并不隐含在 -Wall 中。其中一些警告是用
    户通常不认为可疑的构造，但有时你可能希望检查；其他一些警告在某些情况下是必要的或难
    以避免的构造，没有简单的方法来修改代码以抑制警告。这些警告中的其中一些可以使用选项
    -Wextra 启用，但大多数必须单独启用。

**-Wextra** ::

    启用 -Wall 未启用的一些额外警告标志。此选项以前称为 -W，旧名称仍然受支持，但新名称
    更具描述性。-Wextra 选项还会为以下情况打印警告消息：

    * 指针与整数零使用 <、<=、>、>= 进行比较
    * （仅限C++）条件表达式中同时出现枚举器和非枚举器
    * （仅限C++）歧义的虚基类
    * （仅限C++）对已声明为寄存器的数组进行下标操作
    * （仅限C++）对已声明为寄存器的变量取地址
    * （仅限C++）在派生类的拷贝构造函数中未初始化基类

**-Wabi（仅限 C、Objective-C、C++ 和 Objective-C++）** ::

    警告受 ABI 更改影响的代码。这包括可能与不偏向任何特定供应商的中立 C++ ABI 以及特定
    目标平台的 psABI 不兼容的代码。由于 G++ 现在默认在每个主要版本中更新 ABI，-Wabi
    用于警告 C++ ABI 兼容性问题，这些 ABI 问题是自初版本发布以来发现的添加到后续发布版
    本中的。如果选择了较旧的 ABI 版本（使用-fabi-version=n），-Wabi 会警告更多内容。

    -Wabi 也可以与显式版本号一起使用，以警告与特定 -fabi-version 级别相关的 C++ ABI
    兼容性问题，例如 -Wabi=2 警告与 -fabi-version=2 相比的变化。如果提供了显式版本
    号，并且没有指定 -fabi-compat-version=n，则此选项的版本号用于兼容性别名。如果没有
    提供显式版本号，但指定了 -fabi-compat-version=n 则版本 n 用于 C++ ABI 警告。

    尽管已经尽力警告所有这些情况，即使 G++ 正在生成不兼容的代码，但可能仍有某些情况未发
    出警告。也可能存在编译器生成的代码是兼容的，但仍然发出警告的情况。如果你担心 G++ 生
    成的代码可能与由其他编译器生成的代码不二进制兼容，你应该重写代码以避免这些警告。

    在 -fabi-version=2（从 GCC 3.4 到 4.9 的默认值）版本中已知的不兼容性包括：

    * 如果模板有参数是非类型模板参数并且是引用类型，该模板会被错误地名称修饰：
      extern int N;
      template <int &> struct S {};
      void n (S<N>) {2}
      此问题已在 -fabi-version=3 中修复。

    * 使用 __attribute ((vector_size)) 声明的 SIMD 向量类型以非标准的方式被名称修
      饰，这不允许对接受不同大小向量的函数进行重载。名称修饰在 -fabi-version=4 中被更
      改。

    * __attribute ((const)) 和 noreturn 被作为类型限定符进行名称修饰，且普通声明的
      decltype 被折叠掉了。这些名称修饰问题已在 -fabi-version=5 中修复。

    * 作为可变参数函数参数传递的带作用域的枚举器像无作用域的枚举器一样被提升，导致
      va_arg 发出警告。在大多数目标上，这实际上并不影响参数传递 ABI，因为不可能传递小
      于 int 大小的参数。

      此外，ABI 更改了模板参数 packs、const_cast、static_cast、前缀递增递减、以及用
      作模板参数的类作用域函数的名称修饰。这些问题已在-fabi-version=6中修复。

    * 默认参数作用域中的 lambda 被错误地名称修饰，且 ABI 更改了 nullptr_t 的名称修
      饰。这些问题已在 -fabi-version=7 中修复。

    * 在名称修饰带有函数 cv 限定符的函数类型时，未限定的函数类型被错误地视为替换候选。
      此问题已在 -fabi-version=8 中修复，这是 GCC 5.1 的默认值。

    * decltype(nullptr) 错误地具有 1 的对齐方式，导致未对齐访问。请注意，这并不影响带
      有 nullptr_t 参数的函数的 ABI，因为参数具有最小对齐方式。此问题已在
      -fabi-version=9 中修复，这是 GCC 5.2 的默认值。

    * 影响类型 identity 的目标平台特定属性，例如 ia32 上的函数类型调用约定（stdcall、
      regparm 等）没有影响名称修饰，导致当函数指针用作模板参数时发生名称冲突。此问题已
      在 -fabi-version=10 中修复，这是 GCC 6.1 的默认值。

    此选项还启用了关于 psABI 相关更改的警告。目前已知的 psABI 更改包括：

    * 对于 SysV/x86-64，具有 long double 成员的联合体现在按照 psABI 规定在内存中传
      递。在 GCC 4.4 之前并非如此。例如：
      union U {
          long double ld;
          int i;
      };
      现在，联合体 U 始终在内存中传递而不是寄存器。

静态分析选项
------------

静态分析选项汇总： ::

    -fanalyzer
    -fanalyzer-call-summaries
    -fanalyzer-checker=name
    -fno-analyzer-feasibility
    -fanalyzer-fine-grained
    -fanalyzer-show-events-in-system-headers
    -fno-analyzer-state-merge
    -fno-analyzer-state-purge
    -fno-analyzer-suppress-followups
    -fanalyzer-transitivity
    -fno-analyzer-undo-inlining
    -fanalyzer-verbose-edges
    -fanalyzer-verbose-state-changes
    -fanalyzer-verbosity=level
    -fdump-analyzer
    -fdump-analyzer-callgraph
    -fdump-analyzer-exploded-graph
    -fdump-analyzer-exploded-nodes
    -fdump-analyzer-exploded-nodes-2
    -fdump-analyzer-exploded-nodes-3
    -fdump-analyzer-exploded-paths
    -fdump-analyzer-feasibility
    -fdump-analyzer-infinite-loop
    -fdump-analyzer-json
    -fdump-analyzer-state-purge
    -fdump-analyzer-stderr
    -fdump-analyzer-supergraph
    -fdump-analyzer-untracked
    -Wno-analyzer-double-fclose
    -Wno-analyzer-double-free
    -Wno-analyzer-exposure-through-output-file
    -Wno-analyzer-exposure-through-uninit-copy
    -Wno-analyzer-fd-access-mode-mismatch
    -Wno-analyzer-fd-double-close
    -Wno-analyzer-fd-leak
    -Wno-analyzer-fd-phase-mismatch
    -Wno-analyzer-fd-type-mismatch
    -Wno-analyzer-fd-use-after-close
    -Wno-analyzer-fd-use-without-check
    -Wno-analyzer-file-leak
    -Wno-analyzer-free-of-non-heap
    -Wno-analyzer-imprecise-fp-arithmetic
    -Wno-analyzer-infinite-loop
    -Wno-analyzer-infinite-recursion
    -Wno-analyzer-jump-through-null
    -Wno-analyzer-malloc-leak
    -Wno-analyzer-mismatching-deallocation
    -Wno-analyzer-null-argument
    -Wno-analyzer-null-dereference
    -Wno-analyzer-out-of-bounds
    -Wno-analyzer-overlapping-buffers
    -Wno-analyzer-possible-null-argument
    -Wno-analyzer-possible-null-dereference
    -Wno-analyzer-putenv-of-auto-var
    -Wno-analyzer-shift-count-negative
    -Wno-analyzer-shift-count-overflow
    -Wno-analyzer-stale-setjmp-buffer
    -Wno-analyzer-tainted-allocation-size
    -Wno-analyzer-tainted-assertion
    -Wno-analyzer-tainted-array-index
    -Wno-analyzer-tainted-divisor
    -Wno-analyzer-tainted-offset
    -Wno-analyzer-tainted-size
    -Wanalyzer-symbol-too-complex
    -Wanalyzer-too-complex
    -Wno-analyzer-undefined-behavior-strtok
    -Wno-analyzer-unsafe-call-within-signal-handler
    -Wno-analyzer-use-after-free
    -Wno-analyzer-use-of-pointer-in-stale-stack-frame
    -Wno-analyzer-use-of-uninitialized-value
    -Wno-analyzer-va-arg-type-mismatch
    -Wno-analyzer-va-list-exhausted
    -Wno-analyzer-va-list-leak
    -Wno-analyzer-va-list-use-after-va-end
    -Wno-analyzer-write-to-const
    -Wno-analyzer-write-to-string-literal

调试选项
---------

调试选项汇总： ::

    -g -ggdb -g<level> -ggdb<level> -gvms<level>
    -gdwarf -gdwarf-version -gbtf -gctf -gctf<level>
    -gvms -gcodeview -fdebug-prefix-map=old=new
    -fvar-tracking -fvar-tracking-assignments
    -grecord-gcc-switches -gno-record-gcc-switches
    -gstrict-dwarf -gno-strict-dwarf
    -gas-loc-support -gno-as-loc-support
    -gas-locview-support -gno-as-locview-support
    -gcolumn-info -gno-column-info -gdwarf32 -gdwarf64
    -gstatement-frontiers -gno-statement-frontiers
    -gvariable-location-views -gno-variable-location-views
    -ginternal-reset-location-views -gno-internal-reset-location-views
    -ginline-points -gno-inline-points -gz[=type]
    -gsplit-dwarf -gdescribe-dies -gno-describe-dies
    -fdebug-types-section -fno-eliminate-unused-debug-types
    -femit-struct-debug-baseonly -femit-struct-debug-reduced
    -femit-struct-debug-detailed[=spec-list]
    -fno-eliminate-unused-debug-symbols -femit-class-debug-always
    -fno-merge-debug-strings -fno-dwarf2-cfi-asm

要让 GCC 为调试器生成额外的信息，几乎在所有情况下，你只需添加 -g。一些调试格式可以共存
（例如 DWARF 与 CTF），当对应调试格式都通过添加对应命令行选项启用时。

GCC 允许你将 -g 与 -O 一起使用。优化代码所采取的策略有时可能会令人惊讶：你声明的某些变
量可能根本不存在；控制流可能会短暂移动到你意想不到的地方；某些语句可能不会被执行，因为它
们计算出常量结果或它们的值已经可用；某些语句可能在不同的地方执行，因为它们已被移出循环。
尽管如此，对优化后的输出代码进行调试是可能的。这使得在可能有错误需要调试的程序中使用优化
器变得合理。

如果你没有使用其他优化选项，请考虑将 -Og 与 -g 一起使用。如果没有使用任何 -O 选项，编
译器生成的一些对调试有用信息可能根本不会执行到，因此 -Og 可能会带来更好的调试体验。

**-g** ::

    以操作系统的本地格式（native format），可能是 stabs、COFF、XCOFF 或 DWARF，生成
    调试信息。GDB 可以使用这种调试信息。在大多数使用 stabs 格式的系统上，-g 启用了只有
    GDB 可以使用的额外调试信息；这些额外信息使 GDB 中的调试工作得更好，但可能会导致其
    他调试器崩溃或拒绝读取程序。如果你想要确定是否生成额外信息，使用 -gvms。

**-ggdb** ::

    为 GDB 生成调试信息。这意味着使用最丰富的格式（DWARF 或 stabs，如果上述两种都不支
    持则使用本地格式），包括尽可能的使用 GDB 扩展。

**-g<level> -ggdb<level> -gvms<level>** ::

    请求调试信息，并使用 level 指定信息的详细程度。默认级别是 2。级别 0 不生成任何调试
    信息。因此，-g0 抵消了-g。级别 1 生成最少的信息，足以在你计划不调试的程序部分进行
    回溯。这包括函数和外部变量的描述以及行号表，但不包括局部变量的信息。级别 3 包含额外
    的信息，例如程序中存在的所有宏定义。某些调试器在你使用 -g3 时支持宏展开。如果你使用
    多个 -g 选项，带或不带级别编号，最后一个选项有效。

    -gdwarf 不接受这种形式的调试级别，以避免与 -gdwarf-<version> 混淆。相反可以使用
    额外的 -g<level> 选项来更改 DWARF 的调试级别。

**-gdwarf -gdwarf-<version>** ::

    以 DWARF 格式（如果支持）生成调试信息。version 的值可以是 2、3、4 或 5；大多数目
    标的默认版本是 5（VxWorks、TPF 和 Darwin/macOS 除外，它们默认为版本 2；AIX 默认
    为版本 4）。

    请注意，使用 DWARF 版本 2 时，某些端口需要并始终使用一些不冲突的 DWARF 3 扩展，用
    于展开表。版本 4 可能需要 GDB 7.0 和 -fvar-tracking-assignments 才能获得最大好
    处。版本 5 需要 GDB 8.0 或更高版本。

    GCC 不再支持与版本 2 和后续版本有很大不同的 DWARF 版本 1。由于历史原因，一些其他与
    DWARF 相关的选项（例如 -fno-dwarf2-cfi-asm）在其名称中保留了对 DWARF 版本 2 的引
    用，但适用于所有当前支持的 DWARF 版本。

**-gbtf** ::

    请求 BTF 调试信息。BTF 是 eBPF 目标的默认调试格式。在其他目标上，如 x86，当通过各
    自的命令行选项明确启用两种调试格式时，可以同时生成 BTF 调试信息和 DWARF 调试信息。

**-gctf -gctf<level>** ::

    请求 CTF 调试信息，并使用 level 指定应生成多少 CTF 调试信息。如果指定 -gctf 但没
    有为 level 提供值，则 CTF 调试信息的默认级别为 2。当通过各自的命令行选项明确启用两
    种调试格式时，可以同时生成 CTF 调试信息和 DWARF 调试信息。级别 0 根本不生成 CTF
    调试信息。因此，-gctf0 抵消了-gctf。级别 1 仅生成用于回溯的 CTF 信息。这包括调用
    点信息，但不包括类型信息。级别 2 仅生成文件作用域或全局作用域的实体（函数、数据对象
    等）的类型信息。

**-gvms** ::

    以 Alpha/VMS 调试格式（如果支持）生成调试信息。这是 Alpha/VMS 系统上 DEBUG 使用
    的格式。

**-gcodeview** ::

    以 CodeView 调试格式（如果支持）生成调试信息。这是 Windows 上 Microsoft Visual
    C++ 使用的格式。

**-fdebug-prefix-map=old=new** ::

    在编译位于 old 目录的文件时，将调试信息记录为文件位于 new 目录。这可以用来在调试信
    息中将构建路径替换为安装路径。它也可以用来通过使用 . 作为 new 将绝对路径更改为相对
    路径。这可以实现更具可重现性和位置无关性的构建，但可能需要额外的命令来告诉 GDB 在哪
    里找到源文件。参见 -ffile-prefix-map 和 -fcanon-prefix-map。

**-fvar-tracking** ::

    运行变量跟踪过程。它计算变量在代码中被存储的每个位置，然后生成更好的调试信息（如果
    调试格式支持此信息）。当使用优化（-Os、-O、-O2 等）、调试信息（-g）以及调试格式支
    持此功能时，编译器会默认启用变量跟踪。

**-fvar-tracking-assignments** ::

    在编译初期对用户变量的赋值进行注释，并尝试在整个编译过程中保留这些注释直至结束，以
    尝试在优化时改进调试信息。建议与 -gdwarf-4 一起使用。即使变量跟踪（var-tracking）
    被禁用，也可以启用此选项，在这种情况下，注释会被创建和维护，但在结束时会被丢弃。默
    认情况下，此标志与 -fvar-tracking 一起启用，除非启用了选择性调度（selective
    scheduling）。

优化选项
---------

优化选项汇总： ::

    -O -O1 -O2 -O3 -O0 -Og -Os -Oz -Ofast
    -fwhole-program -flto[=n]
    -flto-partition=alg -flto-compression-level=n
    -fuse-linker-plugin -ffat-lto-objects
    -fprofile-use -fprofile-use=path
    -fauto-profile -fauto-profile=path
    -fbranch-probabilities
    --param name=value

    -O 或 -O1 控制的选项：

    -fauto-inc-dec
    -fbranch-count-reg
    -fcombine-stack-adjustments
    -fcompare-elim
    -fcprop-registers
    -fdce
    -fdefer-pop
    -fdelayed-branch
    -fdse
    -fforward-propagate
    -fguess-branch-probability
    -fif-conversion
    -fif-conversion2
    -finline-functions-called-once
    -fipa-modref
    -fipa-profile
    -fipa-pure-const
    -fipa-reference
    -fipa-reference-addressable
    -fmerge-constants
    -fmove-loop-invariants
    -fmove-loop-stores
    -fomit-frame-pointer
    -freorder-blocks
    -fshrink-wrap
    -fshrink-wrap-separate
    -fsplit-wide-types
    -fssa-backprop
    -fssa-phiopt
    -ftree-bit-ccp
    -ftree-ccp
    -ftree-ch
    -ftree-coalesce-vars
    -ftree-copy-prop
    -ftree-dce
    -ftree-dominator-opts
    -ftree-dse
    -ftree-forwprop
    -ftree-fre
    -ftree-phiprop
    -ftree-pta
    -ftree-scev-cprop
    -ftree-sink
    -ftree-slsr
    -ftree-sra
    -ftree-ter
    -funit-at-a-time

    O2 额外控制的选项：

    -falign-functions -falign-jumps
    -falign-labels -falign-loops
    -fcaller-saves
    -fcode-hoisting
    -fcrossjumping
    -fcse-follow-jumps -fcse-skip-blocks
    -fdelete-null-pointer-checks
    -fdevirtualize -fdevirtualize-speculatively
    -fexpensive-optimizations
    -ffinite-loops
    -fgcse -fgcse-lm
    -fhoist-adjacent-loads
    -finline-functions
    -finline-small-functions
    -findirect-inlining
    -fipa-bit-cp -fipa-cp -fipa-icf
    -fipa-ra -fipa-sra -fipa-vrp
    -fisolate-erroneous-paths-dereference
    -flra-remat
    -foptimize-sibling-calls
    -foptimize-strlen
    -fpartial-inlining
    -fpeephole2
    -freorder-blocks-algorithm=stc
    -freorder-blocks-and-partition -freorder-functions
    -frerun-cse-after-loop
    -fschedule-insns -fschedule-insns2
    -fsched-interblock -fsched-spec
    -fstore-merging
    -fstrict-aliasing
    -fthread-jumps
    -ftree-builtin-call-dce
    -ftree-loop-vectorize
    -ftree-pre
    -ftree-slp-vectorize
    -ftree-switch-conversion -ftree-tail-merge
    -ftree-vrp
    -fvect-cost-model=very-cheap

    O3 额外控制的选项：

    -fgcse-after-reload
    -fipa-cp-clone
    -floop-interchange
    -floop-unroll-and-jam
    -fpeel-loops
    -fpredictive-commoning
    -fsplit-loops
    -fsplit-paths
    -ftree-loop-distribution
    -ftree-partial-pre
    -funswitch-loops
    -fvect-cost-model=dynamic
    -fversion-loops-for-strides

这些选项控制各种优化。如果没有指定任何优化选项，编译器的目标是减少编译成本，并确保调试产
生预期的结果。语句是独立的：如果你在语句之间设置断点来停止程序，然后你可以为任何变量分配
新值，或者将程序计数器更改为函数中的任何其他语句，并从源代码中获得完全预期的结果。

启用优化标志会使编译器尝试通过牺牲编译时间和可能的调试能力来提高性能和优化代码大小。编译
器根据其对程序的了解进行优化。一次性将多个文件编译到单个输出文件模式中，允许编译器在编译
每个文件时使用从所有文件中获得的信息。

并非所有优化都直接由标志控制。只有具有标志的优化才列在本节中。大多数优化在 -O0 或未在命
令行上设置 -O 级别时完全禁用，即使指定了单独的优化标志也是如此。同样，-Og 会抑制许多优
化选项的传递。根据目标平台和 GCC 的配置方式，每个 -O 级别启用的优化集可能与这里列出的略
有不同。你可以使用 -Q --help=optimizers 调用 GCC，以找出每个级别启用的确切优化集。

如果你使用多个 -O 选项，带或不带级别编号，最后一个这样的选项是有效的。形式为 -fflag 的
选项指定与机器无关的标志。大多数标志都有正负两种形式；-ffoo 的负形式是 -fno-foo。这里
只列出了通常使用的一种形式，你可以通过去掉或添加 ‘no-’ 来推断出另一种形式。这些具体优化
选项，它们要么被 -O 选项激活，要么需要对要执行的优化进行“微调”的罕见情况下，可以使用相
应的选项进行单独控制。

**-O -O1** ::

    启用优化。优化编译会花费更多时间，对于大型函数，还会占用更多内存。使用 -O 时，编译
    器试图减少代码大小和执行时间，而不执行任何占用大量编译时间的优化。

**-O2** ::

    进一步优化。GCC 启用几乎所有不涉及空间和速度权衡的优化。与 -O 相比，此选项增加了编
    译时间和提升了生成代码的性能。-O2 启用了 -O1 指定的所有优化标志。请注意在使用了计
    算跳转（computed goto）的程序上启用 -O2 情况下的关于 -fgcse 的警告。

**-O3** ::

    进一步优化。-O3 启用了 -O2 指定的所有优化。

**-O0** ::

    减少编译时间并生成预期的调试信息。这是默认值。

**-Og** ::

    优化调试体验。-Og 应该是标准编辑-编译-调试循环阶段选择的优化级别，提供合理的优化级
    别，同时保持快速编译和良好的调试体验。它比 -O0 更适合生成可调试代码，因为 -O0 禁用
    了一些收集调试信息的编译阶段。与 -O0 一样，-Og 完全禁用了许多优化选项，因此控制它
    们的单独选项没有效果。除此之外 -Og 启用了 -O1 的所有优化标志，除了那些可能干扰调试
    的选项：

    -fbranch-count-reg -fdelayed-branch
    -fdse -fif-conversion -fif-conversion2
    -finline-functions-called-once
    -fmove-loop-invariants -fmove-loop-stores -fssa-phiopt
    -ftree-bit-ccp -ftree-dse -ftree-pta -ftree-sra

**-Os** ::

    优化以减小代码大小为目标。-Os 启用了 -O2 的所有优化，除了那些通常会增加代码大小的
    优化之外：

    -falign-functions -falign-jumps
    -falign-labels -falign-loops
    -fprefetch-loop-arrays -freorder-blocks-algorithm=stc

    它还启用了 -finline-functions，使编译器针对代码大小而非执行速度进行调整，并执行进
    一步的优化以减小代码大小。

**-Oz** ::

    积极优化以减小代码大小而非提高执行速度。如果这些指令需要更少的字节来编码，这可能会
    增加执行的指令数量。-Oz 的行为类似于 -Os，包括启用大多数 -O2 优化。

**-Ofast** ::

    不考虑严格符合标准。-Ofast 启用了 -O3 的所有优化。它还启用了对所有严格符合标准的程
    序都不合法的优化。它启用了 -ffast-math、-fallow-store-data-races 以及 Fortran
    特定的 -fstack-arrays（除非指定了 -fmax-stack-var-size），以及
    -fno-protect-parens。它禁用了 -fsemantic-interposition。

**-fwhole-program** ::

    假设当前编译单元代表正在编译的整个程序。除 main 和通过 externally_visible 属性合
    并的函数和变量外，所有公共函数和变量都变为静态函数，并且被跨过程优化器更积极地优
    化。

    使用 -flto 时，此选项的用途有限。在大多数情况下，二进制文件中使用或导出的符号列表
    是已知的，链接器插件会将这些解析信息传递给链接时优化器。如果没有使用链接器插件，或
    者在增量链接步骤中生成最终代码时（使用 -flto -flinker-output=nolto-rel），此选项
    仍然很有用。

**-flto[=n]** ::

    此选项运行标准的链接时优化器。当使用源代码调用时，它会生成 GIMPLE（GCC 的一种内部
    表示形式），并将其写入目标文件中的特殊 ELF 段。当目标文件链接在一起时，所有函数体都
    会从这些 ELF 段中读取，并像它们属于同一个翻译单元一样进行实例化。

    要使用链接时优化器，编译时和最终链接时都需要指定 -flto 和优化选项。建议使用相同的
    选项编译参与同一链接的所有文件，并在链接时也指定这些选项。例如：

    gcc -c -O2 -flto foo.c
    gcc -c -O2 -flto bar.c
    gcc -o myprog -flto -O2 foo.o bar.o

    前两次调用 GCC 将 GIMPLE 的字节码表示形式保存到 foo.o 和 bar.o 中的特殊 ELF 段
    中。最后一次调用从 foo.o 和 bar.o 中读取 GIMPLE 字节码，将两个文件合并为一个内部
    映像，并像往常一样进行编译。由于 foo.o 和 bar.o 被合并为一个映像，这使得 GCC 中的
    所有跨过程分析和优化都可以像它们是一个单一文件一样在两个文件之间进行。例如，内联器
    可以在 bar.o 中的函数内联到 foo.o 中的函数中，反之亦然。

    另一种（更简单）启用链接时优化的方法是：

    gcc -o myprog -flto -O2 foo.c bar.c

    上述命令为 foo.c 和 bar.c 生成字节码，将它们合并为一个单一的 GIMPLE 表示形式，并
    像往常一样进行优化以生成 myprog。重要的是要记住，要启用链接时优化，需要使用 GCC 驱
    动程序来执行链接步骤。如果任何参与的对象文件是使用 -flto 命令行选项编译的，GCC 会
    自动执行链接时优化。您可以通过在链接命令中传递 -fno-lto 来覆盖自动执行链接时优化的
    决定。

    为了使整个程序优化有效，需要做出某些关于整个程序的假设。编译器需要知道哪些函数和变
    量可以被链接时优化单元之外的库和运行时访问。当链接器支持时，链接器插件（见
    -fuse-linker-plugin）会将有关已使用和外部可见符号的信息传递给编译器。如果没有链接
    器插件，应使用 -fwhole-program 以允许编译器做出这些假设，从而导致更积极的优化决
    策。

    当使用 -flto 编译文件但未使用 -fuse-linker-plugin 时，生成的目标文件比普通目标文
    件大，因为它包含 GIMPLE 字节码和通常的最终代码（见 -ffat-lto-objects）。这意味着
    带有 LTO 信息的目标文件可以像普通目标文件一样进行链接；如果在链接时传递 -fno-lto，
    则不会应用跨过程优化。注意，当启用 -fno-fat-lto-objects 时，编译阶段会更快，但您
    不能对它们进行普通的非 LTO 链接。

    在生成最终二进制文件时，GCC 只对包含字节码的文件应用链接时优化。因此，您可以将带有
    GIMPLE 字节码的目标文件和库与最终目标代码混合使用。GCC 会自动选择哪些文件在 LTO 模
    式下进行优化，哪些文件无需进一步处理即可链接。一般来说，链接时指定的选项会覆盖编译
    时指定的选项，尽管在某些情况下，GCC 会尝试从编译输入文件时使用的设置中推断链接时选
    项。

    如果在链接时未指定优化级别选项 -O，则 GCC 会使用编译目标文件时使用的最高优化级别。
    注意，仅在链接时指定优化级别选项而编译时不指定通常是无效的，原因有两个。首先，不带
    优化进行编译会抑制收集链接时优化所需信息的编译器阶段。其次，某些早期优化阶段只能在
    编译时进行，而不能在链接时进行。

    GCC 在生成字节码时会保留一些代码生成标志，因为它们需要在最终链接时使用。目前，以下
    选项及其设置是从有明确指定的第一个目标文件中获取的：-fcommon、-fexceptions、
    -fnon-call-exceptions、-fgnu-tm 和所有 -m 目标标志。

    选项 -fPIC、-fpic、-fpie 和 -fPIE 根据以下方案进行组合：

    -fPIC + -fpic = -fpic
    -fPIC + -fno-pic = -fno-pic
    -fpic/-fPIC + (无选项) = (无选项)
    -fPIC + -fPIE = -fPIE
    -fpic + -fPIE = -fpie
    -fPIC/-fpic + -fpie = -fpie

    某些会改变 ABI 的标志需要在所有编译单元中匹配，尝试在链接时用冲突的值进行覆盖的操作
    会被忽略。这包括 -freg-struct-return 和 -fpcc-struct-return 等选项。其他选项，
    如 -ffp-contract、-fno-strict-overflow、-fwrapv、-fno-trapv 或
    -fno-strict-aliasing，会传递到链接阶段，并在冲突的翻译单元中保守地合并。具体来
    说，-fno-strict-overflow、-fwrapv 和 -fno-trapv 优先；例如，-ffp-contract=off
    优先于 -ffp-contract=fast。您可以在链接时覆盖它们。

    诊断选项，如 -Wstringop-overflow，会传递到链接阶段，其设置与编译步骤中的函数粒度
    设置相匹配。注意，这仅对优化期间发出的诊断信息有影响。注意，代码转换（如内联）可能
    会导致代码区域的警告被启用或禁用，这与编译时的设置不一致。当您需要通过 -Wa 或
    -Xassembler 将选项传递给汇编器时，请确保要么使用 -fno-lto 编译这些翻译单元，要么
    在所有翻译单元上一致使用相同的汇编器选项。您也可以在 LTO 链接时指定汇编器选项。

    要启用调试信息生成，需要在编译时提供 -g。如果链接时任何输入文件是启用调试信息进行编
    译的，链接也会启用调试信息生成。任何复杂的调试信息设置，如 DWARF 级别 -gdwarf-5，
    需要在链接器命令行中明确重复，并且不鼓励在不同翻译单元中混合不同的设置。

    如果 LTO 遇到在单独编译单元中具有 C 链接声明的目标对象是不兼容类型（根据 ISO C99
    这是未定义行为），可能会发出非致命诊断信息，但运行时的行为仍然是未定义的。类似诊断
    也可能针对其他语言提出。

    LTO 的另一个特性是可以在不同语言编写的文件上应用跨过程优化：
    gcc -c -flto foo.c
    g++ -c -flto bar.cc
    gfortran -c -flto baz.f90
    g++ -o myprog -flto -O3 foo.o bar.o baz.o -lgfortran

    注意，最终链接是使用 g++ 进行的，以获取 C++ 运行时库，并添加了 -lgfortran 以获取
    Fortran 运行时库。一般来说，在 LTO 模式下混合语言时，您应该使用与在普通（非 LTO）
    编译中混合语言时相同的链接命令选项。

    如果包含 GIMPLE 字节码的对象文件存储在库存档中，例如 libfoo.a，并且您使用的是支持
    插件的链接器，则可以在 LTO 链接中提取并使用它们。要创建适合 LTO 的静态库，请使用
    gcc-ar 和 gcc-ranlib 而不是 ar 和 ranlib；要显示带有 GIMPLE 字节码的对象文件的
    符号，请使用 gcc-nm。这些命令需要 ar、ranlib 和 nm 是用插件支持编译的。在链接时，
    使用标志 -fuse-linker-plugin 以确保库参与 LTO 优化过程：
    gcc -o myprog -O2 -flto -fuse-linker-plugin a.o b.o -lfoo
    启用链接器插件后，链接器会从 libfoo.a 中提取所需的 GIMPLE 文件，并将它们传递给运
    行中的 GCC，使其成为要优化的聚合 GIMPLE 映像的一部分。如果您不使用支持插件的链接器
    或未启用链接器插件，则 libfoo.a 中的对象文件将像往常一样提取并链接，但它们不会参与
    LTO 优化过程。为了使静态库既适合 LTO 优化又适合普通链接，请使用 -flto
    -ffat-lto-objects 编译其对象文件。

    链接时优化不需要整个程序的存在即可运行。如果程序不需要导出任何符号，则可以将 -flto
    和 -fwhole-program 结合使用，以允许跨过程优化器使用更积极的假设，这可能会带来更好
    的优化机会。当链接器插件处于活动状态时（见 -fuse-linker-plugin），不需要使用
    -fwhole-program。

    LTO 的当前实现没有尝试生成可在不同类型主机之间移植的字节码。字节码文件是版本化的，
    并且有一个严格的版本检查，因此在一个版本的 GCC 中生成的字节码文件不能与旧版本或新版
    本的 GCC 一起使用。在不使用 ELF 和 DWARF 组合的系统上，链接时优化与调试信息生成配
    合得不好。

    如果指定了可选的 n，则链接时的优化和代码生成将使用安装的 make 程序并行执行 n 个并
    行作业。可以使用环境变量 MAKE 覆盖使用的程序。您也可以指定 -flto=jobserver，以使
    用 GNU make 的作业服务器模式来确定并行作业的数量。当 Makefile 调用的 GCC 已经并行
    执行时，这很有用。您必须在父 Makefile 的命令规则执行指令前加上一个 +，才能使其生
    效。如果 MAKE 是 GNU make，则此选项可能有效。即使没有选项值，GCC 也会尝试自动检测
    正在运行的 GNU make 的作业服务器。使用 -flto=auto，如果可用，则使用 GNU make 的
    作业服务器，否则回退到自动检测系统中可用的 CPU 线程数量。

**-flto-partition=alg** ::

    指定链接时优化器使用的分区算法。值可以是 1to1 指定一个原始源文件镜像分区，或者
    balanced 表示尽可能分区为大小相等的块，或者 max 表示尽可能为每个符号创建新分区。指
    定 none 作为算法将完全禁用分区和流（streaming）。默认值是 balanced，而 1to1 可以
    用作解决各种代码排序问题的变通方法，max 分区仅用于内部测试。值 one 表示仅使用一个
    分区，而值 none 跳过分区并直接从 WPA 阶段执行链接时优化步骤。

**-flto-compression-level=n** ::

    此选项指定写入 LTO 目标文件的中间语言的压缩级别，仅在与 LTO 模式（-flto）结合使用
    时才有意义。GCC 目前支持两种 LTO 压缩算法。对于 zstd，有效值范围为 0（无压缩）到
     19（最大压缩），而 zlib 支持 0 到 9 的值。超出此范围的值将被限制为支持值的最小值
     或最大值。如果未指定该选项，则使用默认的平衡压缩设置。

**-fuse-linker-plugin** ::

    启用链接时优化期间使用链接器插件。此选项依赖于链接器中的插件支持，该支持可在 gold
    或 GNU ld 2.21 或更新版本中找到。此选项允许从库存档中提取包含 GIMPLE 字节码的对象
    文件。这通过向链接时优化器暴露更多代码来提高优化质量。这些信息指定了哪些符号可以被
    外部访问（由非 LTO 对象或动态链接期间访问）。对二进制文件（以及使用隐藏可见性的共享
    库）的代码质量改进类似于 -fwhole-program。有关此标志的影响和使用方法的描述，请参阅
     -flto。当 GCC 启用了 LTO 支持且 GCC 配置为与支持插件的链接器一起使用时（GNU ld
    2.21 或更新版本或 gold），此选项默认启用。

**-ffat-lto-objects** ::

    胖 LTO 对象是包含中间语言和目标代码的对象文件。这使得它们既可以用于 LTO 链接，也可
    以用于普通链接。此选项仅在使用 -flto 编译时有效，并在链接时被忽略。
    -fno-fat-lto-objects 比普通 LTO 编译时间更短，但需要完整的工具链了解 LTO。它需要
    支持插件的链接器才能正常工作。此外，nm、ar 和 ranlib 需要支持链接器插件，才能提供
    功能齐全的构建环境（能够构建静态库等）。GCC 提供了 gcc-ar、gcc-nm 和 gcc-ranlib
    包装器，以将正确的选项传递给这些工具。使用非胖 LTO 时，需要修改 Makefile 以使用它
    们。注意，现代 binutils 提供了插件自动加载机制。将链接器插件安装到
    $libdir/bfd-plugins 中与使用命令包装器（gcc-ar、gcc-nm 和 gcc-ranlib）具有相同
    的效果。在支持链接器插件的目标上，默认值为 -fno-fat-lto-objects。

**-fprofile-use -fprofile-use=path** ::

    启用基于剖析反馈的优化以及以下优化，其中许多优化通常只有在有剖析反馈时才有益。
    -fbranch-probabilities -fprofile-values
    -funroll-loops -fpeel-loops -ftracer -fvpt
    -finline-functions -fipa-cp -fipa-cp-clone -fipa-bit-cp
    -fpredictive-commoning -fsplit-loops -funswitch-loops
    -fgcse-after-reload -ftree-loop-vectorize -ftree-slp-vectorize
    -fvect-cost-model=dynamic -ftree-loop-distribute-patterns
    -fprofile-reorder-functions
    在使用此选项之前，您必须先生成剖析信息。有关 -fprofile-generate 选项的信息，请参
    阅程序指令选项部分。

    默认情况下，如果反馈剖析与源代码不匹配，GCC 会发出错误消息。使用
    -Wnoerror=coverage-mismatch 可将此错误转换为警告。请注意，这可能会导致优化不佳的
    代码。此外，默认情况下，如果反馈剖析不存在，GCC 也会发出警告消息（参见
    -Wmissing-profile）。如果指定了路径，GCC 会在该路径中查找剖析反馈数据文件。参见
    -fprofile-dir。

**-fauto-profile -fauto-profile=path** ::

    启用基于采样的剖析反馈优化以及以下优化，其中许多优化通常只有在有剖析反馈时才有益：
    -fbranch-probabilities -fprofile-values
    -funroll-loops -fpeel-loops -ftracer -fvpt
    -finline-functions -fipa-cp -fipa-cp-clone -fipa-bit-cp
    -fpredictive-commoning -fsplit-loops -funswitch-loops
    -fgcse-after-reload -ftree-loop-vectorize -ftree-slp-vectorize
    -fvect-cost-model=dynamic -ftree-loop-distribute-patterns
    -fprofile-correction

    路径是包含 AutoFDO 剖析信息的文件名。如果省略，则默认为当前目录中的 fbdata.afdo。
    生成 AutoFDO 剖析数据文件需要在支持的 GNU/Linux 目标系统上使用 perf 工具运行您的
    程序。更多信息请参阅 https://perf.wiki.kernel.org/。例如：
    perf record -e br_inst_retired:near_taken -b -o perf.data -- your_program。

    然后使用 create_gcov 工具将原始剖析数据转换为 GCC 可以使用的格式。您还必须为该工
    具提供程序的未剥离二进制文件。详情请参阅 https://github.com/google/autofdo。例
    如：create_gcov --binary=your_program.unstripped --profile=perf.data
    --gcov=profile.afdo。

**-fbranch-probabilities** ::

    在使用 -fprofile-arcs 编译并运行程序后，您可以使用 -fbranch-probabilities 再次
    编译它，以根据每个分支的执行次数改进优化。当使用 -fprofile-arcs 编译的程序退出时，
    它会将弧（arc）执行计数保存到每个源文件的 sourcename.gcda 文件中。此数据文件中的
    信息非常依赖于生成代码的结构，因此您必须在两次编译中使用相同的源代码和相同的优化选
    项。有关文件命名的详细信息，请参阅 -fprofile-arcs。

    使用 -fbranch-probabilities 时，GCC 会在每个 JUMP_INSN 和 CALL_INSN 上放置一个
    REG_BR_PROB 注释。这些可以用于改进优化。目前，它们仅在一个地方使用：在 reorg.cc
    中，REG_BR_PROB 值被用来准确确定哪条路径被走得更频繁，而不是猜测分支最有可能走哪条
    路径。

    该选项由 -fprofile-use 和 -fauto-profile 启用。

**--param name=value** ::

    在某些地方，GCC 使用各种常量来控制优化的程度。例如，GCC 不会内联包含超过一定数量指
    令的函数。您可以使用 --param 选项在命令行上控制这些常量。特定参数的名称以及值的含
    义与编译器的内部实现相关，可能会在未来的版本中无通知地更改。要获取参数的最小值、最
    大值和默认值，请使用 --help=param -Q 选项。在每种情况下，值都是一个整数。

程序指令选项
------------

程序指令选项汇总： ::

    -p -pg -fprofile-arcs
    -fcondition-coverage --coverage -ftest-coverage
    -fprofile-abs-path -fprofile-dir=path
    -fprofile-generate -fprofile-generate=path
    -fprofile-note=path -fprofile-prefix-path=path -fprofile-prefix-map=old=new
    -fstack-protector -fstack-protector-all -fstack-protector-strong
    -fstack-protector-explicit -fstack-check -fstack-clash-protection
    -fstack-limit-register=reg -fstack-limit-symbol=sym -fno-stack-limit
    -fsplit-stack
    -fprofile-info-section -fprofile-info-section=name
    -fprofile-update=method -fprofile-filter-files=regex
    -fprofile-exclude-files=regex
    -fprofile-reproducible=[multithreaded|parallel-runs|serial]
    -fsanitize=style -fsanitize-recover -fsanitize-recover=style
    -fsanitize-trap -fsanitize-trap=style
    -fasan-shadow-offset=number -fsanitize-sections=s1,s2,...
    -fsanitize-undefined-trap-on-error -fbounds-check
    -fcf-protection=[full|branch|return|none|check]
    -fharden-compares -fharden-conditional-branches -fhardened
    -fharden-control-flow-redundancy -fhardcfr-skip-leaf
    -fhardcfr-check-exceptions -fhardcfr-check-returning-calls
    -fhardcfr-check-noreturn-calls=[always|no-xthrow|nothrow|never]
    -fstrub=disable -fstrub=strict -fstrub=relaxed
    -fstrub=all -fstrub=at-calls -fstrub=internal
    -fvtable-verify=[std|preinit|none]
    -fvtv-counts -fvtv-debug
    -finstrument-functions -finstrument-functions-once
    -finstrument-functions-exclude-function-list=sym,sym,...
    -finstrument-functions-exclude-file-list=file,file,...
    -fpatchable-function-entry=N[,M]

GCC 支持许多命令行选项，用于控制向正常生成的代码中添加运行时插桩。例如，插桩的一个目的
是收集用于查找程序热点、代码覆盖率分析、或基于性能分析的优化（PGO，Profile-Guided
Optimization）的程序剖析信息。另一类程序插桩是添加运行时检查，以检测编程错误，如无效指
针解引用或数组越界访问，以及故意的敌对攻击，如堆栈破坏或 C++ vtable 劫持。还有通用的钩
子，可用于实现其他形式的跟踪或函数级插桩，用于调试或程序分析目的。

基于性能分析的优化（PGO）是一种高级的编译优化技术，它利用程序运行时收集的性能分析数据来
指导编译器在编译阶段做出更优的决策，从而提高程序的性能。传统的编译优化是基于通用的规则和
假设对代码进行优化。而 PGO 则是在程序实际运行的环境中收集关于程序行为的详细信息，例如函
数调用频率、分支语句的执行方向（即条件判断的真假情况）等。编译器在后续的编译过程中，会根
据这些实际收集到的数据来调整优化策略，例如对经常执行的代码路径进行更激进的优化，对很少执
行的代码路径进行简化等，从而让程序在实际运行时能够更高效地执行。

**-p -pg** ::

    生成额外的代码，以编写适合分析程序 prof（对于 -p）或 gprof（对于 -pg）的配置文件
    信息。你必须在编译对应的需要分析的源文件时使用此选项，在链接该文件时也使用该选项。
    当你使用这些选项编译时，可以使用函数属性 no_instrument_function 来抑制个别函数的
    代码剖析。参见语言扩展通用函数属性部分。

**-fprofile-arcs** ::

    让编译器在生成的代码中插入特定的代码，用于对程序的控制流路径（program flow arcs）
    进行插桩。程序会记录每个分支（如 if-else 和 switch 语句中的各个分支）和函数调用
    被执行的次数。还会记录每个分支被选择（即条件判断为真时执行该分支）的次数，以及函数
    调用正常返回的次数。在支持带优先级构造函数的目标平台上，使用 -fprofile-arcs 选项进
    行的性能分析能够正确处理以下几种特殊情况。构造函数和析构函数：对于普通的构造函数和
    析构函数，程序在插桩时能准确地记录它们的执行信息。例如，在创建和销毁对象时，会正确
    统计构造函数和析构函数的调用次数和返回情况。全局变量的类型的 C++ 构造函数和析构函
    数：当一个类被用作全局变量的类型时，该类的构造函数和析构函数的执行信息也能被正确记
    录。全局变量的构造和析构有其特殊性，-fprofile-arcs 选项确保在这种情况下也能准确收
    集性能分析数据。

    当编译的程序退出时，它将此数据保存到每个源文件对应的 <auxname>.gcda 文件中。这些数
    据可用于基于性能分析的优化（-fbranch-probabilities），或用于测试覆盖率分析
    （-ftest-coverage）。每个目标文件如果明确指定了且不是最终可执行文件，则它的
    auxname 是由输出文件的名称生成的，否则由源文件的基名生成。在两种情况下，任何后缀都
    被移除。例如，对于输入文件 dir/foo.c，是 foo.gcda，或者对于指定为 -o dir/foo.o
    的输出文件是 dir/foo.gcda。

    注意，如果命令行直接链接源文件，每个源文件对应的 .gcda 文件名还会加上去掉后缀的输
    出文件名作为前缀。例如，gcc a.c b.c -o binary 将生成 binary-a.gcda 和
    binary-b.gcda 文件。

**-fcondition-coverage** ::

    添加代码，以便对程序条件进行插桩。在执行期间，程序记录条件中的哪些项真正参与了确认
    条件的计算，这可以用来验证布尔函数中的所有项都被测试过，并且对决策结果有独立的影
    响。结果可以使用 gcov --conditions 读取。参考代码覆盖测试程序（gcov）部分。

**--coverage** ::

    此选项用于编译和链接用于覆盖率分析的插桩代码。该选项是 -fprofile-arcs
    -ftest-coverage（编译时）和 -lgcov（链接时）的同义词。

    使用 -fprofile-arcs 加上优化和代码生成选项编译源文件。对于测试覆盖率分析，使用
    -ftest-coverage 选项。你不需要对程序中的每个源文件进行性能分析。

    使用 -fprofile-abs-path 编译源文件，以在 .gcno 文件中创建绝对路径名。这允许 gcov
    在编译发生在不同工作目录的项目中找到正确的源文件。

    使用 -lgcov 或 -fprofile-arcs（后者隐含前者）链接你的目标文件。

    在具有代表性的工作负载上运行程序，以生成 arc 性能分析文件信息。这可以重复任意次数。
    你可以运行你的程序的并发实例，只要文件系统支持锁定，数据文件将被正确更新。除非启用
    了严格的 ISO C 方言选项，否则 fork 调用会被正确检测到并正确处理不会有双重计数。此
    外，只要源文件和编译器选项保持不变，目标文件可以多次重新编译，相应的 .gcda 文件将
    被合并。

    对于基于性能分析的优化，使用相同的优化和代码生成选项加上 -fbranch-probabilities
    （参见优化选项）再次编译源文件。

    对于测试覆盖率分析，使用 gcov 从 .gcno 和 .gcda 文件中生成人类可读的信息。

    使用 -fprofile-arcs 时，对于程序中的每个函数，GCC 创建一个程序流程图，然后找到该
    图的生成树。只有不在生成树上的弧需要被插桩：编译器添加代码来计算这些弧被执行的次
    数。当一个弧是块的唯一出口或唯一入口时，插桩代码可以添加到块中；否则，必须创建一个
    新的基本块来保存插桩代码。

    使用 -fcondition-coverage 时，对于程序中的每个条件，GCC 创建一个 bitset 并记录对
    表达式结果有独立影响的已执行布尔值。

**-ftest-coverage** ::

    生成一个说明文件用于 gcov 代码覆盖率工具显示程序覆盖率。每个源文件的说明文件命名为
    <auxname>.gcno。有关 auxname 的描述以及如何生成测试覆盖率数据的说明，请参阅上面
    的 -fprofile-arcs 选项。如果不优化，覆盖率数据与源文件的匹配度更高。

**-fprofile-abs-path** ::

    在 .gcno 文件中自动将相对源文件名转换为绝对路径名。这使得 gcov 能够在编译发生在不
    同工作目录的项目中找到正确的源文件。

**-fprofile-dir=path** ::

    设置查找剖析数据文件的目录为 path。此选项仅影响由 -fprofile-generate、
    -ftest-coverage、-fprofile-arcs 生成的剖析数据，以及由 -fprofile-use 和
    -fbranch-probabilities 及其相关选项使用的剖析数据。可以使用绝对路径和相对路径。默
    认情况下，GCC 使用当前目录作为 path，因此剖析数据文件会出现在与目标文件相同的目录
    中。为了避免文件名冲突，如果目标文件名不是绝对路径，我们会对源文件名的绝对路径进行
    编码，并将其用作 .gcda 文件的文件名。有关文件命名的详细信息，请参阅
    -fprofile-arcs。有关类似选项，请参阅 -fprofile-note。

    当可执行文件在大规模并行环境中运行时，建议将剖析数据保存到不同的文件夹。可以通过
    在运行时导出的 path 中变量来实现：
    %p          进程 ID
    %q{VAR}     环境变量 VAR 的值

**-fprofile-generate -fprofile-generate=path** ::

    启用生成有用的程序剖析数据，这些剖析数据可用于后续基于剖析反馈的优化进行重新编译。
    编译和链接程序时都必须使用 -fprofile-generate。启用的选项包括：-fprofile-arcs、
    -fprofile-values、-finline-functions 和 -fipa-bit-cp。如果指定了 path，GCC 会
    在该路径中查找剖析反馈数据文件。有关详细信息，请参阅 -fprofile-dir。要基于收集的剖
    析信息优化程序，请使用 -fprofile-use。有关更多信息，请参阅优化选项部分。

**-fprofile-note=path** ::

    如果指定了路径，GCC 会将 .gcno 文件保存到指定的路径中。如果您将此选项与多个源文件
    结合使用，.gcno 文件将被覆盖。

**-fprofile-prefix-path=path** ::

    此选项可以与 profile-generate=profile dir 和 profile-use=profile dir 结合使
    用，以告知 GCC 构建源树的根目录在哪里。默认情况下，profile dir 将包含构建项目中所
    有目标文件的编码绝对路径的文件。当用于构建剖析二进制文件的目录与用于构建经过剖析优
    化的二进制文件的目录不同时，这不是一个理想的选择，因为在优化构建过程中无法找到剖析
    数据。在这种情况下，可以使用 -fprofile-prefix-path=path，其中 path 指向构建的根
    目录，以去除路径中无关的部分，并使所有文件名相对于主构建目录。

**-fprofile-prefix-map=old=new** ::

    当编译位于 old 目录中的文件时，记录剖析信息（使用 --coverage），就好像这些文件位于
    目录 new 中一样。另请参阅 -ffile-prefix-map 和 -fcanon-prefix-map。

**-fstack-protector** ::

    生成额外的代码以检查缓冲区溢出，例如堆栈破坏攻击。这是通过向具有易受攻击对象的函数
    添加保护变量来完成的。这包括调用 alloca 的函数，以及具有大于或等于 8 字节缓冲区的
    函数。在函数进入时初始化保护变量，然后在函数退出时进行检查。如果保护检查失败，将打
    印错误消息并退出程序。只有实际在堆栈上分配的变量才被考虑，优化掉的变量或在寄存器中
    分配的变量不计算在内。

**-fstack-protector-all** ::

    与 -fstack-protector 类似，但保护所有函数。

**-fstack-protector-strong** ::

    与 -fstack-protector 类似，但包括更多需要保护的函数，那些具有局部数组定义或引用局
    部帧地址的函数。只有实际在堆栈上分配的变量才被考虑，优化掉的变量或在寄存器中分配的
    变量不计算在内。

**-fstack-protector-explicit** ::

    与 -fstack-protector 类似，但只保护具有 stack_protect 属性的函数。

**-fstack-check** ::

    生成代码以验证不会超出堆栈的边界。如果在多线程环境中运行，则应指定此标志，但在单线
    程环境中很少需要指定此标志，因为如果只有一个堆栈，几乎所有系统都会自动检测堆栈溢
    出。请注意，此开关实际上并不会导致进行检查；操作系统或语言运行时必须执行此操作。该
    开关会生成代码以确保它们看到堆栈正在扩展。

    您还可以指定一个额外的字符串参数：“no” 表示不进行检查，“generic” 表示强制使用旧式
    检查，“specific” 表示使用最佳检查方法，等同于单独的 -fstack-check。旧式检查是一种
    通用机制，不需要编译器在特定目标上提供支持，但存在以下缺点：
    1. 修改了大对象的分配策略：如果它们的大小超过固定的阈值，则始终动态分配。请注意，这
       可能会改变某些代码的语义。
    2. 函数的静态帧大小有固定的限制：当某个特定函数达到该限制时，堆栈检查不可靠，并且编
       译器会发出警告。
    3. 效率低下：由于修改了分配策略以及通用实现，代码性能受到影响。

    请注意，如果编译器中尚未添加特定目标平台支持，则旧式堆栈检查也是 specific 的回退方
    法。-fstack-check= 是为 Ada 的需求设计的，用于检测无限递归和堆栈溢出。当编译 Ada
    代码时，specific 是一个很好的选择。它通常不足以防止堆栈冲突攻击。要防范这些攻击，
    您需要使用 -fstack-clash-protection。

**-fstack-clash-protection** ::

    生成代码以防止堆栈冲突攻击（stack clash style attack）。启用此选项后，编译器每次
    仅分配一页堆栈空间，并且每一页在分配后立即被访问。因此，它防止分配跳过操作系统提供
    的任何堆栈保护页（stack guard page）。大多数目标并不完全支持堆栈冲突保护，然而在这
    些目标上，-fstack-clash-protection 将保护动态堆栈分配。如果目标支持
    -fstack-check=specific，-fstack-clash-protection 也可以为静态堆栈分配提供有限
    的保护。

**-fstack-limit-register=reg -fstack-limit-symbol=sym -fno-stack-limit** ::

    生成代码以确保堆栈不会超出某个值，该值可以是寄存器的值或符号的地址。如果需要更大的
    堆栈，则会在运行时引发信号。对于大多数目标，信号会在堆栈超出边界之前引发，因此可以
    在不采取特殊预防措施的情况下捕获信号。例如，如果堆栈从绝对地址 0x80000000 开始并向
    下增长，则可以使用标志 -fstack-limit-symbol=__stack_limit 和
    -Wl,--defsym,__stack_limit=0x7ffe0000 来强制执行 128KB 的堆栈限制。请注意，这可
    能仅适用于 GNU 链接器。您可以使用 no_stack_limit 函数属性局部覆盖堆栈限制检查。

**-fsplit-stack** ::

    生成代码以在堆栈溢出之前自动拆分堆栈。生成的程序具有不连续的堆栈，只有在程序无法分
    配更多内存时才会溢出。这在运行多线程程序时最有用，因为不再需要为每个线程计算合适的
    堆栈大小。目前，这仅在运行 GNU/Linux 的 x86 目标上实现。

    当使用 -fsplit-stack 编译的代码调用未使用 -fsplit-stack 编译的代码时，后者可能没
    有太多堆栈空间可供运行。如果无法使用 -fsplit-stack 编译所有代码（包括库代码），则
    链接器可以修复这些调用，以便未使用 -fsplit-stack 编译的代码始终具有较大的堆栈。
    GNU binutils 2.21 及更高版本中的 gold 链接器实现了对此的支持。

代码生成选项
------------

代码生成选项汇总： ::

    -fpic -fPIC -fpie -fPIE -fno-plt
    -fno-jump-tables -fno-bit-tests
    -fverbose-asm -fstack-reuse=reuse_level
    -fcall-saved-reg -fcall-used-reg
    -ffixed-reg -fexceptions
    -fnon-call-exceptions -fdelete-dead-exceptions -funwind-tables
    -fasynchronous-unwind-tables
    -fno-gnu-unique
    -finhibit-size-directive -fcommon -fno-ident
    -fpcc-struct-return -fpack-struct[=n]
    -frecord-gcc-switches
    -freg-struct-return -fshort-enums -fshort-wchar
    -fleading-underscore -ftls-model=model
    -ftrampolines -ftrampoline-impl=[stack|heap]
    -ftrapv -fwrapv
    -fvisibility=[default|internal|hidden|protected]
    -fstrict-volatile-bitfields -fsync-libcalls

这些与机器无关的选项控制用于代码生成的接口约定。大多数选项都有正负两种形式；-ffoo 的负
形式是 -fno-foo。这里只列出了非默认的一种形式，你可以通过去掉或添加 ‘no-’ 来推断出另一
种形式。

**-fpic** ::

    如果目标机器支持，生成适合用于共享库的位置无关代码（PIC，Position-Independent
    Code）。这种代码通过全局偏移表（GOT）访问所有地址常量。动态加载器在程序启动时解析
    GOT 条目（动态加载器不是 GCC 的一部分，它是操作系统的一部分）。如果链接的可执行文
    件的 GOT 大小超过了特定于机器的最大大小，你会从链接器那里得到一个错误消息，表明
    -fpic 不起作用；在这种情况下，使用 -fPIC 重新编译。

    位置无关代码需要特殊支持，因此只在某些机器上工作。对于 x86，GCC 在 System V 上支
    持 PIC，但在 Sun 386i 上不支持。为 IBM RS/6000 生成的代码总是位置无关的。设置此
    标志时，会定义宏 __pic__ 和 __PIC__ 并且值为 1。

**-fPIC** ::

    如果目标机器支持，生成适合动态链接并避免全局偏移表大小限制的位置无关代码。此选项在
    AArch64、m68k、PowerPC 和 SPARC 上有会结果产生不同的影响。位置无关代码需要特殊支
    持，因此只在某些机器上工作。设置此标志会定义宏 __pic__ 和 __PIC__ 并且值为 2。

**-fpie -fPIE** ::

    这些选项类似于 -fpic 和 -fPIC ，但生成的位置无关代码只能链接到可执行文件中。通常这
    些选项用于编译那些需要使用 -pie 链接选项进行链接的代码。-fpie 和 -fPIE 都定义了宏
    __pie__ 和 __PIE__。这些宏对于 -fpie 的值为 1，对于 -fPIE 的值为 2。

**-fno-plt** ::

    在位置无关代码中，不要使用 PLT 进行外部函数调用。相反，在调用点从 GOT 加载被调用者
    的地址并跳转到它。这通过消除 PLT 存根并使 GOT 加载暴露于优化中，从而产生更高效的代
    码。在像 32 位 x86 这样的架构上，PLT 存根期望 GOT 指针在特定寄存器中，这为编译器
    提供了更多的寄存器分配自由度。

    延迟绑定需要使用 PLT，使用 -fno-plt 时所有外部符号在加载时解析。或者可以使用函数属
    性 noplt 来避免对特定外部函数的 PLT 调用。在位置相关代码中，一些目标还将对标记为不
    使用 PLT 的函数调用转换为使用 GOT。

**-fno-jump-tables** ::

    即使在比其他代码生成策略更有效的情况下，也不要为 switch 语句使用跳转表。此选项与
    -fpic 或 -fPIC 一起使用，用于构建构成动态链接器一部分的代码，这些代码不能引用跳转
    表的地址。在某些目标平台上，跳转表不需要 GOT，因此不需要此选项。

**-fno-bit-tests** ::

    即使在比其他代码生成策略更有效的情况下，也不要为 switch 语句使用位测试。

**-fverbose-asm** ::

    在生成的汇编代码中添加额外的注释信息，使其更易读。此选项通常只对那些需要阅读生成的
    汇编代码的人有用（可能是在调试编译器本身时）。默认情况下，-fno-verbose-asm 会省略
    这些额外信息，这在比较两个汇编文件的真实内容时很有用。

    添加的注释包括：编译器版本和命令行选项的信息，与汇编指令相关联的源代码行，格式为
    FILENAME:LINENUMBER:CONTENT OF LINE，哪些高级表达式对应于各种汇编指令操作数的提
    示。这些注释旨在供人类阅读，而不是机器，因此注释的确切格式可能会改变。

**-fstack-reuse=reuse-level** ::

    此选项控制用户声明的局部或自动变量，以及编译器生成的临时变量的堆栈空间重用。
    reuse-level 可以是 ‘all’、‘named_vars’ 或 ‘none’。‘all’ 启用了所有局部变量和临
    时变量的堆栈重用，‘named_vars’ 仅对用户定义的有名字的局部变量启用重用，而 ‘none’
    完全禁用了堆栈重用。默认值是 ‘all’。当程序将作用域局部变量或编译器生成的临时变量的
    生命周期扩展到语言定义的终点之外时，需要此选项。当变量的生命周期结束，并且该变量存
    储在内存中时，优化编译器有权将其堆栈空间与其他临时变量或作用域局部变量重用，这些变
    量的生命周期不与之重叠。扩展局部生命周期的旧版本代码可能会被堆栈重用优化破坏。

    编译器生成的临时变量的生命周期由 C++ 标准明确定义。当临时变量的生命周期结束，并且该
    临时变量存储在内存中时，优化编译器有权将其堆栈空间与其他临时变量或作用域局部变量重
    用，这些变量的生命周期不与之重叠。然而，一些遗留代码依赖于旧编译器的行为，其中临时
    变量的堆栈空间不被重用，激进的堆栈重用可能导致运行时错误。此选项用于控制临时变量堆
    栈重用优化。

开发者选项
-----------

开发者选项汇总： ::

    -d<letters> -fdump-rtl-<pass> -fdump-rtl-<pass>=filename
    -Q -fdump-passes -fdump-statistics-option -fstats
    -dumpmachine -dumpversion -dumpfullversion -dumpspecs
    -save-temps -save-temps=cwd -save-temps=obj -time[=file]
    -ftime-report -ftime-report-details
    -fmem-report -fmem-report-wpa -fpre-ipa-mem-report
    -fpost-ipa-mem-report -fstack-usage
    -print-file-name=library -print-multi-directory
    -print-multi-lib -print-multi-os-directory -print-multiarch
    -print-prog-name=program -print-libgcc-file-name
    -print-search-dirs -print-sysroot
    -print-sysroot-headers-suffix
    -fcallgraph-info[=su,da]
    -fchecking -fchecking=n
    -fdbg-cnt-list -fdbg-cnt=counter-value-list
    -fdisable-ipa-pass_name
    -fdisable-rtl-pass_name
    -fdisable-rtl-pass-name=range-list
    -fdisable-tree-pass_name
    -fdisable-tree-pass-name=range-list
    -fdump-debug -fdump-earlydebug
    -fdump-noaddr -fdump-unnumbered -fdump-unnumbered-links
    -fdump-final-insns[=file]
    -fdump-ipa-all -fdump-ipa-cgraph -fdump-ipa-inline
    -fdump-lang-all -fdump-lang-switch
    -fdump-lang-switch-options
    -fdump-lang-switch-options=filename
    -fdump-tree-all -fdump-tree-switch
    -fdump-tree-switch-options
    -fdump-tree-switch-options=filename
    -fcompare-debug[=opts] -fcompare-debug-second
    -fenable-kind-pass -fenable-kind-pass=range-list
    -fira-verbose=n -flto-report -flto-report-wpa
    -fopt-info -fopt-info-options[=file]
    -fmultiflags -fprofile-report
    -frandom-seed=string -fsched-verbose=n
    -fsel-sched-verbose -fsel-sched-dump-cfg -fsel-sched-pipelining-verbose
    -fvar-tracking-assignments-toggle -gtoggle

这里描述了主要对 GCC 开发人员感兴趣的命令行选项，包括支持编译器测试、调查编译器错误和编
译时性能问题的选项。这包括在编译过程中产生调试转储的选项；打印内存使用和执行时间等统计信
息的选项；以及打印有关 GCC 配置的信息的选项，例如它在哪里搜索库。在普通的编译和链接任务
中，你很少需要使用这些选项中的任何一个。

许多导致 GCC 将输出转储到文件的开发人员选项可以接受一个可选的 ‘=filename’ 后缀。你可以
指定 ‘stdout’ 或 ‘-’ 将转储输出到标准输出，以及 ‘stderr’ 用于标准错误。如果省略了
‘=filename’，则通过连接基础转储文件名、阶段编号、阶段字母和阶段名称来构造默认的转储文
件名。基础转储文件名是编译器生成的输出文件的名称（如果明确指定且不是可执行文件）；否则它
是源文件名。阶段编号由阶段管理器注册的阶段顺序决定，这通常与执行顺序相同，但由插件、特定
于目标平台的阶段或以其他方式较晚注册的阶段，即使执行得更早其编号也可能高于名为 ‘final’
的阶段。阶段字母是 ‘i’（过程间分析）、‘l’（特定于语言的）、‘r’（RTL）、或 ‘t’（树）。
这些文件在输出文件的目录中创建。

**-d<letters> -fdump-rtl-<pass> -fdump-rtl-<pass>=filename** ::

    在编译过程中在 letters 指定的时刻生成调试转储，这用于调试编译器所有基于RTL的阶段。
    当使用 -E 进行预处理时，某些 -dletters 开关具有不同的含义。有关预处理器特定转储选
    项的信息，请参阅预处理选项部分。调试转储可以通过 -fdump-rtl 开关或某些 -d 选项字母
    启用。以下是可以在 pass 和 letters 中使用的可能字母及其含义：

    -fdump-rtl-alignments 在计算分支对齐后转储。
    ......
    -da -fdump-rtl-all 生成上述所有转储。
    -dA 在汇编器输出中添加注释，包含各种调试信息。
    -dD 在预处理结束时转储所有宏定义，除了正常输出之外。
    -dH 发生错误时产生核心转储（core dump）。
    -dp 在汇编器输出中添加注释，指示使用了哪个模式和替代方案。每条指令的长度和成本也会
    打印出来。
    -dP 在每条指令之前以注释形式转储 RTL。这也启用了-dp 注释。
    -dx 仅生成函数的 RTL，而不是编译它。通常与 -fdump-rtl-expand 一起使用。

**-Q** ::

    使编译器在编译每个函数时打印出函数名，并在每个阶段完成时打印一些统计信息。

**-fdump-passes** ::

    在 stderr 上打印由当前命令行选项启用和禁用的优化阶段列表。

**-fdump-statistics-option** ::

    启用并控制在单独文件中转储阶段统计信息。文件名是通过在源文件名后附加以
    ‘.statistics’ 结尾的后缀生成的，文件在输出文件所在的目录中创建。如果使用了
    ‘-option’ 形式，‘-stats’ 会在整个编译单元中对计数器进行求和，而 ‘-details’
    会转储每个阶段生成的事件。默认情况下没有选项，则对每个编译的函数求和计数。

**-fstats** ::

    在编译结束时发出前端处理的统计信息。此选项仅由 C++ 前端支持，信息通常只对 g++ 开发
    团队有用。

**-dumpmachine** ::

    打印编译器的目标机器（例如，‘i686-pc-linux-gnu’），并且不执行其他操作。

**-dumpversion** ::

    打印编译器版本（例如，3.0、6.3.0 或 7）并且不执行其他操作。这是用于文件系统路径和
    规范的编译器版本。根据编译器的配置方式，它可能只是一个数字（主版本号），两个用点分
    隔的数字（主版本号和次版本号）或三个用点分隔的数字（主版本号、次版本号和修订版本
    号）。

**-dumpfullversion** ::

    打印完整的编译器版本，并且不执行其他操作。输出始终是三个用点分隔的数字，主版本号、
    次版本号和修订版本号。

**-dumpspecs** ::

    打印编译器的内置规范，并且不执行其他操作。

**-save-temps** ::

    永久存储通常的 “临时” 中间文件；将它们命名为辅助输出文件，如在 -dumpbase 和
    -dumpdir 下所述。当与 -x 命令行选项一起使用时，-save-temps 足够智能，不会用具有相
    同扩展名的中间文件覆盖输入源文件。可以通过在使用 -save-temps 之前重命名源文件来获
    取相应的中间文件。

**-save-temps=cwd** ::

    等同于 -save-temps -dumpdir ./。

**-save-temps=obj** ::

    等同于 -save-temps -dumpdir outdir/，其中 outdir/ 是使用 -o 选项指定的输出文件
    的目录，包括任何目录分隔符。如果没有使用 -o 选项，-save-temps=obj 开关的行为类似
    于 -save-temps=cwd。

**-time[=file]** ::

    报告编译序列中每个子进程所花费的 CPU 时间。对于 C 源文件，这是编译器本身和汇编器
    （以及如果进行链接，则是链接器）的时间。如果没有指定输出文件，输出如下所示：
    # cc1 0.12 0.01
    # as 0.00 0.01

    每行的第一个数字是 “用户时间”，即程序本身执行所花费的时间。第二个数字是 “系统时
    间”，即操作系统代表程序执行所花费的时间。两个数字都以秒为单位。如果指定了输出文件，
    输出将附加到指定的文件中，如下所示：
    0.12 0.01 cc1 options
    0.00 0.01 as options

    “用户时间”和“系统时间”被移到程序名称之前，传递给程序的选项被显示出来，这样就可以知
    道后来编译了哪个文件，以及使用了哪些选项。

**-ftime-report** ::

    使编译器在每个阶段完成时在 stderr 上打印一些关于所花费时间的统计信息。如果通过
    -fdiagnosticsformat=sarif-file 或 -fdiagnostics-format=sarif-stderr 请求了
    SARIF 格式的诊断输出，则 -ftime-report 信息将以 JSON 形式作为 SARIF 输出的一部分
    发出。这种 JSON 数据的精确格式可能会改变，并且由于在编译器内部的略有不同位置写入，
    其值可能与 stderr 上发出的值不完全匹配。

**-ftime-report-details** ::

    为每个传递单独记录基础设施部分所花费的时间。

**-fmem-report** ::

    使编译器在完成时打印一些关于永久内存分配的统计信息。

**-fmem-report-wpa** ::

    仅针对WPA阶段打印一些关于永久内存分配的统计信息。

**-fpre-ipa-mem-report -fpost-ipa-mem-report** ::

    使编译器在跨过程优化之前或之后打印有关永久内存分配的一些统计信息。

**-fstack-usage** ::

    使编译器按函数输出程序的堆栈使用信息。转储的文件名是通过在 auxname 后附加 .su 生成
    的。auxname 是从输出文件名生成的，如果明确指定且不是可执行文件，则使用它，否则使用
    源文件的基名。条目由三个字段组成：函数的名称，字节数，一个或多个限定符 static、
    dynamic、bounded。限定符 static 表示函数以静态方式操作堆栈：在函数入口处为帧分配
    固定数量的字节，并在函数退出时释放；函数中没有其他堆栈调整。第二个字段是这个固定数
    量的字节。限定符 dynamic 表示函数以动态方式操作堆栈：除了上述静态分配外，还在函数
    体中进行堆栈调整，例如在函数调用周围推送/弹出参数。如果还存在限定符 bounded，则这
    些调整的量在编译时有界，第二个字段是函数使用的堆栈总量的上限。如果它不存在，则这些
    调整的量在编译时无界，第二个字段仅表示有界部分。

**-print-file-name=library** ::

    打印在链接时将使用的库文件 library 的完整绝对名称——并且不执行其他操作。使用此选项
    时，GCC 不编译或链接任何内容；它只是打印文件名。

**-print-multi-directory** ::

    打印搜索结果中的所有库文件的目录名称，这些库是命令行中其他选项指定的搜索结果。此目
    录应存在于 GCC_EXEC_PREFIX 中。

**-print-multi-lib** ::

    打印从库文件目录名称到启用它们的编译器选项的映射。目录名称与选项之间用 ‘;’ 分隔，每
    个开关以 ‘@’ 开头而不是 ‘-’，多个选项之间没有空格以简化 shell 处理。

**-print-multi-os-directory** ::

    打印相对于某个 lib 子目录的所有选定库文件的 OS 库路径。如果 OS 库存在于 lib 子目
    录中且并没有使用多个库文件则打印 ‘.’；如果 OS 库存在于 libsuffix 兄弟目录中，那么
    将打印例如 ../lib64、../lib、或 ../lib32；如果 OS 库存在于 lib/subdir 子目录
    中，它将打印例如 amd64、sparcv9 或 ev6。

**-print-multiarch** ::

    打印相对于某个 lib 子目录的选定多架构的 OS 库路径。

**-print-prog-name=program** ::

    类似于 -print-file-name，但搜索像 cpp 这样的程序。

**-print-libgcc-file-name** ::

    与 -print-file-name=libgcc.a 相同。当你使用 -nostdlib 或 -nodefaultlibs 但你确
    实希望链接 libgcc.a 时，这很有用。你可以这样做：
    gcc -nostdlib files... `gcc -print-libgcc-file-name`

**-print-search-dirs** ::

    打印配置的安装目录名称以及 gcc 搜索的程序和库目录列表，并且不执行其他操作。当 gcc
    打印错误消息 ‘installation problem, cannot exec cpp0: No such file or
    directory’ 时，这很有用。要解决这个问题，你需要将 cpp0 和其他编译器组件放在 gcc
    期望找到它们的地方，或者你可以将环境变量 GCC_EXEC_PREFIX 设置为你安装它们的目录。
    别忘了末尾的 ‘/’。

**-print-sysroot** ::

    打印在编译期间使用的目标 sysroot 目录。这是在配置时或使用 --sysroot 选项指定的目
    标 sysroot，可能有一个额外的后缀，这取决于编译选项。如果没有指定目标 sysroot，该选
    项不打印任何内容。

**-print-sysroot-headers-suffix** ::

    打印在搜索头文件时添加到目标 sysroot 的后缀，或者如果编译器未配置此类后缀，则给出
    错误，并且不执行其他操作。

汇编选项
---------

可以使用下面的选项将选项传递给汇编器：

**-Wa,option** ::

    将 option 作为选项传递给汇编器，如果 option 包含逗号，它将在逗号处分割成多个选项。

**-Xassembler option** ::

    将 option 作为选项传递给汇编器，你可以使用此选项将 GCC 不识别的系统特定汇编器选
    项传递给汇编器。如果你想传递一个需要参数的选项，你必须使用 -Xassembler 两次，一次
    用于选项，一次用于参数。

链接选项
---------

链接选项汇总： ::

    object-file-name -flinker-output=type -fuse-ld=bfd|gold|lld|mold
    -llibrary -lobjc
    -nostartfiles -nodefaultlibs -nolibc -nostdlib -nostdlib++
    -e entry --entry=entry
    -pie -no-pie -static-pie
    -pthread -r -rdynamic
    -s -static -shared
    -shared-libgcc -static-libgcc -static-libstdc++
    -static-libasan -static-libtsan -static-liblsan -static-libubsan
    -symbolic -T script -u symbol
    -Wl,option -Xlinker option -z keyword

这些选项仅在编译器将目标文件链接成可执行输出文件时起作用。如果编译器没有执行链接步骤，这
些选项没有意义。

**object-file-name** ::

    不以特殊识别后缀结尾的文件名被视为目标文件或库的名称。链接器根据文件内容区分目标文
    件和库。如果进行链接，这些目标文件将作为链接器的输入。

**-E -S -c** ::

    如果使用了这些选项中的任何一个，那么链接器将不会运行，目标文件名不应作为参数。

**-flinker-output=type** ::

    此选项控制链接时优化器的代码生成。默认情况下，链接器输出由链接器插件自动确定。为了
    调试编译器以及如果需要与非LTO目标文件进行增量链接，手动控制类型可能很有用。

    如果 type 是 ‘exec’，代码生成产生静态二进制文件。在这种情况下，-fpic 和 -fpie 都
    被禁用。

    如果 type 是 ‘dyn’，代码生成产生共享库。在这种情况下，-fpic 或 -fPIC 被保留，但不
    会自动启用。这允许在可能的架构上（比如在 x86 上）构建不使用位置无关代码的共享库。

    如果 type 是 ‘pie’，代码生成产生一个 -fpie 可执行文件。这与 ‘exec’ 产生类似的优
    化，除了如果在编译时指定了 -fpie 该选项不会被禁用。

    如果 type 是 ‘rel’，编译器假设进行增量链接。包含链接时优化（LTO, Link Time
    Optimization）的中间代码的节被合并、预优化，并输出到结果目标文件中。此外，如果指定
    了 -ffat-lto-objects，将为后续非LTO链接生成二进制代码。通过增量链接生成的目标文件
    比从相同目标文件生成的静态库要小。在链接时，假设库中的大多数目标文件被使用，增量链
    接的结果加载速度也比静态库快。

    最后，‘nolto-rel’ 配置编译器进行增量链接，强制代码生成，生成最终二进制文件，并剥离
    后续链接时优化的中间代码。当多个目标文件被链接在一起时，生成的代码比禁用链接时优化
    时优化得更好（例如，跨模块内联发生），但大多数全程序优化的好处都丢失了。

    在增量链接（通过 -r）期间，链接器插件默认为 rel。然而，使用当前 GNU Binutils 接
    口，不可能将LTO对象和非LTO对象增量链接到一个混合对象文件中。如果增量链接中的任何目
    标文件不能用于链接时优化，链接器插件会发出警告并使用 ‘nolto-rel’。为了保持全程序优
    化，建议将这些对象链接到静态库中。或者，可以使用 H.J. Lu 的 binutils，它支持混合
    对象。

**-fuse-ld=bfd|gold|lld|mold** ::

    使用 bfd/gold/LLVM lld/mold 链接器而不是默认链接器。

**-llibrary -l library** ::

    在链接时搜索名为 library 的库。第二个替代方案将库作为单独的参数，仅用于 POSIX 合规
    性，不推荐使用。-l 选项直接由 GCC 传递给链接器。请参阅你的链接器文档以获取确切的详
    细信息。下面的通用描述适用于 GNU 链接器。

    链接器在标准目录列表中搜索库。搜索的目录包括几个标准系统目录，以及你用 -L 指定的目
    录。静态库是目标文件的存档，文件名通常为 liblibrary.a。某些目标还支持共享库，通常
    名称为 liblibrary.so。如果同时找到静态库和共享库，链接器默认链接共享库，除非使用
    -static 选项。

    在命令中写入此选项的位置很重要；链接器按指定顺序搜索和处理库和目标文件。因此，
    ‘foo.o -lz bar.o’ 在 foo.o 之后但 bar.o 之前搜索库 ‘z’。如果 bar.o 引用了 ‘z’ 
    中的函数，这些函数可能不会被加载。在命令行后面的目标文件或库，不能引用命令行之前的
    库中的符号。

**-lobjc** ::

    为了链接 Objective-C 或 Objective-C++ 程序，你需要这个特殊的 -l 选项。

**-nostartfiles** ::

    在链接时不要使用标准系统启动文件。除非使用 -nostdlib、-nolibc 或
    -nodefaultlibs，否则正常使用标准系统库。

**-nodefaultlibs** ::

    在链接时不要使用标准系统库。只有你指定的库被传递给链接器，链接系统库的选项（如
    -static-libgcc 或 -shared-libgcc）被忽略。正常情况下使用标准启动文件，除非使用
    -nostartfiles。编译器可能会生成对 memcmp、memset、memcpy 和 memmove的调用。这些
    条目通常由 libc 中的条目解析。当指定此选项时，应通过其他机制提供这些入口点。

**-nolibc** ::

    在链接时不要使用 C 库或与之紧密耦合的系统库。仍然链接启动文件、libgcc 或工具链提供
    的语言支持库（如 libgnat、libgfortran 或 libstdc++），除非也使用了防止它们包含的
    选项。这通常从链接命令行中移除 -lc，以及通常与 libc 一起使用的因缺少 libc 而变得毫
    无意义的系统库，例如 -lpthread 或 -lm。这适用于没有 C 库可用的裸板目标平台。

**-nostdlib** ::

    在链接时不要使用标准系统启动文件或库。没有启动文件，只有你指定的库被传递给链接器，
    链接系统库的选项（如 -static-libgcc 或 -shared-libgcc）被忽略。编译器可能会生成
    对 memcmp、memset、memcpy 和 memmove的调用，这些条目通常由 libc 中的条目解析。当
    指定此选项时，应通过其他机制提供这些入口点。

    -nostdlib 和 -nodefaultlibs 绕过的标准库之一是 libgcc.a，这是一个内部子程序库，
    GCC 使用它来克服特定机器的不足，或满足某些语言的特殊需求。有关 libgcc.a 的更多讨
    论，请参阅 GCC Internal 中的 Interfacing to GCC Output 部分。在大多数情况下，即
    使你想避免其他标准库，你也需要 libgcc.a。换句话说，当你指定 -nostdlib 或
    -nodefaultlibs 时，你通常也应该指定 -lgcc。这确保了你没有对内部 GCC 库子程序的未
    解析引用。一个这样的内部子程序是 __main，用于确保调用 C++ 构造函数。

**-nostdlib++** ::

    不要隐式链接标准 C++ 库。

**-e entry --entry=entry** ::

    指定程序的入口点是 entry。参数由链接器解释，GNU链接器接受符号名称或地址。

**-pie** ::

    在支持的目标上生成动态链接的位置无关可执行文件。为了获得可预测的结果，你还必须在指
    定此链接器选项时指定与编译时相同的选项集，-fpie、-fPIE、或模型子选项（model
    suboptions）。

**-no-pie** ::

    不要生成动态链接的位置无关可执行文件。

**-static-pie** ::

    在支持的目标上生成静态位置无关可执行文件。静态位置无关可执行文件类似于静态可执行文
    件，但可以在没有动态链接器的情况下加载到任何地址。为了获得可预测的结果，你还必须在
    指定此链接器选项时指定与编译时相同的选项集，-fpie、-fPIE、或模型子选项（model
    suboptions）。

**-pthread** ::

    链接 POSIX 线程库。此选项在 GNU/Linux 目标、大多数其他 Unix 衍生系统以及 x86
    Cygwin 和 MinGW 目标上得到支持。在某些目标上，此选项还会设置预处理器的标志，因此应
    在编译和链接时一致使用。

**-r** ::

    生成可重定位目标作为输出。这通常被称为部分链接。

**-rdynamic** ::

    在支持的目标上，将标志 -export-dynamic 传递给 ELF 链接器。这指示链接器将所有符号
    （不仅仅是已使用的符号）添加到动态符号表中。此选项对于 dlopen 的某些使用或允许从程
    序内部获取回溯是必需的。

**-s** ::

    从可执行文件中移除所有符号表和重定位信息。

**-static** ::

    在支持动态链接的系统上，此选项覆盖 -pie 并防止与共享库链接。在其他系统上，此选项没
    有效果。

**-shared** ::

    生成一个共享对象，然后可以将其与其他对象链接以形成可执行文件。并非所有系统都支持此
    选项。为了获得可预测的结果，你还必须在指定此链接器选项时指定与编译时相同的选项集，
    -fpic、-fPIC、或模型子选项（model suboptions）。

    在某些系统上，‘gcc -shared’ 需要构建额外的存根代码以使构造函数工作。在多库系统
    中，‘gcc -shared’ 必须选择正确的支持库来链接。未能提供正确的标志可能会导致细微的缺
    陷。在不需要的情况下提供这些标志是无害的。-shared 会抑制添加启动代码以改变浮点环
    境，这在某些目标上相当于通过 -ffast-math、-Ofast 或
    -funsafe-math-optimizations 完成一样。

**-shared-libgcc -static-libgcc** ::

    在提供 libgcc 作为共享库的系统上，这些选项分别强制使用共享版本或静态版本。如果在配
    置编译器时没有构建 libgcc 的共享版本，这些选项没有效果。有几种情况，应用程序应该使
    用共享 libgcc 而不是静态版本。最常见的情况是，当应用程序希望在不同的共享库之间抛出
    和捕获异常时。在这种情况下，每个库以及应用程序本身都应该使用共享 libgcc。因此，g++ 
    驱动程序在构建共享库或主可执行文件时自动添加-shared-libgcc，因为 C++ 程序通常使用
    异常，所以这是正确的做法。

    如果，相反，你使用 GCC 驱动程序创建共享库，你可能会发现它们并不总是与共享 libgcc
    链接。如果 GCC 在配置时发现你有一个非 GNU 链接器或一个不支持 --eh-frame-hdr
    选项的 GNU 链接器，它默认将共享版本的 libgcc 链接到共享库中。否则，它利用链接器优
    化掉与共享版本 libgcc 的链接，默认链接静态版本的 libgcc。这允许异常通过这样的共享
    库传播，而不会在库加载时产生重定位成本。

    然而，如果一个库或主可执行文件应该抛出或捕获异常，你必须使用 g++ 驱动程序链接它，或
    者使用 -shared-libgcc 选项，以便它与共享 libgcc 链接。

**-static-libstdc++** ::

    当使用 g++ 程序链接 C++ 程序时，它通常自动链接 libstdc++。如果 libstdc++ 可用作
    共享库，并且没有使用 -static 选项，那么它将链接到 libstdc++ 的共享版本。这通常是
    可行的。然而，有时在不完全静态链接的情况下，固定程序使用的 libstdc++ 版本也很有
    用。-static-libstdc++ 选项指示 g++ 驱动程序静态链接 libstdc++，而不一定对其他库
    进行静态链接。

**-static-libasan** ::

    当使用 -fsanitize=address 选项链接程序时，GCC 驱动程序自动链接 libasan。如果
    libasan 可用作共享库，并且没有使用 -static 选项，那么它将链接到 libasan 的共享版
    本。-static-libasan 选项指示 GCC 驱动程序静态链接 libasan，而不一定对其他库进行
    静态链接。

**-static-libtsan** ::

    当使用 -fsanitize=thread 选项链接程序时，GCC 驱动程序自动链接 libtsan。如果
    libtsan 可用作共享库，并且没有使用 -static 选项，那么它将链接到 libtsan 的共享版
    本。-static-libtsan 选项指示 GCC 驱动程序静态链接 libtsan，而不一定对其他库进行
    静态链接。

**-static-liblsan** ::

    当使用 -fsanitize=leak 选项链接程序时，GCC 驱动程序自动链接 liblsan。如果
    liblsan 可用作共享库，并且没有使用 -static 选项，那么它将链接到 liblsan 的共享版
    本。-static-liblsan 选项指示 GCC 驱动程序静态链接 liblsan，而不一定对其他库进行
    静态链接。

**-static-libubsan** ::

    当使用 -fsanitize=undefined 选项链接程序时，GCC 驱动程序自动链接 libubsan。如果
    libubsan 可用作共享库，并且没有使用 -static 选项，那么它将链接到 libubsan 的共享
    版本。-static-libubsan 选项指示 GCC 驱动程序静态链接 libubsan，而不一定对其他库
    进行静态链接。

**-symbolic** ::

    在构建共享对象时绑定对全局符号的引用。警告任何未解决的引用，除非被链接编辑器选项
    -Xlinker -z -Xlinker defs 覆盖。只有少数系统支持此选项。

**-T script** ::

    使用 script 作为链接器脚本。此选项由大多数使用 GNU 链接器的系统支持。在某些目标
    上，例如没有操作系统的裸板目标平台，链接时需要 -T 选项以避免对未定义符号的引用。

**-u symbol** ::

    假装符号 symbol 未定义，以强制链接库模块来定义它。你可以多次使用 -u，使用不同的符
    号来强制加载额外的库模块。

**-Wl,option** ::

    将 option 作为选项传递给链接器。如果 option 包含逗号，它将在逗号处分割成多个选项。
    你可以使用此语法为选项传递参数。例如，-Wl,-Map,output.map 将 -Map output.map 传
    递给链接器。当使用 GNU 链接器时，你也可以使用 -Wl,-Map=output.map 达到相同的效
    果。

**-Xlinker option** ::

    将 option 作为选项传递给链接器。你可以使用此选项提供 GCC 不识别的系统特定链接器选
    项。如果你想传递一个需要单独参数的选项，你必须使用 -Xlinker 两次，一次用于选项，一
    次用于参数。例如，要传递 -assert definitions，你必须写成 -Xlinker -assert
    -Xlinker definitions。不能写成 -Xlinker "-assert definitions"，因为这会将整个
    字符串作为单个参数传递，这不是链接器所期望的。

    当使用 GNU 链接器时，通常更方便使用 option=value 语法而不是单独的参数来传递链接器
    选项的参数。例如，你可以指定 -Xlinker -Map=output.map，而不是 -Xlinker -Map
    -Xlinker output.map。其他链接器可能不支持此命令行选项的语法。

**-z keyword** ::

    -z 直接传递给链接器，连同关键字 keyword 一起。参阅你的链接器文档中的相关部分，了解
    允许的值及其含义。

特定平台选项
-------------

每个 GCC 支持的目标平台都可以有自己的选项，例如允许相对于特定的处理器变体或 ABI 进行编
译，或者特定于对应的架构平台进行优化。按照惯例，特定平台选项的名称以 ‘-m’ 开头。编译器
的某些配置还支持额外的特定于目标平台的选项，这些通常是为了与同一平台上的其他编译器兼容。

X86 选项
---------

X86 选项汇总： ::

    -m32 -m64 -mx32 -m16 -miamcu
    -march=cpu-type -mtune=cpu-type -mcpu=cpu-type
    -mdump-tune-features -mtune-ctrl=feature-list -mno-default
    -masm=dialect -malign-double -mno-align-double
    -m96bit-long-double -m128bit-long-double
    -mlong-double-64 -mlong-double-80 -mlong-double-128
    -malign-data=type -mrtd -mregparm=num -msseregparm
    -mstackrealign mpreferred-stack-boundary=num -mincoming-stack-boundary=num
    -mstack-protector-guard=guard -mstack-protector-guard-reg=reg
    -mstack-protector-guard-offset=offset -mstack-protector-guard-symbol=symbol
    -mno-align-stringops -minline-all-stringops
    -minline-stringops-dynamically -mstringop-strategy=alg
    -mmemcpy-strategy=strategy -mmemset-strategy=strategy
    -mabi=name -mcrc32 -mno-red-zone
    -mlarge-data-threshold=threshold
    -mcmodel=small|kernel|medium|large
    -maddress-mode=long|short
    -m80387 -m8bit-idiv -msha -msha512 -maes
    -mfpmath=unit -mno-fancy-math-387
    -mno-fp-ret-in-387 -mhard-float -msoft-float
    -mno-wide-multiply -mrtd
    -mcld -mcx16 -msahf -mmovbe -mmwait
    -mrecip -mrecip=opt
    -mvzeroupper -mprefer-avx128 -mprefer-vector-width=opt
    -mpartial-vector-fp-math
    -mmove-max=bits -mstore-max=bits
    -mnoreturn-no-callee-saved-registers
    -mmmx -msse -msse2 -msse3 -mssse3 -msse4.1 -msse4.2 -msse4 -mavx
    -mavx2 -mavx512f -mavx512pf -mavx512er -mavx512cd -mavx512vl
    -mavx512bw -mavx512dq -mavx512ifma -mavx512vbmi
    -mpclmul -mfsgsbase -mrdrnd -mf16c -mfma -mpconfig -mwbnoinvd
    -mptwrite -mprefetchwt1 -mclflushopt -mclwb -mxsavec -mxsaves
    -msse4a -m3dnow -m3dnowa -mpopcnt -mabm -mbmi -mtbm -mfma4 -mxop
    -madx -mlzcnt -mbmi2 -mfxsr -mxsave -mxsaveopt -mrtm -mhle -mlwp
    -mmwaitx -mclzero -mpku -mthreads -mgfni -mvaes -mwaitpkg
    -mshstk -mmanual-endbr -mcet-switch -mforce-indirect-call
    -mavx512vbmi2 -mavx512bf16 -menqcmd
    -mvpclmulqdq -mavx512bitalg -mmovdiri -mmovdir64b -mavx512vpopcntdq
    -mavx5124fmaps -mavx512vnni -mavx5124vnniw -mprfchw -mrdpid
    -mrdseed -msgx -mavx512vp2intersect -mserialize -mtsxldtrk
    -mamx-tile -mamx-int8 -mamx-bf16 -muintr -mhreset -mavxvnni
    -mavx512fp16 -mavxifma -mavxvnniint8 -mavxneconvert
    -mprefetchi -mraoint -mamx-complex -mavxvnniint16
    -msm3 -msm4 -mapxf -mkl -mwidekl -mcmpccxadd -mamx-fp16
    -musermsr -mavx10.1 -mavx10.1-256 -mavx10.1-512 -mevex512
    -mcldemote -mms-bitfields
    -mpush-args -maccumulate-outgoing-args
    -mregparm=num -msseregparm -mveclibabi=type -mvect8-ret-in-mem
    -mpc32 -mpc64 -mpc80 -mdaz-ftz
    -momit-leaf-frame-pointer -mno-red-zone -mno-tls-direct-seg-refs
    -miamcu -mlarge-data-threshold=num
    -msse2avx -mfentry -mrecord-mcount -mnop-mcount
    -minstrument-return=type -mfentry-name=name -mfentry-section=name
    -mavx256-split-unaligned-load -mavx256-split-unaligned-store
    -mgeneral-regs-only -mcall-ms2sysv-xlogues -mrelax-cmpxchg-loop
    -mindirect-branch=choice -mfunction-return=choice
    -mindirect-branch-register -mharden-sls=choice
    -mindirect-branch-cs-prefix -mneeded -mno-direct-extern-access
    -munroll-only-small-loops -mlam=choice

**-m32 -m64 -mx32 -m16 -miamcu** ::

    为 16 位、32 位或 64 位环境生成代码。-m32 选项将 int、long 和指针类型设置为 32
    位，并生成在 32 位模式下运行的代码。-m64 选项将 int 设置为 32 位，long 和指针类型
    设置为 64 位，并为 x86-64 架构生成代码。对于 Darwin，-m64 选项还会关闭 -fno-pic
    和 -mdynamic-no-pic 选项。

    -mx32 选项将 int、long 和指针类型设置为 32 位，并为 x86-64 架构生成代码。-m16 选
    项与 -m32 相同，但会在汇编输出的开头输出 .code16gcc 汇编指令，以便二进制文件可以
    在 16 位模式下运行。-miamcu 选项生成符合 Intel MCU psABI 的代码。它需要启用 -m32
    选项。

**-march=cpu-type** ::

    为指定的机器类型 cpu-type 生成指令。与 -mtune=cpu-type 不同，后者仅针对指定的
    cpu-type 调整生成的代码，-march=cpu-type 允许 GCC 生成可能无法在除指定处理器之外
    的其他处理器上运行的代码。指定 -march=cpu-type 意味着 -mtune=cpu-type，除非另有
    说明。cpu-type 的选择包括：

    native：在编译时选择要为其生成代码的 CPU，通过确定正在编译的机器的处理器类型来实
    现。使用 -march=native 可启用本地机器支持的所有指令子集（因此结果可能无法在不同机
    器上运行）。使用 -mtune=native 可在选定指令集的约束下为本地机器生成优化代码。

    x86-64：具有 64 位扩展的通用 CPU。x86-64-v2、x86-64-v3、x86-64-v4：这些选项选择
    x86-64 psABI 对应的微架构级别。在非 x86-64 psABI 的 ABI 上，它们选择与特定微架构
    级别对应的 CPU 特性。由于这些 cpu-type 值没有对应的 -mtune 设置，使用 -march 与
    这些值时会启用通用调整。可以使用 -mtune=other-cpu-type 选项启用特定调整，其中
    other-cpu-type 是适当的值。

    i386：原始的 Intel i386 CPU。i486：Intel i486 CPU，此芯片尚未实现调度。i586、
    pentium：没有 MMX 支持的 Intel Pentium CPU。lakemont：基于 Intel Pentium CPU
    的 Intel Lakemont MCU。pentium-mmx：基于 Pentium 核心的 Intel Pentium MMX
    CPU，支持 MMX 指令集。pentiumpro：Intel Pentium Pro CPU。i686：当与 -march 一
    起使用时，使用 Pentium Pro 指令集，因此代码可以在所有 i686 系列芯片上运行。当与
    -mtune 一起使用时，其含义与 generic 相同。

    pentium2：基于 Pentium Pro 核心的 Intel Pentium II CPU，支持 MMX 和 FXSR 指令
    集。pentium3、pentium3m：基于 Pentium Pro 核心的 Intel Pentium III CPU，支持
    MMX、FXSR 和 SSE 指令集。pentium-m：Intel Pentium M；Intel Pentium III CPU 的
    低功耗版本，支持 MMX、SSE、SSE2 和 FXSR 指令集。用于 Centrino 笔记本电脑。
    pentium4、pentium4m：支持 MMX、SSE、SSE2 和 FXSR 指令集的 Intel Pentium 4
    CPU。prescott：改进版的 Intel Pentium 4 CPU，支持 MMX、SSE、SSE2、SSE3 和 FXSR
    指令集。nocona：改进版的 Intel Pentium 4 CPU，支持 64 位扩展、MMX、SSE、SSE2、
    SSE3 和 FXSR 指令集。

    i386 i486 i586 pentium lakemont pentium-mmx winchip-c6 winchip2 c3 samuel-2
    c3-2 nehemiah c7 esther i686 pentiumpro pentium2 pentium3 pentium3m
    pentium-m pentium4 pentium4m prescott nocona core2 nehalem corei7 westmere
    sandybridge corei7-avx ivybridge core-avx-i haswell core-avx2 broadwell
    skylake skylake-avx512 cannonlake icelake-client icelake-server cascadelake
    tigerlake bonnell atom silvermont slm goldmont goldmont-plus tremont knl
    knm intel geode k6 k6-2 k6-3 athlon athlon-tbird athlon-4 athlon-xp
    athlon-mp x86-64 eden-x2 nano nano-1000 nano-2000 nano-3000 nano-x2 eden-x4
    nano-x4 k8 k8-sse3 opteron opteron-sse3 athlon64 athlon64-sse3 athlon-fx
    amdfam10 barcelona bdver1 bdver2 bdver3 bdver4 znver1 znver2 btver1 btver2
    generic native

**-mtune=cpu-type -mcpu=cpu-type** ::

    调整生成代码的各个方面，使其适用于 cpu-type，但不包括 ABI 和可用指令集。虽然选择特
    定的 cpu-type 可以适当地为该特定芯片进行优调，但编译器不会生成任何无法在默认机器类
    型上运行的代码，除非您使用了 -march=cpu-type 选项。例如，如果 GCC 配置为
    i686-pc-linux-gnu，则 -mtune=pentium4 生成的代码经过调整以适应 Pentium 4，但仍
    可在 i686 机器上运行。cpu-type 的选择与 -march 相同。此外，-mtune 还支持 2 个额
    外的 cpu-type 选项：

    generic：为最常见的 IA32/AMD64/EM64T 处理器生成优化代码。如果您知道代码将运行的
    CPU，那么您应该使用相应的 -mtune 或 -march 选项，而不是 -mtune=generic。但是，如
    果您不知道应用程序用户的确切 CPU，那么您应该使用此选项。随着新处理器在市场上的部
    署，此选项的行为会发生变化。因此，如果您升级到 GCC 的较新版本，由该选项控制的代码生
    成将发生变化，以反映该版本 GCC 发布时最常见的处理器。没有 -march=generic 选项，因
    为 -march 表示编译器可以使用的指令集，而没有适用于所有处理器的通用指令集。相反，
    -mtune 表示代码优化的处理器（或在这种情况下是一组处理器）。

    intel：为当前的 Intel 处理器生成优化代码，对于此版本的 GCC 来说，这些处理器是
    Haswell 和 Silvermont。如果您知道代码将运行的 CPU，那么您应该使用相应的 -mtune
    或 -march 选项，而不是 -mtune=intel。但是，如果您希望您的应用程序在 Haswell 和
    Silvermont 上都能更好地运行，那么您应该使用此选项。随着新的 Intel 处理器在市场上
    的部署，此选项的行为会发生变化。因此，如果您升级到 GCC 的较新版本，由该选项控制的代
    码生成将发生变化，以反映该版本 GCC 发布时最新的 Intel 处理器。没有 -march=intel
    选项，因为 -march 表示编译器可以使用的指令集，而没有适用于所有处理器的通用指令集。
    相反，-mtune 表示代码优化的处理器（或在这种情况下，是一组处理器）。

    -mcpu=cpu-type 是 -mtune 的过时同义词。

**-mdump-tune-features** ::

    此选项指示 GCC 输出 x86 性能调整特性和默认设置的名称。这些名称可以在
    -mtune-ctrl=feature-list 中使用。

**-mtune-ctrl=feature-list** ::

    此选项用于对 x86 代码生成特性进行细粒度控制。feature-list 是用逗号分隔的特性名称
    列表。请参考 -mdump-tune-features。指定时，如果特性名称前没有 ^，则启用该特性，否
    则禁用。-mtune-ctrl=feature-list 旨在供 GCC 开发者使用。使用它可能会导致未被测试
    覆盖的代码路径，可能会导致编译器内部错误或运行时错误。

**-mno-default** ::

    此选项指示 GCC 关闭所有可调特性。请参考 -mtune-ctrl=feature-list 和
    -mdump-tune-features。

**-masm=dialect** ::

    使用选定的方言输出汇编指令。也会影响基本汇编（见语言扩展基本汇编部分）和扩展汇编
    （见语言扩展扩展汇编部分）中使用的方言。支持 att 或 intel，默认值是 att。Darwin
     不支持 intel。

**-malign-double -mno-align-double** ::

    控制 GCC 是否将 double、long double 和 long long 变量对齐到双字边界（8 字节）或
    单字边界（4 字节）。将 double 变量对齐到双字边界可以在 Pentium 上生成运行速度稍快
    的代码，但会占用更多内存。在 x86-64 上，默认启用 -malign-double。

    警告：如果使用 -malign-double 选项，包含上述类型的结构体将与 x86-32 发布的应用二
    进制接口规范对齐方式不同，并且与未使用该选项编译的代码中的结构体不兼容。

**-m96bit-long-double -m128bit-long-double** ::

    这些选项控制 long double 类型的大小。x86-32 应用二进制接口指定大小为 96 位（12
    字节），因此在 32 位模式下默认使用 -m96bit-long-double。现代架构（Pentium 及更新
    的架构）更倾向于将 long double 对齐到 8 字节或 16 字节边界。在符合 ABI 的数组或结
    构体中，这是不可能的。因此，指定 -m128bit-long-double 会通过在 long double 后面
    填充额外的 32 位零（4 字节），将其对齐到 16 字节边界。

    在 x86-64 编译器中，默认选择是 -m128bit-long-double，因为其 ABI 指定 long
    double 对齐到 16 字节边界。请注意，这些选项都不会启用超过 x87 标准 80 位精度的额
    外精度。

    警告：如果覆盖目标 ABI 的默认值，这将改变包含 long double 变量的结构体和数组的大
    小，并修改接受 long double 的函数调用约定。因此，它们与未使用该选项编译的代码不兼
    容。

**-mlong-double-64|80|128** ::

    这些选项控制 long double 类型的大小。64 位大小使 long double 类型等同于 double
    类型。这是 32 位 Bionic C 库的默认值。128 位大小使 long double 类型等同于
    __float128 类型。这是 64 位 Bionic C 库的默认值。

    警告：如果覆盖目标 ABI 的默认值，这将改变包含 long double 变量的结构体和数组的大
    小，并修改接受 long double 的函数调用约定。因此，它们与未使用该选项编译的代码不兼
    容。

**-malign-data=type** ::

    控制 GCC 如何对齐变量。type 的支持值包括：
    compat：使用与 GCC 4.8 及更早版本兼容的增加对齐值。
    abi：使用 psABI 指定的对齐值。
    cacheline：使用增加的对齐值以匹配缓存行大小。
    默认值是 compat。

**-mrtd** ::

    使用不同的函数调用约定，其中接受固定数量参数的函数使用 ret num 指令返回，该指令在
    返回时弹出它们的参数。这在调用方节省了一条指令，因为无需在那里弹出参数。您可以使用
    函数属性 stdcall 指定某个函数使用这种调用序列。您也可以通过使用函数属性 cdecl 覆盖
    -mrtd 选项。有关详细信息，请参阅语言扩展函数属性部分。

    警告：这种调用约定与通常在 Unix 上使用的调用约定不兼容，因此如果您需要调用用 Unix
    编译器编译的库，则不能使用它。此外，您必须为所有接受可变数量参数的函数（包括
    printf）提供函数原型；否则，对这些函数的调用将生成错误的代码。此外，如果调用函数时
    参数过多，将导致严重错误的代码，正常情况下通多余的参数会被无害地忽略。

**-mregparm=num** ::

    控制用于传递整数参数的寄存器数量。默认情况下，不使用寄存器传递参数，最多可以使用 3
    个寄存器。您可以使用函数属性 regparm 控制特定函数的行为。有关详细信息，请参阅语言
    扩展函数属性部分。

    警告：如果使用此开关，并且 num 不为零，则必须使用相同的值构建所有模块，包括任何库。
    这包括系统库和启动模块。

**-msseregparm** ::

    对 float 和 double 参数及返回值使用 SSE 寄存器传递约定。您可以使用函数属性
    sseregparm 控制特定函数的行为。有关详细信息，请参阅语言扩展函数属性部分。

    警告：如果使用此开关，则必须使用相同的值构建所有模块，包括任何库。这包括系统库和启
    动模块。

**-mstackrealign** ::

    在入口处重新对齐堆栈。在 x86 上，-mstackrealign 选项生成一个替代的序言和尾声，如
    果需要，会重新对齐运行时堆栈。这支持将保持 4 字节堆栈对齐的旧代码与保持 16 字节堆
    栈对齐的现代代码混合使用，以实现 SSE 兼容性。请参阅还适用于单个函数的属性
    force_align_arg_pointer。

**-mpreferred-stack-boundary=num** ::

    尝试将堆栈边界保持在 2 的 num 次幂字节边界上。如果未指定
    -mpreferred-stack-boundary，默认值为 4（即 16 字节或 128 位）。

    警告：当为禁用 SSE 扩展的 x86-64 架构生成代码时，可以使用
    -mpreferred-stack-boundary=3 以将堆栈边界保持在 8 字节边界上。由于 x86-64 ABI
    要求 16 字节堆栈对齐，因此这是 ABI 不兼容的，仅用于堆栈空间严格限制的受控环境中。
    当用 16 字节堆栈对齐编译的函数（例如标准库中的函数）被用未对齐堆栈调用时，此选项会
    导致错误代码。在这种情况下，SSE 指令可能会导致未对齐的内存访问陷阱。此外，对于 16
    字节对齐的对象（包括 x87 long double 和 int128），可变参数处理不正确，导致错误的
    结果。您必须使用 -mpreferred-stack-boundary=3 构建所有模块，包括任何库。这包括系
    统库和启动模块。

**-mincoming-stack-boundary=num** ::

    假设传入的堆栈对齐到 2 的 num 次幂字节边界上。如果未指定
    -mincoming-stack-boundary，则使用 -mpreferred-stack-boundary 指定的值。在
    Pentium 和 Pentium Pro 上，double 和 long double 值应该对齐到 8 字节边界（参见
    -malign-double），否则会遭受显著的运行时性能损失。在 Pentium III 上，流式 SIMD
    扩展（SSE）数据类型 __m128 如果未对齐到 16 字节，则可能无法正常工作。为了确保这些
    值在堆栈上的正确对齐，堆栈边界必须与堆栈上存储的任何值所需的对齐程度一样对齐。此
    外，每个函数都必须生成以保持堆栈对齐。因此，使用较低地址对齐要求编译的函数调用使
    用较高地址对齐要求编译的函数，很可能会导致堆栈未对齐。建议有回调函数的库始终使用默
    认设置。

    这种额外的对齐确实会消耗额外的堆栈空间，并且通常会增加代码大小。对堆栈空间使用敏感
    的代码，如嵌入式系统和操作系统内核，可能希望将首选对齐降低到
    -mpreferred-stack-boundary=2（即 4 字节）。

**-mstack-protector-guard[-reg|-offset|-symbol]=guard|reg|offset|symbol** ::

    这些选项用于生成栈保护代码，栈保护机制是一种防止栈溢出攻击的安全特性。当程序中存在
    栈溢出漏洞时，攻击者可能会覆盖栈上的返回地址等重要信息，从而执行恶意代码。栈保护机
    制通过在栈上插入一个特殊的值（成为金丝雀值，canary）来检测栈是否被破坏。guard 指定
    存储特殊值的位置，支持的位置包括 global 全局值或 tls 每个线程都有自己独立的值，tls
    是默认值。只有在程序指令选项中指定了 -fstack-protector 或 -fstack-protector-all
    时，该选项才有效。

    对于后一种选择，选项 -mstack-protector-guard-reg=reg 和
    -mstack-protector-guard-offset=offset 进一步指定使用哪个段寄存器（%fs 或 %gs）
    作为基寄存器用来读取金丝雀值，以及从该基寄存器的偏移量。其默认值是相关 ABI 中指定的
    值。symbol 用于覆盖金丝雀对应的符号相对于于 TLS 块的偏移值。

**-mno-align-stringops** ::

    不要对内联字符串操作的目标进行对齐。此选项可以减少代码大小，并在目标已经对齐但 GCC
    无法知晓的情况下提高性能。

**-minline-all-stringops** ::

    默认情况下，GCC 只有在目标已知至少对齐到 4 字节边界时才会内联字符串操作。此选项启
    用更多的内联操作，会增加代码大小，但可能会提高依赖于快速 memcpy 和 memset 的代码的
    性能，尤其是在处理较短长度时。该选项还启用了对所有指针对齐的 strlen 的内联展开。

**-minline-stringops-dynamically** ::

    对于未知大小的字符串操作，使用运行时检查：对于小块数据使用内联代码，对于大块数据使
    用库调用。

**-mstringop-strategy=alg** ::

    覆盖内置方法使用启发式方法来确定用于内联字符串操作的特定算法。alg 的允许值包括：
    rep_byte、rep_4byte、rep_8byte 使用指定大小的 i386 rep 前缀进行展开，
    byte_loop、loop 、nrolled_loop 展开为内联循环，libcall 始终使用库调用。

**-mmemcpy-strategy=strategy** ::

    覆盖内置方法来启发式决定 __builtin_memcpy 是否内联以及使用哪种内联算法，当复制操
    作的预期大小是已知的时。strategy 是一个以 alg:max_size:dest_align 三元组为元素的
    用逗号分隔的列表。alg 在 -mstringop-strategy 中指定，max_size 指定允许使用内联算
    法 alg 的最大字节大小。对于最后一个三元组，max_size 必须是 -1。列表中三元组的
    max_size 必须按递增顺序指定。第一个三元组中 alg 的最小字节大小为 0，后续的最小字节
    大小为前一个的 max_size + 1。

**-mmemset-strategy=strategy** ::

    该选项与 -mmemcpy-strategy= 类似，但用于控制 __builtin_memset 的展开。

**-mabi=name** ::

    为指定的调用约定生成代码。允许的值是 sysv（用于 GNU/Linux 和其他系统的 ABI）和 ms
    （用于 Microsoft ABI）。默认情况下，当目标是 Microsoft Windows 时使用 Microsoft
    ABI，而在所有其他系统上使用 SysV ABI。您可以使用函数属性 ms_abi 和 sysv_abi 控制
    特定函数的行为。参考语言扩展函数属性部分的信息。

**-mcrc32** ::

    此选项启用内置函数 __builtin_ia32_crc32qi、__builtin_ia32_crc32hi、
    __builtin_ia32_crc32si 和 __builtin_ia32_crc32di，以生成 crc32 机器指令。

**-mno-red-zone** ::

    对于 x86-64 代码，不要使用所谓的 “红区”。红区是由 x86-64 ABI 规定的；它是一个位于
    堆栈指针位置之外的 128 字节区域，不会被信号或中断处理程序修改，因此可以用于临时数
    据，而无需调整堆栈指针。-mno-red-zone 禁用了这个红区。

**-mlarge-data-threshold=threshold** ::

    当指定 -mcmodel=medium 或 -mcmodel=large 时，大于 threshold 的数据对象将被放置
    在大数据段中。默认值是 65535。

**-mcmodel=small** ::

    为小型代码模型生成代码：程序及其符号必须链接到地址空间的低 2GB 中。指针是 64 位。
    程序可以静态或动态链接。这是默认代码模型。

**-mcmodel=kernel** ::

    为内核代码模型生成代码。内核运行在地址空间的负 2GB 中。此模型必须用于 Linux 内核代
    码。

**-mcmodel=medium** ::

    为中型模型生成代码：程序链接到地址空间的低 2GB 中。小符号也放置在那里。大小大于
    -mlarge-data-threshold 的符号被放入大数据或 BSS 段中，可以位于 2GB 以上。程序可
    以静态或动态链接。

**-mcmodel=large** ::

    为大型模型生成代码。此模型不对段的地址和大小做任何假设。

**-maddress-mode=long** ::

    为长地址模式生成代码。这仅支持 64 位和 x32 环境。它是 64 位环境的默认地址模式。

**-maddress-mode=short** ::

    为短地址模式生成代码。这仅支持 32 位和 x32 环境。它是 32 位和 x32 环境的默认地址
    模式。

AArch64 选项
-------------

AArch64 选项汇总： ::

    -mabi=name -mbig-endian -mlittle-endian
    -mgeneral-regs-only
    -mcmodel=tiny -mcmodel=small -mcmodel=large
    -mstrict-align -mno-strict-align
    -momit-leaf-frame-pointer
    -mtls-dialect=desc -mtls-dialect=traditional
    -mtls-size=size
    -mfix-cortex-a53-835769 -mfix-cortex-a53-843419
    -mlow-precision-recip-sqrt -mlow-precision-sqrt -mlow-precision-div
    -mpc-relative-literal-loads
    -msign-return-address=scope
    -mbranch-protection=none|standard|pac-ret[+leaf+b-key]|bti
    -mharden-sls=opts
    -march=name -mcpu=name -mtune=name
    -moverride=string -mverbose-cost-dump
    -mstack-protector-guard=guard -mstack-protector-guard-reg=sysreg
    -mstack-protector-guard-offset=offset -mtrack-speculation
    -moutline-atomics -mearly-ldp-fusion -mlate-ldp-fusion

ARM 选项
---------

ARM 选项汇总： ::

    -mapcs-frame -mno-apcs-frame
    -mabi=name
    -mapcs-stack-check -mno-apcs-stack-check
    -mapcs-reentrant -mno-apcs-reentrant
    -mgeneral-regs-only
    -msched-prolog -mno-sched-prolog
    -mlittle-endian -mbig-endian
    -mbe8 -mbe32
    -mfloat-abi=name
    -mfp16-format=name
    -mthumb-interwork -mno-thumb-interwork
    -mcpu=name -march=name -mfpu=name
    -mtune=name -mprint-tune-info
    -mstructure-size-boundary=n
    -mabort-on-noreturn
    -mlong-calls -mno-long-calls
    -msingle-pic-base -mno-single-pic-base
    -mpic-register=reg
    -mnop-fun-dllimport
    -mpoke-function-name
    -mthumb -marm -mflip-thumb
    -mtpcs-frame -mtpcs-leaf-frame
    -mcaller-super-interworking -mcallee-super-interworking
    -mtp=name -mtls-dialect=dialect
    -mword-relocations
    -mfix-cortex-m3-ldrd
    -mfix-cortex-a57-aes-1742098
    -mfix-cortex-a72-aes-1655431
    -munaligned-access
    -mneon-for-64bits
    -mslow-flash-data
    -masm-syntax-unified
    -mrestrict-it
    -mverbose-cost-dump
    -mpure-code
    -mcmse
    -mfix-cmse-cve-2021-35465
    -mstack-protector-guard=guard -mstack-protector-guard-offset=offset
    -mfdpic
    -mbranch-protection=none|standard|pac-ret[+leaf][+bti]|bti[+pac-ret[+leaf]]
