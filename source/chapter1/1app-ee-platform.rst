.. 应用程序执行环境与平台支持

Application Execution Environment and Platform Support
=========================================================

.. toctree::
   :hidden:
   :maxdepth: 5


.. 本节导读

Section Guide
-------------------------------

.. 本节介绍了如何设计实现一个提供显示字符服务的用户态执行环境和裸机执行环境，以支持一个应用程序显示字符串。显示字符服务的裸机执行环境和用户态执行环境向下直接或间接与硬件关联，向上可通过函数库给应用提供 **显示字符** 的服务。这也说明了不管执行环境是简单还是复杂，设计实现上是否容易，它都体现了不同操作系统的共性特征--给应用需求提供服务。在某种程度上看，执行环境的软件主体虽然是以函数库的形态存在，但可称为是一种操作系统。

In this section, we start with the simplest Rust application, dig into the multi-layer execution environment under the hood. We analyze how the compiler and operating system provide convenience for the development and operation of the application. The core lesson is that the LibOS operating system is also a layer execution environment -- it needs to provide services for the upper layer applications. We will come into the most core idea in computer science - abstraction. And at the same time we take the real applications of different demand levels as examples to analyze how to make a reasonable abstraction. Finally, we cover some basics of hardware and software platforms, including the RISC-V architecture.


.. 本节我们从一个最简单的 Rust 应用程序入手，深入地挖掘它下面的多层执行环境，分析编译器和操作系统为应用程序的开发和运行提供了怎样的便利条件。主要注意额是，LibOS操作系统也是一层执行环境，因此它需要为上层的应用程序提供服务。我们会接触到计算机科学中最核心的思想 -- 抽象，同时以现实中不同需求层级的应用为例分析如何进行合理的抽象。最后，我们还会介绍软硬件平台（包括 RISC-V 架构）的一些基础知识。

.. 执行应用程序

Run an Application
-------------------------------

Let's first develop and run a simple "Hello, world" application on Linux, and see the whole process of a simple application from development to execution. To get started, let's create a Rust project using the Cargo tool. It doesn't look like anything special:

.. 我们先在Linux上开发并运行一个简单的 “Hello, world” 应用程序，看看一个简单应用程序从开发到执行的全过程。作为一切的开始，让我们使用 Cargo 工具来创建一个 Rust 项目。它看上去没有任何特别之处：

.. code-block:: console

   $ cargo new os --bin

We add the ``--bin`` option to tell Cargo that we create an executable project instead of a library project. At this point, the file structure of the project is as follows:

.. 我们加上了 ``--bin`` 选项来告诉 Cargo 我们创建一个可执行程序项目而不是函数库项目。此时，项目的文件结构如下：

.. code-block:: console
   
   $ tree os
   os
   ├── Cargo.toml
   └── src
       └── main.rs

   1 directory, 2 files

Among them, ``Cargo.toml`` holds the configuration of the project, including the author's information, contact information, library dependencies, and so on. Obviously the source code is stored in the ``src`` directory, so far there is only one file ``main.rs``, let’s take a look at the contents:

.. 其中 ``Cargo.toml`` 中保存着项目的配置，包括作者的信息、联系方式以及库依赖等等。显而易见源代码保存在 ``src`` 目录下，目前为止只有 ``main.rs`` 一个文件，让我们看一下里面的内容：

.. code-block:: rust
   :linenos:
   :caption: the simpluest Rust application

   fn main() {
       println!("Hello, world!");
   }

Enter the root directory of the os project, use the Cargo tool to build and run the project with one command:

.. 进入 os 项目根目录下，利用 Cargo 工具即可一条命令实现构建并运行项目：

.. code-block:: console
   
   $ cargo run
      Compiling os v0.1.0 (/home/shinbokuow/workspace/v3/rCore-Tutorial-v3/os)
       Finished dev [unoptimized + debuginfo] target(s) in 1.15s
        Running `target/debug/os`
   Hello, world!
 
As we expected, we see a line ``Hello, world!`` on the screen. However, it needs to be noted that the convenience of programming and executing programs we enjoy is not taken for granted, and there are various mechanisms behind it, from hardware to software. Especially for the operation of the application program, a powerful execution environment is needed to help. Next, we will look at the powerful execution environment supported by the operating system.

