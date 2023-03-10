.. 内核第一条指令（基础篇）

The First Instruction of Kernel (Basic)
===========================================================================================

.. toctree::
   :hidden:
   :maxdepth: 5

.. 本节导读

Section Guide
--------------------------------------

In the next two sections, we will proceed to the second step of building the "Trilobite" operating system, which is to correctly load our kernel into the Qemu emulator, so that the Qemu emulator can successfully execute the first instruction of the kernel. In this section we'll cover some of the basics, starting with the various hardware components of a computer, specifically the CPU and physical memory. Secondly, we introduce the abstract model, usage method and startup process of the Qemu emulator. As a knowledge supplement, we also introduced the general computer startup process. Finally, we introduced the program memory layout and compilation process, especially the knowledge about linking. These basic knowledge can help us better understand the motivation and purpose behind the practical operation in the next section.

.. 接下来两节我们将进行构建“三叶虫”操作系统的第二步，即将我们的内核正确加载到 Qemu 模拟器上，使得 Qemu 模拟器可以成功执行内核的第一条指令。本节我们将介绍一些相关基础知识，首先介绍计算机的各个硬件组成部分，特别是 CPU 和物理内存。其次介绍 Qemu 模拟器的抽象模型、使用方法以及启动流程。作为知识补充，我们还介绍了一般计算机的启动流程。最后我们介绍了程序内存布局和编译流程，特别是链接的相关知识。这些基本知识可以帮助我们更好的理解下一节的实践操作背后的动机和目的。

.. 计算机组成基础

Basics of Computer Composition
--------------------------------------

When writing an application, in most cases we only need to call library functions to achieve various functions with the support of the operating system, without caring about how the operating system schedules and manages various hardware and software resources. The operating system provides some monitoring tools (such as the task manager on Windows or the ``ps`` tool on Linux), which can help us count the usage of resources such as CPU, memory, hard disk, and network. So that we can roughly Learn about the usage of these resources and help us better develop or deploy applications. However, when actually writing an operating system, we must face these hardware resources, manage them and provide efficient and easy-to-use abstractions for applications. To do this, we must improve our understanding of these hardware.

.. 当编写应用程序的时候，大多数情况下我们只需调用库函数即可在操作系统的支持下实现各项功能，而无需关心操作系统如何调度管理各类软硬件资源。操作系统提供了一些监控工具（如 Windows 上的任务管理器或 Linux 上的 ``ps`` 工具），这些工具可以帮助我们统计 CPU、内存、硬盘、网络等资源的占用情况，从而让我们大致上了解这些资源的使用情况，并帮助我们更好地开发或部署应用程序。然而，在实际编写操作系统的时候，我们就必须直面这些硬件资源，将它们管理起来并为应用程序提供高效易用的抽象。为此，我们必须增进对于这些硬件的了解。

.. _term-physical-address:

The computer is mainly composed of three parts: processor (also known as central processing unit, CPU), physical memory and I/O peripherals. In the first eight chapters we mainly used CPU and physical memory. The main function of the processor is to read instructions from physical memory, decode and execute them, and deal with physical memory and I/O peripherals in the process. Physical memory is an important part of computer architecture. In terms of storage, the only thing the CPU can directly access is data in physical memory, which it can do through memory fetch instructions. From the perspective of the CPU, physical memory can be seen as a large byte array, and the physical address corresponds to a subscript that can be used to access an element in the array. Different from our daily programming habits, the subscript usually does not start with 0, but usually starts with a constant, such as ``0x80000000``. In short, the CPU can address a data location by a physical address and access data in physical memory **byte by byte**.

.. 计算机主要由处理器（Processor，也即中央处理器，CPU，Central Processing Unit），物理内存和 I/O 外设三部分组成。在前八章我们主要用到 CPU 和物理内存。处理器的主要功能是从物理内存中读取指令、译码并执行，在此过程中还要与物理内存和 I/O 外设打交道。物理内存则是计算机体系结构中一个重要的组成部分。在存储方面，CPU 唯一能够直接访问的只有物理内存中的数据，它可以通过访存指令来达到这一目的。从 CPU 的视角看来，可以将物理内存看成一个大字节数组，而物理地址则对应于一个能够用来访问数组中某个元素的下标。与我们日常编程习惯不同的是，该下标通常不以 0 开头，而通常以一个常数，如 ``0x80000000`` 开头。简言之，CPU 可以通过物理地址来寻址，并 **逐字节** 地访问物理内存中保存的数据。

It is worth mentioning that when the CPU accesses physical memory in units of multiple bytes (such as 2/4/8 or more) (in fact, it is not limited to physical memory, but also includes the data space of I/O peripherals), it may introduce endian (also known as byte order) and memory address alignment problems. Since this is not the key point, we will not explain it here. If readers are interested, they can refer to the supplementary explanation below.

