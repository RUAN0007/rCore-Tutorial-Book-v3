.. 实现应用程序

Implement Applications
===========================

.. toctree::
   :hidden:
   :maxdepth: 5

.. 本节导读

Section Guide
-------------------------------

This section mainly explains how to design and implement applications that are loaded and run one by one by the batch operating system system, on a premise that application are assumed to run in user mode (U privileged mode). In fact, if the code of the application conforms to the constraints of the user-mode privilege level, it can run normally in the user-mode; but if the application executes privileged instructions or illegal operations (such as executing illegal instructions, accessing an illegal address, etc. ), that would generate an exception and cause the program to exit. It is one of the key tasks of the batch operating system system to be implemented to ensure that the code of the application program can run normally in the user mode. The key points of design and implementation of the application program are:

- Memory layout of the application
- System calls issued by the application

To some extent, the application designed here has many similarities with the minimal execution environment in Chapter 1. That is to design an application program and a basic support function library, so that the application program fulfills its own tasks through the services provided by the operating system in the user mode.

.. 本节主要讲解如何设计实现被批处理系统逐个加载并运行的应用程序。这有个前提，即应用程序假定在用户态（U 特权级模式）下运行。实际上，如果应用程序的代码都符合用户态特权级的约束，那它完全可以正常在用户态中运行；但如果应用程序执行特权指令或非法操作（如执行非法指令，访问一个非法的地址等），那会产生异常，并导致程序退出。保证应用程序的代码在用户态能正常运行是将要实现的批处理系统的关键任务之一。应用程序的设计实现要点是：

.. - 应用程序的内存布局
.. - 应用程序发出的系统调用

.. 从某种程度上讲，这里设计的应用程序与第一章中的最小用户态执行环境有很多相同的地方。即设计一个应用程序和基本的支持功能库，这样应用程序在用户态通过操作系统提供的服务完成自身的任务。

.. 应用程序设计

Design of Applications
-----------------------------

Applications and user libraries (composed of multiple rs files including entry functions, initialization functions, I/O functions, and system call interfaces) are placed in the ``user`` directory of the project root directory, which is the same as the bare-metal application in Chapter 1 The difference is mainly in the directory file structure and memory layout of the project:

- ``user/src/bin/*.rs``: individual applications
- ``user/src/*.rs``: user library (including entry function, initialization function, I/O function and system call interface, etc.)
- ``user/src/linker.ld``: Description of the memory layout of the application.

.. 应用程序、用户库（包括入口函数、初始化函数、I/O 函数和系统调用接口等多个 rs 文件组成）放在项目根目录的 ``user`` 目录下，它和第一章的裸机应用不同之处主要在项目的目录文件结构和内存布局上：

.. - ``user/src/bin/*.rs`` ：各个应用程序
.. - ``user/src/*.rs`` ：用户库（包括入口函数、初始化函数、I/O 函数和系统调用接口等）
.. - ``user/src/linker.ld`` ：应用程序的内存布局说明。

.. 项目结构

Project Structure
^^^^^^^^^^^^^^^^^^^^^^

.. 似乎并不是这样？

    用户库看起来很复杂，它预留了直到 ch7 内核才能实现的系统调用接口，console 模块还实现了输出缓存区。它们不是为本章准备的，你只需关注本节提到的部分即可。

We see that there is a ``bin`` directory under the ``user/src`` directory. There are multiple files in ``bin``. Currently there are at least three programs (one file is an application program), which are:

- ``hello_world``: Print a line ``Hello world from user mode program!`` to the screen
- ``store_fault``: access an illegal physical address, test whether the batch system will be affected by this error
- ``power`` : Continuously switch privilege levels between computing operations and printing string operations

The batch system loads and runs the filenames in ascending order of the numbers at the beginning of their names.

The implementation of each application is in a corresponding single file. Open one of the files, and you will see the entire application logic with only one ``main`` function and several related functions.

We can also see that the code tries to import external libraries:

.. 我们看到 ``user/src`` 目录下面多出了一个 ``bin`` 目录。``bin`` 里面有多个文件，目前里面至少有三个程序（一个文件是一个应用程序），分别是：

.. - ``hello_world`` ：在屏幕上打印一行 ``Hello world from user mode program!``
.. - ``store_fault`` ：访问一个非法的物理地址，测试批处理系统是否会被该错误影响
.. - ``power`` ：不断在计算操作和打印字符串操作之间进行特权级切换

.. 批处理系统会按照文件名开头的数字编号从小到大的顺序加载并运行它们。

.. 每个应用程序的实现都在对应的单个文件中。打开其中一个文件，会看到里面只有一个 ``main`` 函数和若干相关的函数所形成的整个应用程序逻辑。

.. 我们还能够看到代码中尝试引入了外部库：

.. code-block:: rust

    #[macro_use]
    extern crate user_lib;

This external library is actually ``lib.rs`` in the ``user`` directory and several submodules it refers to. As for why this external library is called ``user_lib`` instead of ``user``, the name of the directory where ``lib.rs`` is located, it is because in ``user/Cargo.toml`` we have the name of the library is set: ``name = "user_lib"``. It serves as a user library that the source programs in the ``bin`` directory depend on, and is equivalent to the standard library provided by other programming languages.

In ``lib.rs`` we define the entry point ``_start`` for the user library:

.. 这个外部库其实就是 ``user`` 目录下的 ``lib.rs`` 以及它引用的若干子模块中。至于这个外部库为何叫 ``user_lib`` 而不叫 ``lib.rs`` 所在的目录的名字 ``user`` ，是因为在 ``user/Cargo.toml`` 中我们对于库的名字进行了设置： ``name =  "user_lib"`` 。它作为 ``bin`` 目录下的源程序所依赖的用户库，等价于其他编程语言提供的标准库。

.. 在 ``lib.rs`` 中我们定义了用户库的入口点 ``_start`` ：

.. code-block:: rust
    :linenos:

    #[no_mangle]
    #[link_section = ".text.entry"]
    pub extern "C" fn _start() -> ! {
        clear_bss();
        exit(main());
        panic!("unreachable after sys_exit!");
    }

The second line uses the Rust macro to put ``_start`` in the compiled assembly code into a code segment named ``.text.entry``, which is convenient for us to adjust its location for the subseuquent linking. So that it can be used as an entry point for the user library.

Starting from line 4, after entering the user library entry, firstly, as in the first chapter, manually clear the ``.bss`` section (unfortunately it can only be done in the user library); then call the ``main`` function to get a return value of type ``i32``.And finally it calls the ``exit`` interface provided by the user library to exit the application and sets the return value of the ``main`` function to informs the batch operating system.

We also see another ``main`` in ``lib.rs``:
In ``lib.rs`` we define the entry point ``_start`` for the user library:

.. 第 2 行使用 Rust 的宏将 ``_start`` 这段代码编译后的汇编代码中放在一个名为 ``.text.entry`` 的代码段中，方便我们在后续链接的时候调整它的位置使得它能够作为用户库的入口。

.. 从第 4 行开始，进入用户库入口之后，首先和第一章一样，手动清空需要零初始化的 ``.bss`` 段（很遗憾到目前为止底层的批处理系统还没有这个能力，所以我们只能在用户库中完成）；然后调用 ``main`` 函数得到一个类型为 ``i32`` 的返回值，最后调用用户库提供的 ``exit`` 接口退出应用程序，并将 ``main`` 函数的返回值告知批处理系统。

.. 我们还在 ``lib.rs`` 中看到了另一个 ``main`` ：

.. code-block:: rust
    :linenos:

    #[linkage = "weak"]
    #[no_mangle]
    fn main() -> i32 {
        panic!("Cannot find main!");
    }

On line 1, we use Rust's macro to mark its function symbol ``main`` as weakly linked. In this way, at the time of the final link, although  ``lib.rs`` and applications in ``bin`` directories both have the ``main`` symbol, but because the `` The `main`` symbol is a weak link, and the linker will prioritize to use the main logic of the application in the ``bin`` directory as real ``main``. Here we mainly carry out a certain degree of protection. If there is no ``main`` in the ``bin`` directory, then the compilation can pass, but an error will be reported at runtime.

In order to support the above link operations, we need to add at the beginning of ``lib.rs``:

.. 第 1 行，我们使用 Rust 的宏将其函数符号 ``main`` 标志为弱链接。这样在最后链接的时候，虽然在 ``lib.rs`` 和 ``bin`` 目录下的某个应用程序都有 ``main`` 符号，但由于 ``lib.rs`` 中的 ``main`` 符号是弱链接，链接器会使用 ``bin`` 目录下的应用主逻辑作为 ``main`` 。这里我们主要是进行某种程度上的保护，如果在 ``bin`` 目录下找不到任何 ``main`` ，那么编译也能够通过，但会在运行时报错。

.. 为了支持上述这些链接操作，我们需要在 ``lib.rs`` 的开头加入：

.. code-block:: rust

    #![feature(linkage)]


.. _term-app-mem-layout:

.. 内存布局

Memory Layout
^^^^^^^^^^^^^^^^^^^^^^

In ``user/.cargo/config``, we use the linker script ``user/src/linker.ld`` when setting up links as in Chapter 1. Among the important things we do are:

- Adjust the starting physical address of the program to ``0x80400000``, and all three applications will be loaded to run on this physical address;
- Put ``.text.entry`` where ``_start`` is located at the beginning of the entire program, that is to say, As long as the batch processing system jumps to ``0x80400000`` after loading, it has entered the entry point of the user library, and will jump to the main logic of the application after initialization;
- Provides the start and end addresses of the ``.bss`` section of the final executable file, which is convenient for the ``clear_bss`` function to use.

The rest are basically the same as the first chapter.
In order to support the above link operations, we need to add at the beginning of ``lib.rs``:

.. 在 ``user/.cargo/config`` 中，我们和第一章一样设置链接时使用链接脚本 ``user/src/linker.ld`` 。在其中我们做的重要的事情是：

.. - 将程序的起始物理地址调整为 ``0x80400000`` ，三个应用程序都会被加载到这个物理地址上运行；
.. - 将 ``_start`` 所在的 ``.text.entry`` 放在整个程序的开头，也就是说批处理系统只要在加载之后跳转到 ``0x80400000`` 就已经进入了 用户库的入口点，并会在初始化之后跳转到应用程序主逻辑；
.. - 提供了最终生成可执行文件的 ``.bss`` 段的起始和终止地址，方便 ``clear_bss`` 函数使用。

.. 其余的部分和第一章基本相同。

.. _term-call-syscall:

.. 系统调用

System Calls
^^^^^^^^^^^^^^^^^^^^^^

In the submodule ``syscall``, the application program calls the interface provided by the batch operating system through ``ecall``. Since the application program runs in user mode (ie U mode), the ``ecall`` command will trigger the command named *Environment call from U-mode* exception, and trap into the S mode to execute the service code specially provided by the batch operating system for this exception. Since this interface is between the batch processing system in S mode and the application program in U mode, we can know from the previous section that this interface can be called ABI or system call. Now we don't care how the underlying batch operating system provides the functions required by the application, we just use it from the perspective of the application.

In this chapter, the following two system calls are agreed between the application program and the batch system according to the structure of the API:

.. 在子模块 ``syscall`` 中，应用程序通过 ``ecall`` 调用批处理系统提供的接口，由于应用程序运行在用户态（即 U 模式）， ``ecall`` 指令会触发 名为 *Environment call from U-mode* 的异常，并 Trap 进入 S 模式执行批处理系统针对这个异常特别提供的服务代码。由于这个接口处于 S 模式的批处理系统和 U 模式的应用程序之间，从上一节我们可以知道，这个接口可以被称为 ABI 或者系统调用。现在我们不关心底层的批处理系统如何提供应用程序所需的功能，只是站在应用程序的角度去使用即可。

.. 在本章中，应用程序和批处理系统之间按照 API 的结构，约定如下两个系统调用：

.. code-block:: rust
    :caption: 第二章新增系统调用

    /// Function: Write the data in the buffer in the memory to the file.
    /// Parameter: `fd` indicates the file descriptor to be written to the file;
    /// `buf` indicates the starting address of the buffer in memory;
    /// `len` indicates the length of the buffer in memory.
    /// Return value: Returns the length of the successful write.
    /// syscall ID: 64
    fn sys_write(fd: usize, buf: *const u8, len: usize) -> isize;


    /// Function: Exit the application and inform the batch operating system of the return value.
    /// Parameter: `exit_code` indicates the return value of the application.
    /// Return value: This system call should not return.
    /// syscall ID：93
    fn sys_exit(exit_code: usize) -> !;

We know that the system call is actually a binary interface at the assembly instruction level, so what is given here is only the API version described in the Rust language. When actually calling, we need to place the parameters of the system call in the appropriate registers according to the RISC-V calling specification (that is, the ABI format), and then execute the ``ecall`` instruction to trigger the Trap. After Trap returns to the application code in U mode, the execution continues from the next instruction of ``ecall``, and we can read the return value in the appropriate register according to the calling specification.

.. 我们知道系统调用实际上是汇编指令级的二进制接口，因此这里给出的只是使用 Rust 语言描述的 API 版本。在实际调用的时候，我们需要按照 RISC-V 调用规范（即ABI格式）在合适的寄存器中放置系统调用的参数，然后执行 ``ecall`` 指令触发 Trap。在 Trap 回到 U 模式的应用程序代码之后，会从 ``ecall`` 的下一条指令继续执行，同时我们能够按照调用规范在合适的寄存器中读取返回值。

.. note::

    **RISC-V Register numbers and Aliases**

    RISC-V registers are numbered from ``0~31``, expressed as ``x0~x31``. in:

    - ``x10~x17`` : corresponds to ``a0~a7``
    - ``x1``: corresponds to ``ra``

In the RISC-V calling convention, similar to the ABI situation of the function call, it is agreed that the register ``a0~a6`` holds the parameters of the system call, and ``a0`` holds the return value of the system call. The slight difference is that the register ``a7`` is used to pass the syscall ID. This is because all syscalls are triggered by the ``ecall`` instruction. In addition to the input parameters, we also need an additional register to save which system call was requested. Since this is beyond the expressive capabilities of the Rust language, we need to use inline assembly in our code to accomplish parameter/return value binding and ``ecall`` instruction insertion:

.. 在 RISC-V 调用规范中，和函数调用的 ABI 情形类似，约定寄存器 ``a0~a6`` 保存系统调用的参数， ``a0`` 保存系统调用的返回值。有些许不同的是寄存器 ``a7`` 用来传递 syscall ID，这是因为所有的 syscall 都是通过 ``ecall`` 指令触发的，除了各输入参数之外我们还额外需要一个寄存器来保存要请求哪个系统调用。由于这超出了 Rust 语言的表达能力，我们需要在代码中使用内嵌汇编来完成参数/返回值绑定和 ``ecall`` 指令的插入：

.. code-block:: rust
    :linenos:

    // user/src/syscall.rs
    use core::arch::asm;
    fn syscall(id: usize, args: [usize; 3]) -> isize {
        let mut ret: isize;
        unsafe {
            asm!(
                "ecall",
                inlateout("x10") args[0] => ret,
                in("x11") args[1],
                in("x12") args[2],
                in("x17") id
            );
        }
        ret
    }

In line 3, we encapsulate all system calls into ``syscall`` function, you can see that it supports passing in syscall ID and 3 parameters.

``syscall`` uses the ``asm!`` macro starting on line 5 to embed the ``ecall`` instruction to trigger the system call. In the first chapter, we used the ``global_asm!`` macro to embed global assembly code, and the ``asm!`` macro here can embed assembly code into a local function context. Compared with ``global_asm!``, the ``asm!`` macro can obtain variable information in the context and allow embedded assembly code to operate on these variables. Since the compiler is not capable enough to determine the safety of inserting assembly code, we need to wrap it in an unsafe block to take care of it ourselves.

.. 第 3 行，我们将所有的系统调用都封装成 ``syscall`` 函数，可以看到它支持传入 syscall ID 和 3 个参数。

.. ``syscall`` 中使用从第 5 行开始的 ``asm!`` 宏嵌入 ``ecall`` 指令来触发系统调用。在第一章中，我们曾经使用 ``global_asm!`` 宏来嵌入全局汇编代码，而这里的 ``asm!`` 宏可以将汇编代码嵌入到局部的函数上下文中。相比 ``global_asm!`` ， ``asm!`` 宏可以获取上下文中的变量信息并允许嵌入的汇编代码对这些变量进行操作。由于编译器的能力不足以判定插入汇编代码这个行为的安全性，所以我们需要将其包裹在 unsafe 块中自己来对它负责。

From the perspective of RISC-V calling conventions, just like functions have input parameters and return values, ``ecall`` instructions also have input and output registers: ``a0~a2`` and ``a7`` are used as input registers. They respectively indicate the system call parameters and system call ID. And when the system call returns, ``a0`` is used as an output register to save the return value of the system call. In the function context, the input parameter array ``args`` and the variable ``id`` hold the system call parameters and the system call ID, while the variable ``ret`` holds the system call return value, which is also the function ``syscall`` output/return value. These input/output variables can be in one-to-one correspondence with the input/output registers of the ``ecall`` instruction. If we write the assembly code entirely by ourselves, then how to bind variables to registers becomes a problem: for example, before the ``ecall`` instruction is executed, we need to set the value of the register ``a7`` to the variable ``id`` value. Then we first need to know where the value of the current variable ``id`` is stored -- it may be on the stack or in a register.

.. 从 RISC-V 调用规范来看，就像函数有着输入参数和返回值一样， ``ecall`` 指令同样有着输入和输出寄存器： ``a0~a2`` 和 ``a7`` 作为输入寄存器分别表示系统调用参数和系统调用 ID ，而当系统调用返回后， ``a0`` 作为输出寄存器保存系统调用的返回值。在函数上下文中，输入参数数组 ``args`` 和变量 ``id`` 保存系统调用参数和系统调用 ID ，而变量 ``ret`` 保存系统调用返回值，它也是函数 ``syscall`` 的输出/返回值。这些输入/输出变量可以和 ``ecall`` 指令的输入/输出寄存器一一对应。如果完全由我们自己编写汇编代码，那么如何将变量绑定到寄存器则成了一个难题：比如，在 ``ecall`` 指令被执行之前，我们需要将寄存器 ``a7`` 的值设置为变量 ``id`` 的值，那么我们首先需要知道目前变量 ``id`` 的值保存在哪里，它可能在栈上也有可能在某个寄存器中。

As programmers, we don't know the information that only the compiler knows. So we can only complete the binding of variables to registers with the help of the compiler. Now look at the format of the ``asm!`` macro: First, line 6 is the assembly code segment we want to insert. Here we only insert one line of ``ecall`` instruction, but it can support inserting multiple instructions at the same time. From line 7 we bind the input/output variables to registers with the help of the compiler. For example, ``in("x11") args[1]`` in line 8 means to bind the input parameter ``args[1]`` to the input register ``x11`` of ``ecall``, ie` ``a1``. The compiler automatically inserts relevant instructions and ensures that the value of the register ``a1`` is the same as ``args[1]`` before the ``ecall`` instruction is executed. In the same way we can bind the input parameters ``args[2]`` and ``id`` into the input registers ``a2`` and ``a7`` respectively. The special thing here is the ``a0`` register, which is used as both input and output. So we change ``in`` to ``inlateout``, and use ``{in_var} => { out_var}``, where ``{in_var}`` and ``{out_var}`` represent the input and output variables in the context, respectively.

.. 作为程序员我们并不知道这些只有编译器才知道的信息，因此我们只能在编译器的帮助下完成变量到寄存器的绑定。现在来看 ``asm!`` 宏的格式：首先在第 6 行是我们要插入的汇编代码段本身，这里我们只插入一行 ``ecall`` 指令，不过它可以支持同时插入多条指令。从第 7 行开始我们在编译器的帮助下将输入/输出变量绑定到寄存器。比如第 8 行的 ``in("x11") args[1]`` 则表示将输入参数 ``args[1]`` 绑定到 ``ecall`` 的输入寄存器 ``x11`` 即 ``a1`` 中，编译器自动插入相关指令并保证在 ``ecall`` 指令被执行之前寄存器 ``a1`` 的值与 ``args[1]`` 相同。以同样的方式我们可以将输入参数 ``args[2]`` 和 ``id`` 分别绑定到输入寄存器 ``a2`` 和 ``a7`` 中。这里比较特殊的是 ``a0`` 寄存器，它同时作为输入和输出，因此我们将 ``in`` 改成 ``inlateout`` ，并在行末的变量部分使用 ``{in_var} => {out_var}`` 的格式，其中 ``{in_var}`` 和 ``{out_var}`` 分别表示上下文中的输入变量和输出变量。

Sometimes it is not necessary to bind variables to fixed registers, and the ``asm!`` macro can automatically complete the register allocation. Some assembly code segments will also bring some side effects that the compiler cannot predict. In this case, you need to inform the compiler of these possible side effects through ``options`` in ``asm!``. It can help the compiler avoid mistakes and allocate registers more efficiently. In fact, the ``asm!`` macro is far more powerful and easier to use than what we introduce here. Please refer to the Rust-related RFC document [#rust-asm-macro-rfc]_ for details.

The meaning and content of the above assembly code are the same as the assembly instructions in the SBI assembly code of :ref:`RustSBI output to screen from in Chapter 1 <term-llvm-sbicall>`. But the meaning of the parameter-passing register is different. Interested students can review ``console.rs`` and ``sbi.rs`` in the first chapter.

So ``sys_write`` and ``sys_exit`` just need to wrap ``syscall``:

.. 有些时候不必将变量绑定到固定的寄存器，此时 ``asm!`` 宏可以自动完成寄存器分配。某些汇编代码段还会带来一些编译器无法预知的副作用，这种情况下需要在 ``asm!`` 中通过 ``options`` 告知编译器这些可能的副作用，这样可以帮助编译器在避免出错更加高效分配寄存器。事实上， ``asm!`` 宏远比我们这里介绍的更加强大易用，详情参考 Rust 相关 RFC 文档 [#rust-asm-macro-rfc]_ 。

.. 上面这一段汇编代码的含义和内容与 :ref:`第一章中的 RustSBI 输出到屏幕的 SBI 调用汇编代码 <term-llvm-sbicall>` 涉及的汇编指令一样，但传递参数的寄存器的含义是不同的。有兴趣的同学可以回顾第一章的 ``console.rs`` 和 ``sbi.rs`` 。

.. 于是 ``sys_write`` 和 ``sys_exit`` 只需将 ``syscall`` 进行包装：

.. code-block:: rust
    :linenos:

    // user/src/syscall.rs

    const SYSCALL_WRITE: usize = 64;
    const SYSCALL_EXIT: usize = 93;

    pub fn sys_write(fd: usize, buffer: &[u8]) -> isize {
        syscall(SYSCALL_WRITE, [fd, buffer.as_ptr() as usize, buffer.len()])
    }

    pub fn sys_exit(xstate: i32) -> isize {
        syscall(SYSCALL_EXIT, [xstate as usize, 0, 0])
    }

.. _term-fat-pointer:

Note that ``sys_write`` uses a ``&[u8]`` slice type to describe the buffer, which is a **fat pointer** (Fat Pointer), which contains both the starting address and the length of the buffer. We can take them out via the ``as_ptr`` and ``len`` methods and use them independently as actual system call parameters.

We further encapsulate the above two system calls in the user library ``user_lib``, so as to get closer to the actual system call interface on platforms such as Linux:

.. 注意 ``sys_write`` 使用一个 ``&[u8]`` 切片类型来描述缓冲区，这是一个 **胖指针** (Fat Pointer)，里面既包含缓冲区的起始地址，还
.. 包含缓冲区的长度。我们可以分别通过 ``as_ptr`` 和 ``len`` 方法取出它们并独立地作为实际的系统调用参数。

.. 我们将上述两个系统调用在用户库 ``user_lib`` 中进一步封装，从而更加接近在 Linux 等平台的实际系统调用接口：

.. code-block:: rust
    :linenos:

    // user/src/lib.rs
    use syscall::*;

    pub fn write(fd: usize, buf: &[u8]) -> isize { sys_write(fd, buf) }
    pub fn exit(exit_code: i32) -> isize { sys_exit(exit_code) }

We change the ``Stdout::write_str`` in the ``console`` sub-module to an implementation based on ``write``, and set the incoming ``fd`` parameter to 1, which represents the standard output, that is, output to the screen. We don't need to consider other ``fd`` selections for now. Thus, the application's ``println!`` macro becomes available via the system call.

Refer to the code snippet below:

.. 我们把 ``console`` 子模块中 ``Stdout::write_str`` 改成基于 ``write`` 的实现，且传入的 ``fd`` 参数设置为 1，它代表标准输出， .. 也就是输出到屏幕。目前我们不需要考虑其他的 ``fd`` 选取情况。这样，应用程序的 ``println!`` 宏借助系统调用变得可用了。

.. 参考下面的代码片段：

.. code-block:: rust
    :linenos:

    // user/src/console.rs
    const STDOUT: usize = 1;

    impl Write for Stdout {
        fn write_str(&mut self, s: &str) -> fmt::Result {
            write(STDOUT, s.as_bytes());
            Ok(())
        }
    }


The ``exit`` interface is used in ``_start`` in the user library. When the application’s main logic ``main`` returns, use it to exit the application and inform the underlying batch processing system of the return value.

.. ``exit`` 接口则在用户库中的 ``_start`` 内使用，当应用程序主逻辑 ``main`` 返回之后，使用它退出应用程序并将返回值告知 底层的批处理系统。


.. 编译生成应用程序二进制码

Compile and generate application binary code
--------------------------------------------------------------

Here is a brief introduction to the automatic construction of the application. Just run ``make build`` in the ``user`` directory:

1. For each application under ``src/bin``, generate an ELF executable file with the same name under the ``target/riscv64gc-unknown-none-elf/release`` directory;
2. Use the objcopy binary tool to delete all ELF headers and symbols from the ELF file generated in the previous step to obtain a pure binary image file with ``.bin`` suffix. They will be linked into the kernel and loaded into memory by the kernel at the appropriate time.

.. 这里简要介绍一下应用程序的自动构建。只需要在 ``user`` 目录下 ``make build`` 即可：

.. 1. 对于 ``src/bin`` 下的每个应用程序，在 ``target/riscv64gc-unknown-none-elf/release`` 目录下生成一个同名的 ELF 可执行文件；
.. 2. 使用 objcopy 二进制工具将上一步中生成的 ELF 文件删除所有 ELF header 和符号得到 ``.bin`` 后缀的纯二进制镜像文件。它们将被链接进内核并由内核在合适的时机加载到内存。

.. 实现操作系统前执行应用程序

Execute Applications before Implementing an Operating System
---------------------------------------------------------------------- 

We haven't implemented the operating system yet, can we pre-execute or test the application? Can! This is because in addition to a full-system emulator ``qemu-system-riscv64`` that can emulate a RISC-V 64 computer, we also have a semi-system emulator `qemu-riscv64` that directly supports running RISC-V 64 user programs qemu-riscv64. However, it should be noted that if you want the user-mode application to execute on the ``qemu-riscv64`` emulator (actually a Linux operating system under the RISC-V architecture), the effect will be the same as on the OS we wrote, it is necessary to ensure that the interface of the two system calls is the same (including the system call number, the specific register and stack of the parameter agreement, etc.).

.. 我们还没有实现操作系统，能提前执行或测试应用程序吗？可以！这是因为我们除了一个能模拟一台 RISC-V 64 计算机的全系统模拟器 ``qemu-system-riscv64`` 外，还有一个直接支持运行 RISC-V 64 用户程序的半系统模拟器 ``qemu-riscv64`` 。不过需要注意的是，如果想让用户态应用程序在 ``qemu-riscv64`` 模拟器（实际上是一个 RISC-V 架构下的 Linux 操作系统）上和在我们自己写的 OS 上执行效果一样，需要做到二者的系统调用的接口是一样的（包括系统调用编号，参数约定的具体的寄存器和栈等）。

.. note::

    **Qemu's User Mode Emulation and System Level Emulation**

    Qemu has two modes of operation: user mode emulation (User mode) and system level emulation (System mode). In the RISC-V architecture, the user mode simulation can use the ``qemu-riscv64`` simulator, which can simulate a RISC-V computer with a pre-installed Linux operating system. But in general we don't interact with it by entering commands (just like we normally use the Linux operating system), it only supports loading and executing a single executable file. Specifically, it can parse a RISC-V-based application-level ELF executable, load it into memory, and jump to the entry point to start execution. When translating and executing instructions, if it encounters system call-related assembly instructions, it will convert the Linux system calls of different processors (such as RISC-V) to the Linux system on the native processor (such as x86-64), so that the native Linux can complete the system call and return the result (converted into data that RISC-V can recognize) to these applications. In contrast, we use the ``qemu-system-riscv64`` emulator to simulate a RISC-V 64 bare metal at the system level, which includes processors, memory and other peripherals, and supports running a complete operating system.

    .. **Qemu 的用户态模拟和系统级模拟**

    .. Qemu 有两种运行模式：用户态模拟（User mode）和系统级模拟（System mode）。在 RISC-V 架构中，用户态模拟可使用 ``qemu-riscv64`` 模拟器，它可以模拟一台预装了 Linux 操作系统的 RISC-V 计算机。但是一般情况下我们并不通过输入命令来与之交互（就像我们正常使用 Linux 操作系统一样），它仅支持载入并执行单个可执行文件。具体来说，它可以解析基于 RISC-V 的应用级 ELF 可执行文件，加载到内存并跳转到入口点开始执行。在翻译并执行指令时，如果碰到是系统调用相关的汇编指令，它会把不同处理器（如 RISC-V）的 Linux 系统调用转换为本机处理器（如 x86-64）上的 Linux 系统调用，这样就可以让本机 Linux 完成系统调用，并返回结果（再转换成 RISC-V 能识别的数据）给这些应用。相对的，我们使用 ``qemu-system-riscv64`` 模拟器来系统级模拟一台 RISC-V 64 裸机，它包含处理器、内存及其他外部设备，支持运行完整的操作系统。

.. _term-csr-instr-app:

Assuming we've finished compiling and producing an application in ELF executable format, we can try it out. First, let's see what will happen if the application executes :ref:`RV64 S mode privileged command <term-csr-instr>`, the corresponding application can be found in the ``user/src/bin`` directory.

.. 假定我们已经完成了编译并生成了 ELF 可执行文件格式的应用程序，我们就可以来试试。首先看看应用程序执行 :ref:`RV64 的 S 模式特权指令 <term-csr-instr>` 会出现什么情况，对应的应用程序可以在 ``user/src/bin`` 目录下找到。

.. code-block:: rust

    // user/src/bin/03priv_inst.rs
    use core::arch::asm;
    #[no_mangle]
    fn main() -> i32 {
        println!("Try to execute privileged instruction in U Mode");
        println!("Kernel should kill this application!");
        unsafe {
            asm!("sret");
        }
        0
    }

    // user/src/bin/04priv_csr.rs
    use riscv::register::sstatus::{self, SPP};
    #[no_mangle]
    fn main() -> i32 {
        println!("Try to access privileged CSR in U Mode");
        println!("Kernel should kill this application!");
        unsafe {
            sstatus::set_spp(SPP::User);
        }
        0
    }

In the above code, both applications will print a prompt message, and then the application ``03priv_inst`` will try to execute the privileged instruction ``sret`` in the kernel mode in the user mode, and the application ``04priv_csr`` will try to execute the command ``sret`` in the user mode. Modify the kernel mode CSR ``sstatus``.

Next, we try to execute these two applications on the user-mode emulator ``qemu-riscv64``:

.. 在上述代码中，两个应用都会打印提示信息，随后应用 ``03priv_inst`` 会尝试在用户态执行内核态的特权指令 ``sret`` ，而应用 ``04priv_csr`` 则会试图在用户态修改内核态 CSR ``sstatus`` 。

.. 接下来，我们尝试在用户态模拟器 ``qemu-riscv64`` 执行这两个应用：

.. code-block:: console

    $ cd user
    $ make build
    $ cd target/riscv64gc-unknown-none-elf/release/

    # Confirm that the application to be executed is in ELF format
    $ file 03priv_inst
    03priv_inst: ELF 64-bit LSB executable, UCB RISC-V, version 1 (SYSV), statically linked, not stripped

    # Error when executing privileged instruction
    $ qemu-riscv64 ./03priv_inst
    Try to execute privileged instruction in U Mode
    Kernel should kill this application!
    Illegal instruction (core dumped)

    # Error when executing instruction accessing privileged CSR
    $ qemu-riscv64 ./04priv_csr
    Try to access privileged CSR in U Mode
    Kernel should kill this application!
    Illegal instruction (core dumped)

It seems that RV64's privilege level mechanism does work. For general user mode applications, can they be executed correctly under ``qemu-riscv64`` simulator?

.. 看来RV64的特权级机制确实有用。那对于一般的用户态应用程序，在 ``qemu-riscv64`` 模拟器下能正确执行吗？

.. code-block:: console

    $ cd user/target/riscv64gc-unknown-none-elf/release/
    $ qemu-riscv64 ./00hello_world
    Hello, world!
    # correctly output a string
    $ qemu-riscv64 ./01store_fault
    Into Test store_fault, we will insert an invalid store operation...
    Kernel should kill this application!
    Segmentation fault (core dumped)
    # Deliberately accessing an illegal address, causing the application and qemu-riscv64 to be killed by the Linux kernel
    $ qemu-riscv64 ./02power
    3^10000=Segmentation fault (core dumped)
    # fail to work due to version mismatch between Qemu and Rust compilers

It can be seen that, except ``02power``, the other two applications can be executed and ended successfully. This is due to the fact that they are supported at runtime by the operating system Linux for RISC-V 64. The example of ``02power`` also shows that the compatibility of our application is relatively limited. When the application uses more features, it is likely that it will no longer be compatible with Qemu. We expect the Devonian "Dunkleosteus" operating system, which we begin to implement in the next section, to also load and execute these applications correctly.

.. 可以看到，除了 ``02power`` 之外，其余两个应用程序都能够执行并顺利结束。这是由于它们在运行时得到了操作系统 Linux for RISC-V 64 的支持。而 ``02power`` 的例子也说明我们应用的兼容性比较受限，当应用用到较多特性时很可能就不再兼容 Qemu 了。我们期望在下一节开始实现的泥盆纪“邓式鱼”操作系统也能够正确加载和执行这些应用程序。

.. [#rust-asm-macro-rfc] https://doc.rust-lang.org/reference/inline-assembly.html