.. 如我们预想的一样，我们在屏幕上看到了一行 ``Hello, world!`` 。但是，需要注意到我们所享受到的编程和执行程序的方便性并不是理所当然的，背后有着从硬件到软件多种机制的支持。特别是对于应用程序的运行，需要有一个强大的执行环境来帮助。接下来，我们就要看看有操作系统加持的强大的执行环境。

.. 应用程序执行环境

Application Runtime Environment
---------------------------------

As shown in the figure below, the operation of applications on general-purpose operating systems (such as Linux, etc.) now requires the support of the multi-level execution environment stack below. The up-to-bottom white blocks in the figure (the lower, the closer to the bottom layer. The lower layer is the execution environment of the upper, which supports the operation of the upper layer code. ) represents the execution environment at all levels, and the black block represents the interface between the execution environments of two adjacent layers.

.. 如下图所示，现在通用操作系统（如 Linux 等）上的应用程序运行需要下面多层次的执行环境栈的支持，图中的白色块自上而下（越往下则越靠近底层，下层作为上层的执行环境支持上层代码的运行）表示各级执行环境，黑色块则表示相邻两层执行环境之间的接口。

.. _app-software-stack:

.. figure:: app-software-stack.png
   :align: center

   Application Runtime Environment Stack

   .. 应用程序执行环境栈


.. _term-execution-environment:

Our application is at the top layer, and it can complete complex functions with only a small amount of source code by calling the standard library provided by the programming language or the function interface provided by other third-party libraries. But the functions of these libraries are not limited to this. In fact they are part of the application **Execution Environment**. Where we usually don't notice, these software libraries also complete some initialization work before executing the application, and monitor the application while it is executing. The ``println!`` macro we use to print ``Hello, world!`` is provided by the Rust standard library std.


.. 我们的应用位于最上层，它可以通过调用编程语言提供的标准库或者其他三方库对外提供的函数接口，使得仅需少量的源代码就能完成复杂的功能。但是这些库的功能不仅限于此，事实上它们属于应用程序 **执行环境** (Execution Environment)的一部分。在我们通常不会注意到的地方，这些软件库还会在执行应用之前完成一些初始化工作，并在应用程序执行的时候对它进行监控。我们在打印 ``Hello, world!`` 时使用的 ``println!`` 宏正是由 Rust 标准库 std提供的。

.. _term-system-call:

From the perspective of the operating system kernel, everything on it belongs to user mode software, and it itself belongs to kernel mode software. Regardless of how the user-mode application is written, whether it is handwritten assembly code, or a standard library or a third-party library based on a high-level programming language, some functions must be directly or indirectly provided by the operating system kernel **system call** (System Call) to achieve. System calls thus act as a boundary between the user and the kernel. As the execution environment of user-mode software, the kernel not only provides a system call interface, but also needs to monitor and manage the execution of user-mode software.


.. 从操作系统内核的角度看来，它上面的一切都属于用户态软件，而它自身属于内核态软件。无论用户态应用如何编写，是手写汇编代码，还是基于某种高级编程语言调用其标准库或三方库，某些功能总要直接或间接的通过操作系统内核提供的 **系统调用** (System Call) 来实现。因此系统调用充当了用户和内核之间的边界。内核作为用户态软件的执行环境，它不仅要提供系统调用接口，还需要对用户态软件的执行进行监控和管理。