.. 值得一提的是，当 CPU 以多个字节（比如 2/4/8 或更多）为单位访问物理内存（事实上并不局限于物理内存，也包括I/O外设的数据空间）中的数据时，就有可能会引入端序（也称字节顺序）和内存地址对齐的问题。由于这并不是重点，我们在这里不展开说明，如读者有兴趣可以参考下面的补充说明。

.. note::
 
   **Endian**
  
    Endianness or endianness, also known as byte order. In the field of computer science, it refers to the arrangement order of bytes (Byte) of multi-byte words (Word) in computer memory or in digital communication links. There are two general rules for how bytes are arranged. For example, if the low bits of a multi-digit number are placed at a smaller address and the high bits are placed at a larger address, it is called little-endian; otherwise, it is called big-endian. Common x86, RISC-V and other architectures use little endian.

   ..  **端序或尾序**
  
   ..  端序或尾序（Endianness），又称字节顺序。在计算机科学领域中，指电脑内存中或在数字通信链路中，多字节组成的字（Word）的字节（Byte）的排列顺序。字节的排列方式有两个通用规则。例如，将一个多位数的低位放在较小的地址处，高位放在较大的地址处，则称小端序（little-endian）；反之则称大端序（big-endian）。常见的 x86、RISC-V 等架构采用的是小端序。
 
.. note::
 
   **Memory address alignment**
  
    Memory address alignment is the arrangement of data in memory and the way the CPU accesses memory data, including two parts: basic data alignment and structure data alignment. The CPU reads and writes data in the memory by byte blocks. In theory, any type of variable access can start from any address in the memory. But in the computer system, the CPU accesses the memory through the data bus (, which determines the number of data bits per read) and address bus (, which determines the scope of address space). Based on the physical composition and performance requirements of the computer, CPU generally requires the value of the first address to be an multiple of 4 or 8 when accessing the memory data. 

    The basic type of data alignment means that the offset address of the data in the memory must be a  multiple of a word. This way of storing data can improve the performance of the system when reading data. Structure data alignment refers to filling some useless bytes at the end of the previous data field and the beginning of the next data field in the structure to ensure that each data field (assumed to be basic type data) can be aligned (i.e. Aligned by primitive type data).

    For RISC-V processors, when the load/store instruction performs data access, the address of the data in memory should be aligned. If accessing 32-bit data, the memory address should be 32-bit (4-byte) aligned. If the address of the data is not aligned, an exception will be generated when performing a memory access operation. This is also a bug that is often encountered in learning kernel programming.

   ..  **内存地址对齐**
  
   ..  内存地址对齐是内存中的数据排列，以及 CPU 访问内存数据的方式，包含了基本数据对齐和结构体数据对齐的两部分。CPU 在内存中读写数据是按字节块进行操作，理论上任意类型的变量访问可以从内存的任何地址开始，但在计算机系统中，CPU 访问内存是通过数据总线（决定了每次读取的数据位数）和地址总线（决定了寻址范围）来进行的，基于计算机的物理组成和性能需求，CPU 一般会要求访问内存数据的首地址的值为 4 或 8 的整数倍。 

   ..  基本类型数据对齐是指数据在内存中的偏移地址必须为一个字的整数倍，这种存储数据的方式，可以提升系统在读取数据时的性能。结构体数据对齐，是指在结构体中的上一个数据域结束和下一个数据域开始的地方填充一些无用的字节，以保证每个数据域（假定是基本类型数据）都能够对齐（即按基本类型数据对齐）。

   ..  对于 RISC-V 处理器而言，load/store 指令进行数据访存时，数据在内存中的地址应该对齐。如果访存 32 位数据，内存地址应当按 32 位（4字节）对齐。如果数据的地址没有对齐，执行访存操作将产生异常。这也是在学习内核编程中经常碰到的一种 bug。


.. 了解 Qemu 模拟器

Learn about the Qemu emulator
--------------------------------------

The kernel we wrote will mainly run on the Qemu emulator to verify its correctness. This is mainly for convenience and quickness. You only need to enter a line of commands on the command line to make the kernel run. In order to allow our kernel to be correctly connected to the Qemu emulator, we must first have a certain understanding of the Qemu emulator. In this book, we use the software ``qemu-system-riscv64`` to simulate a 64-bit RISC-V architecture computer, which includes a CPU, physical memory, and several I/O peripherals. Its specific configuration (such as the number of CPU cores or the size of physical memory) can be adjusted by the user through the execution parameter options of Qemu. As an emulator, it is just a user program on a host machine, so the resources mentioned above are all simulated by using the resources provided to it by the host (that is, the platform where Qemu runs, such as Linux/Windows/macOS). The performance of some hardware resources simulated on Qemu is very high, even close to the performance of native resources on the host machine; while the simulation overhead of other hardware resources is relatively high, resulting in relatively slow overall hardware performance simulated by Qemu. But the hardware performance emulated by Qemu on current x86-64 processors is fast enough for the various operating systems practiced in this book.

Next let's see how to start Qemu. As you can see from the ``os/Makefile`` in the code of each chapter, we use the following commands to start Qemu and run our kernel:


.. 我们编写的内核将主要在 Qemu 模拟器上运行来检验其正确性。这样做主要是为了方便快捷，只需在命令行输入一行命令即可让内核跑起来。为了让我们的内核能够正确对接到 Qemu 模拟器上，我们首先要对 Qemu 模拟器有一定的了解。在本书中，我们使用软件 ``qemu-system-riscv64`` 来模拟一台 64 位 RISC-V 架构的计算机，它包含CPU 、物理内存以及若干 I/O 外设。它的具体配置（比如 CPU 的核数或是物理内存的大小）均可由用户通过Qemu的执行参数选项来调整。作为模拟器，在宿主机看来它只是一个用户程序，因此上面提到的资源都是它利用宿主机（即 Qemu 运行所在的平台，如 Linux/Windows/macOS）提供给它的资源模拟出来的。在 Qemu 上模拟出来的某些硬件资源性能很高，甚至接近宿主机上原生资源的性能；而另一些硬件资源的模拟开销较大，从而导致Qemu模拟的整体硬件性能相对较慢。但对于本书所实践的各种操作系统而言，在当前x86-64处理器上的Qemu所模拟的硬件性能已经足够快了。

.. 接下来我们来看如何启动 Qemu 。从各章节代码中的 ``os/Makefile`` 可以看到，我们使用如下命令来启动 Qemu 并运行我们的内核：

.. code-block:: console
   :linenos:

   $ qemu-system-riscv64 \
       -machine virt \
       -nographic \
       -bios ../bootloader/rustsbi-qemu.bin \
       -device loader,file=target/riscv64gc-unknown-none-elf/release/os.bin,addr=0x80200000

The meanings of each execution parameter option are as follows:

.. 其中各个执行参数选项的含义如下：

.. _term-bootloader:

- ``-machine virt`` means to set up the simulated 64-bit RISC-V machine as a virtual machine named ``virt``. We know that even if they belong to the same instruction set architecture, there will be many different computer configurations, such as different manufacturers and models of CPUs, and different types of I/O peripherals supported. See [#virt_platform]_ for more information on the ``virt`` platform. Qemu also supports emulation of other RISC-V computers, including the famous HiFive Unleashed development board produced by SiFive.
- ``-nographic`` indicates that the emulator does not need to provide a graphical interface, but only needs to output a character stream externally.
- Through ``-bios``, you can set the bootloader used for initialization when the Qemu emulator is turned on. Here we use the pre-compiled ``rustsbi-qemu.bin``, which needs to be placed in the  ``bootloader`` directory at the same level as ``os``. The directory is available from the code branch of each chapter.
- Through the ``loader`` attribute in the virtual device ``-device``, you can load a file on the host machine into the specified location of Qemu’s physical memory before the Qemu emulator starts. The ``file`` and ``addr`` attribute can set the path of the file to be loaded and the physical address of the Qemu physical memory where the file is loaded. Note that the file we loaded here has a ``.bin`` suffix. It is not the kernel executable file that we built after removing the standard library dependencies in the previous section, but it needs to be processed to get the kernel image. We'll discuss that in depth later.


.. - ``-machine virt`` 表示将模拟的 64 位 RISC-V 计算机设置为名为 ``virt`` 的虚拟计算机。我们知道，即使同属同一种指令集架构，也会有很多种不同的计算机配置，比如 CPU 的生产厂商和型号不同，支持的 I/O 外设种类也不同。关于 ``virt`` 平台的更多信息可以参考 [#virt_platform]_ 。Qemu 还支持模拟其他 RISC-V 计算机，其中包括由 SiFive 公司生产的著名的 HiFive Unleashed 开发板。
.. - ``-nographic`` 表示模拟器不需要提供图形界面，而只需要对外输出字符流。
.. - 通过 ``-bios`` 可以设置 Qemu 模拟器开机时用来初始化的引导加载程序（bootloader），这里我们使用预编译好的 ``rustsbi-qemu.bin`` ，它需要被放在与 ``os`` 同级的 ``bootloader`` 目录下，该目录可以从每一章的代码分支中获得。
.. - 通过虚拟设备 ``-device`` 中的 ``loader`` 属性可以在 Qemu 模拟器开机之前将一个宿主机上的文件载入到 Qemu 的物理内存的指定位置中， ``file`` 和 ``addr`` 属性分别可以设置待载入文件的路径以及将文件载入到的 Qemu 物理内存上的物理地址。注意这里我们载入的文件带有 ``.bin`` 后缀，它并不是上一节中我们移除标准库依赖后构建得到的内核可执行文件，而是还要进行加工处理得到内核镜像。我们后面再进行深入讨论。

.. Qemu 启动流程

Qemu startup process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _term-physical-memory:

On the ``virt`` hardware platform simulated by Qemu, the starting physical address of the physical memory is ``0x80000000``. And the default size of the physical memory is 128MiB, which can be configured by the ``-m`` option. In this book, we will only use the lowest 8MiB of physical memory, and the corresponding physical address range is ``[0x80000000,0x80800000)``. If you start Qemu with the command given above, before Qemu starts to execute any instructions, first load two files into Qemu's physical memory: that is, load ``rustsbi-qemu.bin`` as a bootloader to the physical memory area starting with the physical address ``0x80000000``. And at the same time, it loads the kernel image ``os.bin`` to the area starting with the physical address ``0x80200000``.

.. 在Qemu模拟的 ``virt`` 硬件平台上，物理内存的起始物理地址为 ``0x80000000`` ，物理内存的默认大小为 128MiB ，它可以通过 ``-m`` 选项进行配置。在本书中，我们只会用到最低的 8MiB 物理内存，对应的物理地址区间为 ``[0x80000000,0x80800000)`` 。如果使用上面给出的命令启动 Qemu ，那么在 Qemu 开始执行任何指令之前，首先把两个文件加载到 Qemu 的物理内存中：即作把作为 bootloader 的 ``rustsbi-qemu.bin`` 加载到物理内存以物理地址 ``0x80000000`` 开头的区域上，同时把内核镜像 ``os.bin`` 加载到以物理地址 ``0x80200000`` 开头的区域上。

Why are they loaded in these two locations? This is related to the running process of the Qemu simulated computer after it is powered on. Generally speaking, the startup process after the computer is powered on can be divided into several stages, and each stage is in charge of a layer of software or :ref:`firmware<term-firmware>`. The function of each layer of software or firmware is to perform the initialization work it should undertake, and then jump to the entry address of the next layer of software or firmware.  Or equivalent to say, it hands over the control of the computer to the next layer of software or firmware. The startup process of the Qemu simulation can be divided into three stages: the first stage by a small assembler program solidified in Qemu, the second stage by the bootloader, and the third stage by the kernel image.

.. 为什么加载到这两个位置呢？这与 Qemu 模拟计算机加电启动后的运行流程有关。一般来说，计算机加电之后的启动流程可以分成若干个阶段，每个阶段均由一层软件或 :ref:`固件 <term-firmware>` 负责，每一层软件或固件的功能是进行它应当承担的初始化工作，并在此之后跳转到下一层软件或固件的入口地址，也就是将计算机的控制权移交给了下一层软件或固件。Qemu 模拟的启动流程则可以分为三个阶段：第一个阶段由固化在 Qemu 内的一小段汇编程序负责；第二个阶段由 bootloader 负责；第三个阶段则由内核镜像负责。

- The first stage: After loading the necessary files into the Qemu physical memory, the program counter (PC, Program Counter) of the Qemu CPU will be initialized to ``0x1000``. So the first instruction actually executed by Qemu is located at the physical address ``0x1000``. Then it will execute a few instructions and jump to the instruction corresponding to the physical address ``0x80000000`` and enter the second stage. It can be seen from the subsequent debugging process that the address ``0x80000000`` is solidified in Qemu. As a user of Qemu, we cannot change it without touching the source code of Qemu.
- The second stage: Since the first stage of Qemu fixedly jumps to ``0x80000000``, we need to put the bootloader ``rustsbi-qemu.bin`` responsible for the second stage at the beginning of the physical address ``0x80000000``. In this way, it can ensure that ``0x80000000`` just saves the first instruction of the bootloader. At this stage, the bootloader is responsible for some initialization work on the computer, and jumps to the entry of the next-stage software. On Qemu, the control of the computer can be handed over to our kernel image ``os.bin``. It should be noted here that for different bootloaders, the entry of the next stage software is not necessarily the same, and the method and time point of obtaining this information are also different: The entry address may be a pre-agreed fixed value, or it may be a value that is dynamically obtained during the bootloader operation.  The RustSBI we choose pre-appoints the entry address of the next stage as a fixed ``0x80200000``. After the initialization of RustSBI is completed, it will jump to this address and hand over the control of the computer to the next stage software - aka our kernel image.
- The third stage: In order to correctly interface with the previous stage of RustSBI, we need to ensure that the first instruction of the kernel is located at the physical address ``0x80200000``. To do this, we need to preload the kernel image onto the area of ​​Qemu physical memory starting with address ``0x80200000``. Once the CPU starts executing the first instruction of the kernel, it proves that control of the computer has been handed over to our kernel, and the goal of this section is achieved.


.. - 第一阶段：将必要的文件载入到 Qemu 物理内存之后，Qemu CPU 的程序计数器（PC, Program Counter）会被初始化为 ``0x1000`` ，因此 Qemu 实际执行的第一条指令位于物理地址 ``0x1000`` ，接下来它将执行寥寥数条指令并跳转到物理地址 ``0x80000000`` 对应的指令处并进入第二阶段。从后面的调试过程可以看出，该地址 ``0x80000000`` 被固化在 Qemu 中，作为 Qemu 的使用者，我们在不触及 Qemu 源代码的情况下无法进行更改。
.. - 第二阶段：由于 Qemu 的第一阶段固定跳转到 ``0x80000000`` ，我们需要将负责第二阶段的 bootloader ``rustsbi-qemu.bin`` 放在以物理地址 ``0x80000000`` 开头的物理内存中，这样就能保证 ``0x80000000`` 处正好保存 bootloader 的第一条指令。在这一阶段，bootloader 负责对计算机进行一些初始化工作，并跳转到下一阶段软件的入口，在 Qemu 上即可实现将计算机控制权移交给我们的内核镜像 ``os.bin`` 。这里需要注意的是，对于不同的 bootloader 而言，下一阶段软件的入口不一定相同，而且获取这一信息的方式和时间点也不同：入口地址可能是一个预先约定好的固定的值，也有可能是在 bootloader 运行期间才动态获取到的值。我们选用的 RustSBI 则是将下一阶段的入口地址预先约定为固定的 ``0x80200000`` ，在 RustSBI 的初始化工作完成之后，它会跳转到该地址并将计算机控制权移交给下一阶段的软件——也即我们的内核镜像。
.. - 第三阶段：为了正确地和上一阶段的 RustSBI 对接，我们需要保证内核的第一条指令位于物理地址 ``0x80200000`` 处。为此，我们需要将内核镜像预先加载到 Qemu 物理内存以地址 ``0x80200000`` 开头的区域上。一旦 CPU 开始执行内核的第一条指令，证明计算机的控制权已经被移交给我们的内核，也就达到了本节的目标。

.. note::

   **The power-on process of a real computer**

   The startup process of a real computer can also be roughly divided into three stages:

	.. **真实计算机的加电启动流程**

	.. 真实计算机的启动流程大致上也可以分为三个阶段：

.. _term-firmware:

   - The first stage: After power-on, the PC register of the CPU is set to the physical address of the computer's internal read-only memory (ROM, Read-only Memory), and then the CPU starts to run the software in the ROM. We generally call this software firmware (Firmware). Its function is to perform some initialization operations on the CPU, load the code and data of the bootloader in the subsequent stage from the hard disk to the physical memory, and finally jump to the appropriate address to control the computer. Rights are transferred to the bootloader. It roughly corresponds to the first phase of Qemu startup, a few instructions placed at physical address ``0x1000``. It can be seen that the firmware on Qemu is very simple, because it does not need to be responsible for loading the bootloader from the hard disk into physical memory. This task has been completed by Qemu itself before.
   - The second stage: The bootloader also completes some CPU initialization work, loads the operating system image from the hard disk into the physical memory, and finally jumps to the appropriate address to transfer control to the operating system. It can be seen that in general, the bootloader needs to complete some data loading work, which is the source of the loader in its name. It corresponds to the second phase of Qemu startup. In Qemu, the RustSBI we use is weak, and it has no ability to complete the loading work. The kernel image is actually loaded into physical memory together with the bootloader before Qemu starts.
   - Phase 3: Control is transferred to the operating system. Due to space limitations, we won't go into details.

   It is worth mentioning that in order to make the startup of the computer more flexible, the bootloader may be very complicated at present: it may also be divided into multiple stages, and it can manage some hardware resources. In terms of complexity, it is close to a traditional operating system .


	.. - 第一阶段：加电后 CPU 的 PC 寄存器被设置为计算机内部只读存储器（ROM，Read-only Memory）的物理地址，随后 CPU 开始运行 ROM 内的软件。我们一般将该软件称为固件（Firmware），它的功能是对 CPU 进行一些初始化操作，将后续阶段的 bootloader 的代码、数据从硬盘载入到物理内存，最后跳转到适当的地址将计算机控制权转移给 bootloader 。它大致对应于 Qemu 启动的第一阶段，即在物理地址 ``0x1000`` 处放置的若干条指令。可以看到 Qemu 上的固件非常简单，因为它并不需要负责将 bootloader 从硬盘加载到物理内存中，这个任务此前已经由 Qemu 自身完成了。
	.. - 第二阶段：bootloader 同样完成一些 CPU 的初始化工作，将操作系统镜像从硬盘加载到物理内存中，最后跳转到适当地址将控制权转移给操作系统。可以看到一般情况下 bootloader 需要完成一些数据加载工作，这也就是它名字中 loader 的来源。它对应于 Qemu 启动的第二阶段。在 Qemu 中，我们使用的 RustSBI 功能较弱，它并没有能力完成加载的工作，内核镜像实际上是和 bootloader 一起在 Qemu 启动之前加载到物理内存中的。
	.. - 第三阶段：控制权被转移给操作系统。由于篇幅所限后面我们就不再赘述了。

	.. 值得一提的是，为了让计算机的启动更加灵活，bootloader 目前可能非常复杂：它可能也分为多个阶段，并且能管理一些硬件资源，从复杂性上它已接近一个传统意义上的操作系统。

Based on the above introduction to the Qemu startup process, we can know that in order for our kernel image to be correctly connected to Qemu and RustSBI, the kernel image file we submit to Qemu must satisfy: the beginning of the file is the first kernel to be executed an instruction. But as will be mentioned later, the executable file obtained by removing the standard library dependency in the previous section does not actually meet this condition. Therefore, we also need to do some manipulation on the executable to get a kernel image that can be submitted to Qemu. To illustrate these conditions, we first need to know a little about the program's memory layout and compilation flow.

.. 基于上面对 Qemu 启动流程的介绍，我们可以知道为了让我们的内核镜像能够正确对接到 Qemu 和 RustSBI 上，我们提交给 Qemu 的内核镜像文件必须满足：该文件的开头即为内核待执行的第一条指令。但后面会讲到，在上一节中我们通过移除标准库依赖得到的可执行文件实际上并不满足该条件。因此，我们还需要对可执行文件进行一些操作才能得到可提交给 Qemu 的内核镜像。为了说明这些条件，首先我们需要了解一些关于程序内存布局和编译流程的知识。

.. 程序内存布局与编译流程

Program Memory Layout and Compilation Process
--------------------------------------------------

.. 程序内存布局

Program Memory Layout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _term-section:
.. _term-memory-layout:

After we compile the source code into an executable, it turns out to be a seemingly random file full of bytes. But we know that these bytes can be divided into at least two parts: code and data. When the program is running, their functions are not the same: the code part is composed of instructions that can be decoded and executed by the CPU, while the data part is only viewed by the CPU as a readable and writable memory space. In fact, we can further divide the two parts into smaller units according to their functions: **section**. Different section swill be placed in different locations in memory by the compiler, which constitutes the program's **Memory Layout** (Memory Layout). A typical program relative memory layout is as follows:

.. 在我们将源代码编译为可执行文件之后，它就会变成一个看似充满了杂乱无章的字节的一个文件。但我们知道这些字节至少可以分成代码和数据两部分，在程序运行起来的时候它们的功能并不相同：代码部分由一条条可以被 CPU 解码并执行的指令组成，而数据部分只是被 CPU 视作可读写的内存空间。事实上我们还可以根据其功能进一步把两个部分划分为更小的单位： **段** (Section) 。不同的段会被编译器放置在内存不同的位置上，这构成了程序的 **内存布局** (Memory Layout)。一种典型的程序相对内存布局如下所示：

.. figure:: MemoryLayout.png
   :align: center

   A typical program relative memory layout

   .. 一种典型的程序相对内存布局

As can be seen in the figure above, the code section only has the code segment ``.text``, which stores all the assembly codes of the program. The data part can further be broken down: 

.. 在上图中可以看到，代码部分只有代码段 ``.text`` 一个段，存放程序的所有汇编代码。而数据部分则还可以继续细化：

.. _term-heap:

- The initialized data segment holds the initialized global data in the program, and is divided into two parts ``.rodata`` and ``.data``. The former stores read-only global data, usually some constants or Constant strings, etc.; and the latter stores modifiable global data.
- The uninitialized data segment ``.bss`` saves the uninitialized global data in the program, and is usually zero-initialized by the program loader, that is, this area is cleared byte by byte;
- The **heap** (heap) area is used to store data dynamically allocated when the program is running. For example, the data body allocated by malloc/new in C/C++ is placed in the heap area, and it grows to a high address;
- The **stack** (stack) area is not only used to save and restore the function call context, but the local variables in the scope of each function are also placed in its stack frame by the compiler, and it grows towards the lower address.

.. - 已初始化数据段保存程序中那些已初始化的全局数据，分为 ``.rodata`` 和 ``.data`` 两部分。前者存放只读的全局数据，通常是一些常数或者是
..   常量字符串等；而后者存放可修改的全局数据。
.. - 未初始化数据段 ``.bss`` 保存程序中那些未初始化的全局数据，通常由程序的加载者代为进行零初始化，即将这块区域逐字节清零；
.. - **堆** （heap）区域用来存放程序运行时动态分配的数据，如 C/C++ 中的 malloc/new 分配到的数据本体就放在堆区域，它向高地址增长；
.. - **栈** （stack）区域不仅用作函数调用上下文的保存与恢复，每个函数作用域内的局部变量也被编译器放在它的栈帧内，它向低地址增长。

.. note::

   **local variables and global variables**

   From the perspective of a function, its accessible variables include the following:
   
   - The input parameters and local variables of the function: stored in some registers or the stack frame of the function. If it is in the stack frame, it is accessed based on the current stack pointer plus an offset;
   - Global variables: stored in the data segment ``.data`` and ``.bss``, in some cases the gp(x3) register holds a position between the two data segments. So the global variable is based on gp plus an offset to access.
   - Dynamic variables on the heap: they are stored on the heap, and the size can only be determined at runtime. And we can only *directly* access variables of **compile-time determined size** on the stack or in the global data segment. Therefore, we need to access it through a pointer to the data on the heap obtained by allocating memory at runtime. The bit width of the pointer can indeed be determined at compile time. The pointer can be placed in the stack frame as a local variable, or placed in the global data segment as a global variable.


   .. **局部变量与全局变量**

   .. 在一个函数的视角中，它能够访问的变量包括以下几种：
   
   .. - 函数的输入参数和局部变量：保存在一些寄存器或是该函数的栈帧里面，如果是在栈帧里面的话是基于当前栈指针加上一个偏移量来访问的；
   .. - 全局变量：保存在数据段 ``.data`` 和 ``.bss`` 中，某些情况下 gp(x3) 寄存器保存两个数据段中间的一个位置，于是全局变量是基于 gp 加上一个偏移量来访问的。
   .. - 堆上的动态变量：本体被保存在堆上，大小在运行时才能确定。而我们只能 *直接* 访问栈上或者全局数据段中的 **编译期确定大小** 的变量。因此我们需要通过一个运行时分配内存得到的一个指向堆上数据的指针来访问它，指针的位宽确实在编译期就能够确定。该指针即可以作为局部变量放在栈帧里面，也可以作为全局变量放在全局数据段中。

.. 编译流程

Compilation process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The process of compiling an executable from source can be broken down into stages (although typing a single command can do them all):

.. 从源代码得到可执行文件的编译流程可被细化为多个阶段（虽然输入一条命令便可将它们全部完成）：

.. _term-compiler:
.. _term-assembler:
.. _term-linker:
.. _term-object-file:

1. **Compiler** converts each source file from a high-level programming language to assembly language. Note that the source file is still an ASCII or other encoded text file at this time;
2. **Assembler** (Assembler) converts the text-format instructions in each source file in the previous step into machine code, and obtains a binary **object file** (Object File);
3. **Linker** (Linker) links all the object files obtained in the previous step and some possible external object files together to form a complete executable file.

.. 1. **编译器** (Compiler) 将每个源文件从某门高级编程语言转化为汇编语言，注意此时源文件仍然是一个 ASCII 或其他编码的文本文件；
.. 2. **汇编器** (Assembler) 将上一步的每个源文件中的文本格式的指令转化为机器码，得到一个二进制的 **目标文件** (Object File)；
.. 3. **链接器** (Linker) 将上一步得到的所有目标文件以及一些可能的外部目标文件链接在一起形成一个完整的可执行文件。

Each object file output by the assembler has an independent program memory layout, which describes where the sections are located within the object file. What the linker does is to integrate all input object files into an overall memory layout. During this period, the linker mainly completes two tasks:

.. 汇编器输出的每个目标文件都有一个独立的程序内存布局，它描述了目标文件内各段所在的位置。而链接器所做的事情是将所有输入的目标文件整合成一个整体的内存布局。在此期间链接器主要完成两件事情：

- The first task is to rearrange the segments from different object files in the object memory layout. As shown in the figure below, during the linking process, the sections from the target file ``1.o`` and ``2.o`` are classified according to the functions of the sections, and the sections with the same function are arranged together in the assembly in the following object file ``output.o``. Note that the memory layouts of the object files ``1.o`` and ``2.o`` are in conflict, and the same address stores different contents in different memory layouts. In the merged memory layout, these conflicts are resolved. 

.. - 第一件事情是将来自不同目标文件的段在目标内存布局中重新排布。如下图所示，在链接过程中，分别来自于目标文件 ``1.o`` 和 ``2.o`` 段被按照段的功能进行分类，相同功能的段被排在一起放在拼装后的目标文件 ``output.o`` 中。注意到，目标文件 ``1.o`` 和 ``2.o`` 的内存布局是存在冲突的，同一个地址在不同的内存布局中存放不同的内容。而在合并后的内存布局中，这些冲突被消除。

.. figure:: link-sections.png
   :align: center

   Rearrangement of sections from different object files

   .. 来自不同目标文件的段的重新排布

- The second thing is to replace the symbols with specific addresses. What do the symbols here refer to? We know that when we do modular programming, each module will provide some global variables, functions, etc. exposed to other modules for other modules to access, and will also access the content exposed to it by other modules. To access a variable or call a function, at the source code level we only need to know their names, which we call symbols. Depending on whether the symbol comes from within the module or from another module, we can further divide symbols into internal symbols and external symbols. However, at the machine code level (that is, in an object file or executable) we do not index the variable or function we want to access through symbols, but directly through the address of the variable or function. For example, if we want to call a function, then in the machine code of the instruction we can find the absolute address of the function entry or the relative address relative to the current PC.

So, when is the symbol replaced with a specific address? Because the variable or function corresponding to the symbol is placed in a fixed position in a certain section (for example, global variables are often placed in the ``.bss`` or ``.data`` section, while functions are placed in ``.text`` segment). So we need to wait until the symbol-holding segments determine their locations in the memory layout, before we can know these symbols' exact addresses. When a module is converted into an object file, its internal symbols have been converted into specific addresses in the object file. It is because the object file gives the memory layout of the module. 
It means the location of each segment in the module has been identified. However, the addresses of external symbols used by the module cannot be determined at this time. We need to record these external symbols and put them in an area called the symbol table in the target file. Since subsequent relocation may be needed, internal symbols also need to be recorded in the symbol table.

External symbols need to wait to be converted into specific addresses until the time of linking. Assume that module 1 uses the content provided by module 2. When the object files of the two modules are linked together, their memory layouts will be merged. It means that the positions of each segment of the two modules are determined. At this point, the external symbols from module 2 used by module 1 can be translated into specific addresses. At the same time, we also need to pay attention: the segments of the two modules are rearranged in the merged memory layout, and their final positions may have changed compared with their positions in the local memory layout of the module itself. Therefore, the address of the internal symbol of each module may also change, and we also need to correct it. The above process is called Relocation, and this process looks a bit like a puzzle -- since module 1 uses the content of module 2, the two are equivalent to a concave and convex puzzle respectively, due to the way we stitch them together. 


.. - 第二件事情是将符号替换为具体地址。这里的符号指什么呢？我们知道，在我们进行模块化编程的时候，每个模块都会提供一些向其他模块公开的全局变量、函数等供其他模块访问，也会访问其他模块向它公开的内容。要访问一个变量或者调用一个函数，在源代码级别我们只需知道它们的名字即可，这些名字被我们称为符号。取决于符号来自于模块内部还是其他模块，我们还可以进一步将符号分成内部符号和外部符号。然而，在机器码级别（也即在目标文件或可执行文件中）我们并不是通过符号来找到索引我们想要访问的变量或函数，而是直接通过变量或函数的地址。例如，如果想调用一个函数，那么在指令的机器码中我们可以找到函数入口的绝对地址或者相对于当前 PC 的相对地址。

  那么，符号何时被替换为具体地址呢？因为符号对应的变量或函数都是放在某个段里面的固定位置（如全局变量往往放在 ``.bss`` 或者 ``.data`` 段中，而函数则放在 ``.text`` 段中），所以我们需要等待符号所在的段确定了它们在内存布局中的位置之后才能知道它们确切的地址。当一个模块被转化为目标文件之后，它的内部符号就已经在目标文件中被转化为具体的地址了，因为目标文件给出了模块的内存布局，也就意味着模块内的各个段的位置已经被确定了。然而，此时模块所用到的外部符号的地址无法确定。我们需要将这些外部符号记录下来，放在目标文件一个名为符号表（Symbol table）的区域内。由于后续可能还需要重定位，内部符号也同样需要被记录在符号表中。

  外部符号需要等到链接的时候才能被转化为具体地址。假设模块 1 用到了模块 2 提供的内容，当两个模块的目标文件链接到一起的时候，它们的内存布局会被合并，也就意味着两个模块的各个段的位置均被确定下来。此时，模块 1 用到的来自模块 2 的外部符号可以被转化为具体地址。同时我们还需要注意：两个模块的段在合并后的内存布局中被重新排布，其最终的位置有可能和它们在模块自身的局部内存布局中的位置相比已经发生了变化。因此，每个模块的内部符号的地址也有可能会发生变化，我们也需要进行修正。上面的过程被称为重定位（Relocation），这个过程形象一些来说很像拼图：由于模块 1 用到了模块 2 的内容，因此二者分别相当于一块凹进和凸出一部分的拼图，正因如此我们可以将它们无缝地拼接到一起。

Above we briefly introduced the knowledge about program memory layout and compilation process, especially the linking process. So how to get a kernel image that can run successfully on Qemu? First of all, we need to adjust the memory layout of the kernel executable file through the link script. So that the first instruction executed by the kernel is located at address ``0x80200000``, and the address of the code segment should be lower than other segments. This is because the area of ​​Qemu physical memory below ``0x80200000`` is not allocated to the kernel, but is mostly used by RustSBI. Secondly, we need to discard the metadata in the kernel executable file to get the kernel image, which only contains the code and data that will actually be used. This is because Qemu's loading function is too simple and direct. It directly copies the input file byte by byte into physical memory, so it can also be said that this step is to help Qemu manually load the executable file into physical memory. In the next section we will successfully build the kernel image and verify on Qemu that control is transferred to the kernel.

.. 上面我们简单介绍了程序内存布局和编译流程特别是链接过程的相关知识。那么如何得到一个能够在 Qemu 上成功运行的内核镜像呢？首先我们需要通过链接脚本调整内核可执行文件的内存布局，使得内核被执行的第一条指令位于地址 ``0x80200000`` 处，同时代码段所在的地址应低于其他段。这是因为 Qemu 物理内存中低于 ``0x80200000`` 的区域并未分配给内核，而是主要由 RustSBI 使用。其次，我们需要将内核可执行文件中的元数据丢掉得到内核镜像，此内核镜像仅包含实际会用到的代码和数据。这则是因为 Qemu 的加载功能过于简单直接，它直接将输入的文件逐字节拷贝到物理内存中，因此也可以说这一步是我们在帮助 Qemu 手动将可执行文件加载到物理内存中。下一节我们将成功生成内核镜像并在 Qemu 上验证控制权被转移到内核。

.. [#virt_platform] https://www.qemu.org/docs/master/system/riscv/virt.html