.. note::

   **Hello, world! Which system calls are used?**

   From the output of the previous ``cargo run``, it can be seen that the previously built executable file is os in the target/debug directory. On the Ubuntu system, you can use the ``strace`` tool to run a program and output all system calls and return values ​​requested from the kernel during the running of the program. We can simply type ``strace target/debug/os`` to see a long list of various system calls.

   Among them, it is easy to see that there are only two system calls related to the actual execution of the ``Hello, world!`` application:
   
   .. 从之前的 ``cargo run`` 的输出可以看出之前构建的可执行文件是在 target/debug 目录下的 os 。在 Ubuntu 系统上，可以通过 ``strace`` 工具来运行一个程序并输出程序运行过程当中向内核请求的所有的系统调用及其返回值。我们只需输入 ``strace target/debug/os`` 即可看到一长串的各种系统调用。

   .. 其中，容易看出与 ``Hello, world!`` 应用实际执行相关的只有两个系统调用：

   .. code-block::

      # output a string
      write(1, "Hello, world!\n", 14)         = 14
      # exit the function execution
      exit_group(0)

   .. .. code-block::

   ..    # 输出字符串
   ..    write(1, "Hello, world!\n", 14)         = 14
   ..    # 程序退出执行
   ..    exit_group(0)

   We will not explain the specific meaning of its parameters here.

   The rest of the system calls are basically used to initialize the two-layer execution environment of the function library and the kernel, and to monitor and manage the operation of the upper-layer applications. Later, as the application scenarios become more complex, the operating system needs to provide stronger hardware abstraction capabilities and resource management capabilities, and implement some related system calls.

   .. 其参数的具体含义我们暂且不在这里进行解释。

   .. 其余的系统调用基本上分别用于函数库和内核两层执行环境的初始化工作和对上层应用的运行进行监控和管理。之后，随着应用场景的复杂化，操作系统需要提供更强的硬件抽象能力和资源管理能力，并实现相关的一些系统调用。   

.. _term-isa:

From a hardware point of view, everything on it belongs to software. Hardware can be divided into three types: processor (also called CPU), memory and I/O devices. Among them, the processor is undoubtedly the most complex, but also the most critical one. It agrees with the software on a set of **Instruction Set Architecture** (ISA, Instruction Set Architecture), so that the software can access various hardware resources through the machine instructions provided in the ISA. Of course, the software also needs to know how the processor will execute these instructions, and the results of the instructions after execution. Of course, the actual situation is far more complicated than this. In order to adapt to the scenario of modern applications, the processor needs to provide many additional mechanisms (such as privilege level, page table, TLB, exception/interrupt response, etc.) to manage applications execution process, rather than just letting data flow between CPU registers, memory, and I/O devices.


.. 从硬件的角度来看，它上面的一切都属于软件。硬件可以分为三种： 处理器 (Processor，也称CPU)，内存 (Memory) 还有 I/O 设备。其中处理器无疑是其中最复杂，同时也最关键的一个。它与软件约定一套 **指令集体系结构** (ISA, Instruction Set Architecture)，使得软件可以通过 ISA 中提供的机器指令来访问各种硬件资源。软件当然也需要知道处理器会如何执行这些指令，以及指令执行后的结果。当然，实际的情况远比这个要复杂得多，为了适应现代应用程序的场景，处理器还需要提供很多额外的机制（如特权级、页表、TLB、异常/中断响应等）来管理应用程序的执行过程，而不仅仅是让数据在 CPU 寄存器、内存和 I/O 设备三者之间流动。

.. note::

    All problems in computer science can be solved by another level of indirection。

    -- David Wheeler


..     计算机科学中遇到的所有问题都可通过增加一层抽象来解决。

..     All problems in computer science can be solved by another level of indirection。

..     -- 计算机科学家 David Wheeler


.. _term-abstraction:

.. note::

   .. **多层执行环境都是必需的吗？**

   **Are all multi-layer execution environments compulsory?**

   In addition to the top-level application program and the bottom-level hardware platform must exist, the function library and operating system kernel as the middle layer do not have to exist:
   They all abstract the underlying resources (Abstraction/Indirection) and provide an execution environment for the upper layer (also can be understood as some service functions). The advantage of abstraction is that it allows the upper layer to obtain the required functions at a small cost, and at the same time it can provide some protection. But abstraction is also a limitation, and some flexibility will be lost. For example, when you are considering which function library you should use in your project, you often need to weigh this aspect: too much abstraction and too little abstraction are naturally inappropriate. It is also important to understand the requirements of the application. An operating system design that can reasonably meet application requirements is a problem that operating system designers need to consider deeply. This is also a trade-off. Too many service functions and too few service functions are naturally inappropriate.

   .. 除了最上层的应用程序和最下层的硬件平台必须存在之外，作为中间层的函数库和操作系统内核并不是必须存在的：
   .. 它们都是对下层资源进行了 **抽象** (Abstraction/Indirection)，并为上层提供了一个执行环境（也可理解为一些服务功能）。抽象的优点在于它让上层以较小的代价获得所需的功能，并同时可以提供一些保护。但抽象同时也是一种限制，会丧失一些应有的灵活性。比如，当你在考虑在项目中应该使用哪个函数库的时候，就常常需要这方面的权衡：过多的抽象和过少的抽象自然都是不合适的。理解应用的需求也很重要。一个能合理满足应用需求的操作系统设计是操作系统设计者需要深入考虑的问题。这也是一种权衡，过多的服务功能和过少的服务功能自然都是不合适的。


   In fact, we use the characteristics and needs of the application to judge what level of abstraction and functionality the operating system requires.

   - If the function library and the operating system kernel do not exist, then we need to write assembly code to control the hardware. This method has the highest flexibility and the lowest abstraction ability, which is basically equivalent to writing assembly code to directly control the hardware. We usually use this method to implement some architecture-related small modules or code fragments that cannot be described only by high-level programming languages.
   - If there is only a function library and no operating system kernel, it means that we don't need the overly general abstraction provided by the operating system kernel. This often happens in embedded scenarios with a single function. Although embedded devices also include processors, memory and I/O devices, usually only one or a few small applications with very simple functions will run on them at the same time, such as timing display, real-time data collection, face recognition punching system, etc. A common solution is to build a single application using only the function library or to manage a small number of applications with a lightweight function library specially tailored for the application scenario. This requires only one layer of execution environment in the form of a function library.
   - If there are function libraries and operating system kernels, this means that the application requirements are more diverse and will require concurrent execution. Common general-purpose operating systems such as Windows/Linux/macOS all support running multiple different applications concurrently. To do this requires more powerful operating system abstraction and functionality, which will require a multi-layer execution environment.



   .. 实际上，我们通过应用程序的特征和需求来判断操作系统需要什么程度的抽象和功能。

   .. - 如果函数库和操作系统内核都不存在，那么我们就需要手写汇编代码来控制硬件，这种方式具有最高的灵活性，抽象能力则最低，基本等同于编写汇编代码来直接控制硬件。我们通常用这种方式来实现一些架构相关且仅通过高级编程语言无法描述的小模块或者代码片段。
   .. - 如果仅存在函数库而不存在操作系统内核，意味着我们不需要操作系统内核提供过于通用的抽象。在那些功能单一的嵌入式场景就常常会出现这种情况。嵌入式设备虽然也包含处理器、内存和 I/O 设备，但是它上面通常只会同时运行一个或几个功能非常简单的小应用程序，比如定时显示、实时采集数据、人脸识别打卡系统等。常见的解决方案是仅使用函数库构建单独的应用程序或是用专为应用场景特别裁减过的轻量级函数库管理少数应用程序。这就只需要一层函数库形态的执行环境。
   .. - 如果存在函数库和操作系统内核，这意味着应用需求比较多样，会需要并发执行。常见的通用操作系统如 Windows/Linux/macOS 等都支持并发运行多个不同的应用程序。为此需要更加强大的操作系统抽象和功能，也就会需要多层执行环境。

.. note::

   .. **“用力过猛”的现代操作系统**

   **Modern operating systems that "overdrive"**


   We can see execution environment support provided by modern operating systems "overstretched" for simpler small applications like this:

   .. 对于如下更简单的小应用程序，我们可以看到“用力过猛”的现代操作系统提供的执行环境支持：

   .. code-block:: rust
      :linenos:
      
      //ch1/donothing.rs
      fn main()  {
         //do nothing
      }
   
   For this program, its presence is almost imperceptible during its execution. After compiling and running, what you can see is:

   .. 对于这个程序，在它的执行过程中，几乎感知不到它存在感。在编译后运行，可以看到的情况是：

   .. code-block:: console

      $ rustc donothing.rs
      $ ./donothing
      $ (no output)

   As we expected, after the program is executed, there is no obvious output information, and the program ends very quickly. But if you re-execute the program through `strace`, a tool that monitors the system call request of the program, you can see a lot of dynamic information during the execution of the program:

   .. 与我们预计一样，程序执行后，看不到有何明显的输出信息，程序非常快地就结束了。但如果通过监视程序系统调用请求的工具 `strace` 来重新执行一下这个程序，就可以看到程序执行过程中的大量动态信息：
   
   .. code-block:: console

      $ strace ./donothing
         (As many as 93 lines of output, indicating that donothing issued 93 various system calls to the Linux operating system kernel)
         execve("./donothing", ["./donothing"], 0x7ffe02c9ca10 /* 67 vars */) = 0
         brk(NULL)                               = 0x563ba0532000
         arch_prctl(0x3001 /* ARCH_??? */, 0x7fff2da54360) = -1 EINVAL (invalid parameter)
         ......

   This shows that modern operating systems, such as `Linux`, implement a large number of functions for the sake of generality. But for very simple programs, there are many functions that are redundant.

   .. 这说明了现在的操作系统，如 `Linux` ，为了通用性，而实现了大量的功能。但对于非常简单的程序而言，有很多的功能是多余的。

.. 目标平台与目标三元组

Target platform and target triplet
---------------------------------------

.. _term-platform:

.. note::

   The main workflow of a modern compiler toolset (using a C or Rust compiler as an example) is as follows:
   
   1. Source code --> preprocessor --> Source code with expanded macro
   2. Source code with expanded macro --> compiler --> Assembling code
   3. Assembling code --> assembler --> Object code
   4. Object code --> linker --> executable

   .. 现代编译器工具集（以C或Rust编译器为例）的主要工作流程如下：
   
   .. 1. 源代码（source code） --> 预处理器（preprocessor） --> 宏展开的源代码
   .. 2. 宏展开的源代码 --> 编译器（compiler） --> 汇编程序
   .. 3. 汇编程序 --> 汇编器（assembler）--> 目标代码（object code）
   .. 4. 目标代码 --> 链接器（linker） --> 可执行文件（executables）


For an application source code implemented in a certain programming language, the compiler needs to know which **Platform** (Platform) the program will run on when compiling and linking it to obtain an executable file. The platform here mainly refers to the combination of CPU type, operating system type and standard runtime library. From the :ref:`application execution environment stack <app-software-stack>` given above, it can be seen that:

- If the user mode is based on different kernels, the system call interface will be different or the semantics will be inconsistent;
- If the underlying hardware is different, the way to access hardware resources will be different. Especially if the ISA is different, the instruction set and registers presented to the software are different.

They all lead to very different final executables. It should be pointed out that some compilers support that the same source code can be compiled to multiple different target platforms and run on them without modification. In this case, the source code is **cross-platform**. Other compilers have been preset for a fixed target platform.


.. 对于一份用某种编程语言实现的应用程序源代码而言，编译器在将其通过编译、链接得到可执行文件的时候需要知道程序要在哪个 **平台** (Platform) 上运行。这里平台主要是指 CPU 类型、操作系统类型和标准运行时库的组合。从上面给出的 :ref:`应用程序执行环境栈 <app-software-stack>` 可以看出：

- 如果用户态基于的内核不同，会导致系统调用接口不同或者语义不一致；
- 如果底层硬件不同，对于硬件资源的访问方式会有差异。特别是如果 ISA 不同，则向软件提供的指令集和寄存器都不同。

它们都会导致最终生成的可执行文件有很大不同。需要指出的是，某些编译器支持同一份源代码无需修改就可编译到多个不同的目标平台并在上面运行。这种情况下，源代码是 **跨平台** 的。而另一些编译器则已经预设好了一个固定的目标平台。

.. _term-target-triplet:

The Rust compiler uses **Target Triplet**  to describe the target platform on which a software runs. It generally includes information such as the CPU, operating system, and runtime library, thereby controlling the executable code generation of the Rust compiler. For example, we can try to see what the target platform of the previous ``Hello, world!`` is. This can be done by printing the default configuration information for the compiler rustc:

.. Rust编译器通过 **目标三元组** (Target Triplet) 来描述一个软件运行的目标平台。它一般包括 CPU、操作系统和运行时库等信息，从而控制Rust编译器可执行代码生成。比如，我们可以尝试看一下之前的 ``Hello, world!`` 的目标平台是什么。这可以通过打印编译器 rustc 的默认配置信息：

.. code-block:: console

   $ rustc --version --verbose
      rustc 1.57.0-nightly (e1e9319d9 2021-10-14)
      binary: rustc
      commit-hash: e1e9319d93aea755c444c8f8ff863b0936d7a4b6
      commit-date: 2021-10-14
      host: x86_64-unknown-linux-gnu
      release: 1.57.0-nightly
      LLVM version: 13.0.0

From the host item, we can see that the default target platform is ``x86_64-unknown-linux-gnu``, where the CPU architecture is x86_64, the CPU manufacturer is unknown, the operating system is linux, and the runtime library is GNU libc (encapsulated Linux system calls, and provide a POSIX interface-based function library). This is the simplest and most common case that both the compiler and the compiler-generated executable run on the same platform. But soon we will confront another issue. 

.. 从其中的 host 一项可以看出默认的目标平台是 ``x86_64-unknown-linux-gnu``，其中 CPU 架构是 x86_64，CPU 厂商是 unknown，操作系统是 linux，运行时库是 GNU libc（封装了 Linux 系统调用，并提供 POSIX 接口为主的函数库）。这种无论编译器还是其生成的可执行文件都在我们当前所处的平台运行是一种最简单也最普遍的情况。但是很快我们就将遇到另外一种情况。

After talking so much, it's finally time to introduce our main mission. We want to be able to run ``Hello, world!`` on another hardware platform, and the difference from the previous default platform is that we change the CPU architecture from x86_64 to RISC-V.

.. 讲了这么多，终于该介绍我们的主线任务了。我们希望能够在另一个硬件平台上运行 ``Hello, world!``，而与之前的默认平台不同的地方在于，我们将 CPU 架构从 x86_64 换成 RISC-V。

.. chyyuu note::
   **Why based on RISC-V architecture instead of x86 family architecture? **
   In order to maintain compatibility with applications/kernels based on older architectures while upgrading, the x86 architecture has a lot of historical burden -- some features no longer meaningful for the current applications. But they take long time to set up in order for CPU to function properly. This is indeed essential in order to establish and maintain the application ecology of the architecture. But for teaching, it is almost a waste of time. The nascent RISC-V architecture is very concise, and the core part of the architecture document is less than a hundred pages. Their provided functionalities are enough to support a concise, abstract and runnable kernel. 

   .. **为何基于 RISC-V 架构而非 x86 系列架构？**
   .. x86 架构为了在升级换代的同时保持对基于旧版架构应用程序/内核的兼容性，存在大量的历史包袱，也就是一些对于目前的应用场景没有任何意义，但又必须花大量时间正确设置才能正常使用 CPU 的奇怪设定。为了建立并维护架构的应用生态，这确实是必不可少的，但站在教学的角度几乎是在浪费时间。而新生的 RISC-V 架构十分简洁，架构文档需要阅读的核心部分不足百页，且这些功能已经足以用来构造一个具有相当抽象能力且可以运行的简洁内核了。

You can take a look at which RISC-V-based target platforms are currently supported by the Rust compiler:

.. 可以看一下目前 Rust 编译器支持哪些基于 RISC-V 的目标平台：

.. code-block:: console

   $ rustc --print target-list | grep riscv
   riscv32gc-unknown-linux-gnu
   riscv32i-unknown-none-elf
   riscv32imac-unknown-none-elf
   riscv32imc-unknown-none-elf
   riscv64gc-unknown-linux-gnu
   riscv64gc-unknown-none-elf
   riscv64imac-unknown-none-elf

Here we choose the ``riscv64gc-unknown-none-elf`` target platform. Among them, the CPU architecture is `riscv64gc`, the CPU manufacturer is `unknown`, the operating system is `none`, and `elf` means that there is no standard runtime library (indicating that there is no support for system calls). But it allows to generate ELF-formatted program. The reason why we do not choose the target platform ``riscv64gc-unknown-linux-gnu`` with the linux-gnu system call support here is because we just want to run a ``Hello, world!`` application running on a bare metal environment program. There is no need to use the high-level abstraction and redundant operating system services provided by the Linux operating system. And we are very clear that what we are going to develop in the future is an operating system kernel, which must directly face the underlying physical hardware (bare-metal) to provide slim operating system services. Many system call services provided by general-purpose operating systems such as Linux are redundant to this kernel.


.. 这里我们选择 ``riscv64gc-unknown-none-elf`` 目标平台。这其中的 CPU 架构是 `riscv64gc` ，CPU厂商是 `unknown` ，操作系统是 `none` ， `elf` 表示没有标准的运行时库（表明没有任何系统调用的封装支持），但可以生成 ELF 格式的执行程序。这里我们之所以不选择有 linux-gnu 系统调用支持的目标平台 ``riscv64gc-unknown-linux-gnu``，是因为我们只是想跑一个在裸机环境上运行的 ``Hello, world!`` 应用程序，没有必要使用Linux操作系统提供的那么高级的抽象和多余的操作系统服务。而且我们很清楚后续我们要开发的是一个操作系统内核，它必须直面底层物理硬件（bare-metal）来提供精简的操作系统服务功能，通用操作系统（如 Linux）提供的很多系统调用服务对这个内核而言是多余的。

.. note::

   **RISC-V instruction set extension**

   Since processors based on RISC-V architecture may be used in embedded scenarios or general-purpose computing scenarios, the instruction set specification divides the instruction set into the most basic RV32/64I and several standard instruction set extensions. Each processor only needs to expand the instruction set as needed according to its actual application scenario.

   - RV32/64I: The basic integer instruction set that every processor must implement. In RV32I, each general-purpose register is 32 bits wide; in RV64I, it is 64 bits. It can be used to simulate instructions in most standard instruction set extensions, except for the special A extension, which requires special hardware support.
   - M extension: Provide instructions related to integer multiplication and division.
   - A extension: provide atomic instructions and some related memory synchronization mechanisms, which will be expanded later.
   - F/D extension: Provide single/double precision floating point arithmetic support.
   - C Extensions: Provides compressed instruction extensions.

   G extension is the general term for the basic integer instruction set I plus the standard instruction set extension MAFD, so riscv64gc is equivalent to riscv64imafdc. The rest of our content is based on this processor architecture. In addition, the RISC-V architecture has many standard instruction set extensions, some of which are still being updated and are not yet stable. Interested students can refer to the latest version of the RISC-V instruction set specification.

   .. **RISC-V 指令集拓展**

   .. 由于基于 RISC-V 架构的处理器可能用于嵌入式场景或是通用计算场景，因此指令集规范将指令集划分为最基本的 RV32/64I 以及若干标准指令集拓展。每款处理器只需按照其实际应用场景按需实现指令集拓展即可。

   .. - RV32/64I：每款处理器都必须实现的基本整数指令集。在 RV32I 中，每个通用寄存器的位宽为 32 位；在 RV64I 中则为 64 位。它可以用来模拟绝大多数标准指令集拓展中的指令，除了比较特殊的 A 拓展，因为它需要特别的硬件支持。
   .. - M 拓展：提供整数乘除法相关指令。
   .. - A 拓展：提供原子指令和一些相关的内存同步机制，这个后面会展开。
   .. - F/D 拓展：提供单/双精度浮点数运算支持。
   .. - C 拓展：提供压缩指令拓展。

   .. G 拓展是基本整数指令集 I 再加上标准指令集拓展 MAFD 的总称，因此 riscv64gc 也就等同于 riscv64imafdc。我们剩下的内容都基于该处理器架构完成。除此之外 RISC-V 架构还有很多标准指令集拓展，有一些还在持续更新中尚未稳定，有兴趣的同学可以浏览最新版的 RISC-V 指令集规范。

.. Rust 标准库与核心库

Rust Standard and Core Libraries
----------------------------------

Let's try changing the target platform of the current ``Hello, world!`` program to riscv64gc-unknown-none-elf and see what happens:

.. 我们尝试一下将当前的 ``Hello, world!`` 程序的目标平台换成 riscv64gc-unknown-none-elf 看看会发生什么事情：

.. code-block:: console
   
   $ cargo run --target riscv64gc-unknown-none-elf
      Compiling os v0.1.0 (/home/shinbokuow/workspace/v3/rCore-Tutorial-v3/os)
   error[E0463]: can't find crate for `std`
     |
     = note: the `riscv64gc-unknown-none-elf` target may not be installed

.. _term-bare-metal:

In the previous development environment setup, we have installed this target platform support in the rustup toolchain. So it is not a problem that the target platform is not installed. This question simply means that the Rust standard library std cannot be found on this target platform. We have mentioned before that some functions of the standard library or third-party library of the programming language will directly or indirectly use the system calls provided by the operating system. But currently there is no operating system support for our chosen target platform, so Rust does not support the full standard library std for this target platform. Platforms like this are often referred to as **bare-metal platforms** (bare-metal). This means that software on bare-metal platforms has no traditional operating system support.

.. 在之前的开发环境配置中，我们已经在 rustup 工具链中安装了这个目标平台支持，因此并不是该目标平台未安装的问题。这个问题只是单纯的表示在这个目标平台上找不到 Rust 标准库 std。我们之前曾经提到过，编程语言的标准库或三方库的某些功能会直接或间接的用到操作系统提供的系统调用。但目前我们所选的目标平台不存在任何操作系统支持，于是 Rust 并没有为这个目标平台支持完整的标准库 std。类似这样的平台通常被我们称为 **裸机平台** (bare-metal)。这意味着在裸机平台上的软件没有传统操作系统支持。

.. note::

   **Rust Tips: Rust language standard library std and core library core**

   The Rust language standard library--std is the basis for portability of software developed in the Rust language, similar to the LibC standard library for the C language. It's a small set of tried-and-tested shared abstractions for development in the wider Rust ecosystem. It provides core types such as Vec and Option, language primitive operations defined by the class library, standard macros, I/O and multithreading, etc. By default, we can use the Rust language standard library to support the development of Rust applications. But one limitation of the Rust language standard library is that it requires operating system support. Therefore, if the software you want to implement is an operating system running on bare metal, you cannot directly use the Rust language standard library.

   .. **Rust Tips：Rust语言标准库std和核心库core**

   .. Rust 语言标准库--std 是让 Rust 语言开发的软件具备可移植性的基础，类似于 C 语言的 LibC 标准库。它是一组小巧的、经过实践检验的共享抽象，适用于更广泛的 Rust 生态系统开发。它提供了核心类型，如 Vec 和 Option、类库定义的语言原语操作、标准宏、I/O 和多线程等。默认情况下，我们可以使用 Rust 语言标准库来支持 Rust 应用程序的开发。但 Rust 语言标准库的一个限制是，它需要有操作系统的支持。所以，如果你要实现的软件是运行在裸机上的操作系统，就不能直接用 Rust 语言标准库了。

Fortunately, Rust has a Rust language core library core that has been tailored to the Rust language standard library --std. The core library does not require any operating system support, and its functions are relatively limited. But it also includes a considerable part of the core mechanism of the Rust language, which can meet most of our functional requirements. The Rust language is a system-oriented (including operating system) development language, so in the Rust language ecosystem, there are many third-party libraries that do not depend on the standard library std but only on the core library "core". Using them can greatly reduce our programming burden. They are the most important thing we can rely on to survive on bare metal platforms, and they are also necessary for most of the Rust embedded software that runs without operating system support.

So, we know that on the bare metal platform we need to replace the standard library std with the core library "core". But in practice, there are still some trivial things that need to be solved.

.. 幸运的是，Rust 有一个对 Rust 语言标准库--std 裁剪过后的 Rust 语言核心库 core。core库是不需要任何操作系统支持的，它的功能也比较受限，但是也包含了 Rust 语言相当一部分的核心机制，可以满足我们的大部分功能需求。Rust 语言是一种面向系统（包括操作系统）开发的语言，所以在 Rust 语言生态中，有很多三方库也不依赖标准库 std 而仅仅依赖核心库 core。对它们的使用可以很大程度上减轻我们的编程负担。它们是我们能够在裸机平台挣扎求生的最主要倚仗，也是大部分运行在没有操作系统支持的 Rust 嵌入式软件的必备。

.. 于是，我们知道在裸机平台上我们要将对于标准库 std 的引用换成核心库 core。但是实际做起来其实还要有一些琐碎的事情需要解决。