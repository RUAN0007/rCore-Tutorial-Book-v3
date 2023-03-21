.. _term-trap-handle:

.. 实现特权级的切换

Implement the Privilege Level Switching
======================================================

.. toctree::
   :hidden:
   :maxdepth: 5

.. 本节导读

Section Guide
-------------------------------

Since the processor has a hardware-level privilege level mechanism, when the application program is running at the user mode privilege level, it cannot directly access the functions in the batch operating system kernel at the kernel mode privilege level through function calls. However, the application program needs to obtain the services provided by the operating system, so the application program and the operating system need to complete the switching between privilege levels through a certain cooperation mechanism. So that the user mode application program can obtain the service of the kernel mode operating system services. This section will explain how the batch operating system and applications cooperate with each other to complete the privilege level switching under the U/S privilege level provided by the RISC-V 64 processor.

.. 由于处理器具有硬件级的特权级机制，应用程序在用户态特权级运行时，是无法直接通过函数调用访问处于内核态特权级的批处理操作系统内核中的函数。但应用程序又需要得到操作系统提供的服务，所以应用程序与操作系统需要通过某种合作机制完成特权级之间的切换，使得用户态应用程序可以得到内核态操作系统函数的服务。本节将讲解在 RISC-V 64 处理器提供的 U/S 特权级下，批处理操作系统和应用程序如何相互配合，完成特权级切换的。

.. RISC-V特权级切换

RISC-V Privilege Level Switching
---------------------------------------

.. 特权级切换的起因

Why Need Privilege Level Switching
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We know that the batch operating system is designed to run at the kernel-mode privilege level (RISC-V's S mode), which is guaranteed by RustSBI as SEE (Supervisor Execution Environment). The application program is designed to run at the user mode privilege level (RISC-V U mode), and is supervised by the execution environment with the operating system as the core. In this chapter, the execution environment of this application is the AEE (Application Execution Environment) provided by the "Dunkleosteus" batch operating system. In order to establish the execution environment of the application program, the batch operating system needs to perform some initialization work before executing the application program, and monitor the execution of the application program, which is specifically reflected in:

- When starting an application, it is necessary to initialize the user mode context of the application and switch to the user mode to execute the application;
- After the application initiates a system call (i.e. issues a Trap), it needs to be processed in the batch operating system;
- When an error occurs in the execution of an application, it is necessary to go to the batch operating system to kill the application and load and run the next application;
- When the execution of the application program ends, it needs to load and run the next application in the batch operating system (in fact, it is also implemented through the system call ``sys_exit``).

These processes all involve privilege-level switching, so applications, operating systems, and hardware need to work together to complete the privilege-level switching mechanism.

.. 我们知道，批处理操作系统被设计为运行在内核态特权级（RISC-V 的 S 模式），这是作为 SEE（Supervisor Execution Environment）的 RustSBI 所保证的。而应用程序被设计为运行在用户态特权级（RISC-V 的 U 模式），被操作系统为核心的执行环境监管起来。在本章中，这个应用程序的执行环境即是由“邓式鱼” 批处理操作系统提供的 AEE（Application Execution Environment) 。批处理操作系统为了建立好应用程序的执行环境，需要在执行应用程序之前进行一些初始化工作，并监控应用程序的执行，具体体现在：

.. - 当启动应用程序的时候，需要初始化应用程序的用户态上下文，并能切换到用户态执行应用程序； 
.. - 当应用程序发起系统调用（即发出 Trap）之后，需要到批处理操作系统中进行处理；
.. - 当应用程序执行出错的时候，需要到批处理操作系统中杀死该应用并加载运行下一个应用； 
.. - 当应用程序执行结束的时候，需要到批处理操作系统中加载运行下一个应用（实际上也是通过系统调用 ``sys_exit`` 来实现的）。

.. 这些处理都涉及到特权级切换，因此需要应用程序、操作系统和硬件一起协同，完成特权级切换机制。


特权级切换相关的控制状态寄存器

Control Status Registers Related to Privilege Level Switching
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When discussing the trap mechanism of the RISC-V architecture in a general sense, there are usually two points to note:

- Which privilege level the CPU is running at before triggering the Trap;
- Which privilege level the CPU needs to switch to to process the Trap, and return to the original privilege level after the processing is completed.
  
But in this chapter we only consider the following process: when the CPU runs the application program at the user-mode privilege level (RISC-V’s U mode), executes to Trap, switches to the kernel mode privilege level (RISC-V’s S mode), run the corresponding system code responding to the Trap and executes the system call service. After the processing is completed, the application program returns from the kernel mode to the user mode and continues to execute subsequent instructions.

.. 当从一般意义上讨论 RISC-V 架构的 Trap 机制时，通常需要注意两点：

.. - 在触发 Trap 之前 CPU 运行在哪个特权级；
.. - CPU 需要切换到哪个特权级来处理该 Trap ，并在处理完成之后返回原特权级。
  
.. 但本章中我们仅考虑如下流程：当 CPU 在用户态特权级（ RISC-V 的 U 模式）运行应用程序，执行到 Trap，切换到内核态特权级（ RISC-V的S 模式），批处理操作系统的对应代码响应 Trap，并执行系统调用服务，处理完毕后，从内核态返回到用户态应用程序继续执行后续指令。

.. _term-s-mod-csr:

In the RISC-V architecture, there is an important rule about Trap: the privilege level before Trap must not be higher than the privilege level after trap. Therefore, if you switch to the S privilege level after the Trap is triggered (hereinafter referred to as Trap to S), it means that the CPU can only run at the S/U privilege level before the trap occurs. But in any case, as long as it is a trap to S privilege level, the operating system will use the **Control and Status Register** (CSR) related to trap in the S privilege level to assist in trap processing. When we write the trap processing-related code running in the S privileged batch operating system, we need to use the S mode CSR register as shown below.

.. 在 RISC-V 架构中，关于 Trap 有一条重要的规则：在 Trap 前的特权级不会高于 Trap 后的特权级。因此如果触发 Trap 之后切换到 S 特权级（下称 Trap 到 S），说明 Trap 发生之前 CPU 只能运行在 S/U 特权级。但无论如何，只要是 Trap 到 S 特权级，操作系统就会使用 S 特权级中与 Trap 相关的 **控制状态寄存器** (CSR, Control and Status Register) 来辅助 Trap 处理。我们在编写运行在 S 特权级的批处理操作系统中的 Trap 处理相关代码的时候，就需要使用如下所示的 S 模式的 CSR 寄存器。

.. list-table:: Relevant CSR when entering the S privilege Level Trap
    :header-rows: 1
    :align: center
    :widths: 30 100

    * - CSR name
      - The Trap-relevant function of this CSR
    * - sstatus
      - Fields such as ``SPP`` give information such as which privilege level (S/U) the CPU is in before the Trap occurs
    * - sepc
      - When the trap is an exception, record the address of the last instruction executed before the trap occurred
    * - cause
      - Describe the reason for the trap
    * - stval
      - Give trap additional information
    * - stvec
      - Control the entry address of the trap processing code


.. .. list-table:: 进入 S 特权级 Trap 的相关 CSR
..    :header-rows: 1
..    :align: center
..    :widths: 30 100

..    * - CSR 名
..      - 该 CSR 与 Trap 相关的功能
..    * - sstatus
..      - ``SPP`` 等字段给出 Trap 发生之前 CPU 处在哪个特权级（S/U）等信息
..    * - sepc
..      - 当 Trap 是一个异常的时候，记录 Trap 发生之前执行的最后一条指令的地址
..    * - scause
..      - 描述 Trap 的原因
..    * - stval
..      - 给出 Trap 附加信息
..    * - stvec
..      - 控制 Trap 处理代码的入口地址

.. note::

    **The Most Important sstatus Register in S mode**

    Note ``sstatus`` is the most important CSR of the S privilege level, which can control the CPU behavior and execution status of the S privilege level from multiple aspects.

..    **S模式下最重要的 sstatus 寄存器**

..    注意 ``sstatus`` 是 S 特权级最重要的 CSR，可以从多个方面控制 S 特权级的 CPU 行为和执行状态。
   
.. chy   
   我们在这里先给出它在 Trap 处理过程中的作用。

.. 特权级切换

Privilege Level Switching
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When executing a Trap-type instruction (such as ``ecall``), the CPU finds that an exception has been triggered and requires special handling, which involves :ref:`Execution context switch <term-ee-switch>`. Specifically, the application program in the user mode execution environment requests a certain service function from the operating system in the kernel mode execution environment through the ``ecall`` instruction. Then the processor and the operating system willswitch to the kernel mode execution environment. And after the operating system completes the service, switch back to the user mode execution environment again. And then the application program will continue to execute immediately after the ``ecall`` instruction. Refer to :ref:`Diagram <environment-call-flow>`.

.. 当执行一条 Trap 类指令（如 ``ecall`` 时），CPU 发现触发了一个异常并需要进行特殊处理，这涉及到 :ref:`执行环境切换 <term-ee-switch>` 。具体而言，用户态执行环境中的应用程序通过 ``ecall`` 指令向内核态执行环境中的操作系统请求某项服务功能，那么处理器和操作系统会完成到内核态执行环境的切换，并在操作系统完成服务后，再次切换回用户态执行环境，然后应用程序会紧接着 ``ecall`` 指令的后一条指令位置处继续执行，参考 :ref:`图示 <environment-call-flow>` 。

.. chy ？？？: 这条触发 Trap 的指令和进入 Trap 之前执行的最后一条指令不一定是同一条。

.. chy？？？ 下面的内容合并到第零章的os 抽象一节中， 执行环境切换, context等
    _term-execution-of-thread:
    回顾第一章的 :ref:`函数调用与栈 <function-call-and-stack>` ，我们知道在一个固定的 CPU 上，只要有一个栈作为存储空间，我们就能以多种
    普通控制流（顺序、分支、循环结构和多层嵌套函数调用）组合的方式，来一行一行的执行源代码（以编程语言级的视角），也是一条一条的执行汇编指令
    （以汇编语言级的视角）。只考虑普通控制流，那么从某条指令开始记录，该 CPU 可用的所有资源，包括自带的所有通用寄存器（包括虚拟的描述当前执行
    指令地址的寄存器 pc ）和当前特权级可用的 CSR 以及位于内存中的一块栈空间，它们会随着指令的执行而逐渐发生变化。这种局限在普通控制流（相对于 :ref:`异常控制流 <term-ecf>` 而言）之内的
    连续指令执行和与之同步的对相关资源的改变我们用一个新名词 **执行流** (Execution Flow) 来命名。执行流的状态是一个由它衍生出来的
    概念，表示截止到某条指令执行完毕所有相关资源（包括寄存器、栈）的状态集合，它完整描述了自记录起始之后该执行流的指令执行历史。
    .. note::
    实际上 CPU 还有其他资源可用：
    - 内存除了与执行流绑定的栈之外的其他存储空间，比如程序中的数据段；
    - 外围 I/O 设备。
    它们也会在执行期间动态发生变化。但它们可能由多条执行流共享，难以清晰的从中单独区分出某一条执行流的状态变化。因此在执行流概念中，
    我们不将其纳入考虑。

.. chy？？？ 内容与第一段有重复
    让我们通过从 U 特权级 Trap 到 S 特权级的切换过程（这是一个特例，实际上在 Trap 的前后，特权级也可以不变）来分析一下在 Trap 前后发生了哪些事情。首先假设CPU 正处在 U 特权级跑着应用程序的代码。在执行完某一条指令之后， CPU 发现一个
    中断/异常被触发，于是必须将应用执行流暂停，先 Trap 到更高的 S 特权级去执行批处理操作系统提供的相应服务代码，等待执行
    完了之后再回过头来恢复到并继续执行应用程序的执行流。
.. chy？？？  内容与第一段有重复
    我们可以将 CPU 在 S 特权级执行的那一段指令序列也看成一个控制流，因为它全程只是以普通控制流的模式在 S 特权级执行。这个控制流的意义就在于
    处理 Trap ，我们可以将其称之为 **Trap 专用** 控制流，它在 Trap 触发的时候开始，并于 Trap 处理完毕之后结束。于是我们可以从执行环境切换和控制流的角度来看待 
    操作系统对Trap 的整个处理过程： CPU 从用户态执行环境中的应用程序的普通控制流转到内核态执行环境中的 Trap 执行流，然后再切换回去继续运行应用程序。站在应用程序的角度， 由操作系统和CPU协同完成的Trap 机制对它是完全透明的，无论应用程序在它的哪一条指令执行结束后进入 Trap ，它总是相信在 Trap 结束之后 CPU 能够在与被打断的时候"相同"的执行环境中继续正确的运行应用程序的指令。
.. chy？？？ 
    .. note::
.. chy？？？  内容与第一段有重复
        这里所说的相同并不是绝对相同，但是其变化是完全能够被应用程序预知到的。比如应用程序通过 ``ecall`` 指令请求底层高特权级软件的功能，
        由调用规范它知道 Trap 之后 ``a0~a1`` 两个寄存器会被用来保存返回值，所以会发生变化。这个信息是应用程序明确知晓的，但某种程度上
        确实也体现了执行流的变化。

After the application is switched back, it needs to restore the application context from the execution location where the system call request was issued and continue to execute. It requires maintaining the application context before and after the switch. The context of the application program includes two main parts, the general-purpose register and the stack. Since the CPU shares a set of general-purpose registers under different privilege levels, the operating system will also use these registers during the trap processing. This will change the context of the application. Therefore, just like function calls need to save the function call context / activity record, before executing the operating system's trap processing (which will modify the general-purpose registers), we need to save these registers somewhere (a memory block or the kernel's stack) and save them. These registers are restored after the trap processing ends.

.. 应用程序被切换回来之后需要从发出系统调用请求的执行位置恢复应用程序上下文并继续执行，这需要在切换前后维持应用程序的上下文保持不变。应用程序的上下文包括通用寄存器和栈两个主要部分。由于 CPU 在不同特权级下共享一套通用寄存器，所以在运行操作系统的 Trap 处理过程中，操作系统也会用到这些寄存器，这会改变应用程序的上下文。因此，与函数调用需要保存函数调用上下文/活动记录一样，在执行操作系统的 Trap 处理过程（会修改通用寄存器）之前，我们需要在某个地方（某内存块或内核的栈）保存这些寄存器并在 Trap 处理结束后恢复这些寄存器。

In addition to general-purpose registers, there are some CSRs that may be modified during trap processing, such as the privilege level of the CPU. We want to ensure that their changes are within our expectations. For example, for the privilege level switching, it should be at the U privilege level before the Trap, and at the S privilege level when processing the trap, and then return to the U privilege level after returning. As for the stack problem, it is relatively simple. As long as the memory areas corresponding to the stacks used to record the execution history during the execution of the two applications do not intersect, there will be no overwriting or data corruption problems that may bother us, and there is no need to perform save/restore.

.. 除了通用寄存器之外还有一些可能在处理 Trap 过程中会被修改的 CSR，比如 CPU 所在的特权级。我们要保证它们的变化在我们的预期之内。比如，对于特权级转换而言，应该是 Trap 之前在 U 特权级，处理 Trap 的时候在 S 特权级，返回之后又需要回到 U 特权级。而对于栈问题则相对简单，只要两个应用程序执行过程中用来记录执行历史的栈所对应的内存区域不相交，就不会产生令我们头痛的覆盖问题或数据破坏问题，也就无需进行保存/恢复。

Part of the privilege level switching is directly completed by the hardware, and the other part needs to be implemented by the operating system.

.. 特权级切换的具体过程一部分由硬件直接完成，另一部分则需要由操作系统来实现。

.. _trap-hw-mechanism:

.. 特权级切换的硬件控制机制

Hardware Control Mechanism for Privilege Level Switching
--------------------------------------------------------------------------

When the CPU finishes executing an instruction (such as ``ecall``) and prepares to trap (``Trap``) from the user privilege level to the S privilege level, the hardware will automatically complete the following steps:

- The ``SPP`` field of ``sstatus`` will be modified to the current privilege level (U/S) of the CPU.
- ``sepc`` will be modified to the address of the next instruction that will be executed by default after the trap processing is completed.
- ``scause/stval`` will be modified to the reason of this trap and related additional information respectively.
- The CPU will jump to the trap processing entry address set by ``stvec``, set the current privilege level to S, and then start executing from the trap processing entry address.

.. 当 CPU 执行完一条指令（如 ``ecall`` ）并准备从用户特权级 陷入（ ``Trap`` ）到 S 特权级的时候，硬件会自动完成如下这些事情：

.. - ``sstatus`` 的 ``SPP`` 字段会被修改为 CPU 当前的特权级（U/S）。
.. - ``sepc`` 会被修改为 Trap 处理完成后默认会执行的下一条指令的地址。
.. - ``scause/stval`` 分别会被修改成这次 Trap 的原因以及相关的附加信息。
.. - CPU 会跳转到 ``stvec`` 所设置的 Trap 处理入口地址，并将当前特权级设置为 S ，然后从Trap 处理入口地址处开始执行。

.. note::

    **stvec related details**

    In RV64, ``stvec`` is a 64-bit CSR, which stores the entry address of the interrupt processing when the interrupt is enabled. It has two fields:

    - MODE is at [1:0], the length is 2 bits;
    - BASE is located at [63:2] and has a length of 62 bits.

    When the MODE field is 0, ``stvec`` is set to Direct mode. At this time, no matter what the reason is for the trap entering S mode, the entry address for processing the trap is ``BASE<<2``, and the CPU will jump to this place for exception handling. In this book we will only set ``stvec`` to Direct mode. And ``stvec`` can also be set to Vectored mode. Interested students can refer to the RISC-V instruction set privilege level specification by themselves.

..    **stvec 相关细节**

..    在 RV64 中， ``stvec`` 是一个 64 位的 CSR，在中断使能的情况下，保存了中断处理的入口地址。它有两个字段：

..    - MODE 位于 [1:0]，长度为 2 bits；
..    - BASE 位于 [63:2]，长度为 62 bits。

..    当 MODE 字段为 0 的时候， ``stvec`` 被设置为 Direct 模式，此时进入 S 模式的 Trap 无论原因如何，处理 Trap 的入口地址都是 ``BASE<<2`` ， CPU 会跳转到这个地方进行异常处理。本书中我们只会将 ``stvec`` 设置为 Direct 模式。而 ``stvec`` 还可以被设置为 Vectored 模式，有兴趣的同学可以自行参考 RISC-V 指令集特权级规范。

When the CPU completes the Trap processing and prepares to return, it needs to complete it through an S-privileged privileged instruction ``sret``. This instruction specifically performs the following step:

- The CPU will set the current privilege level to U or S according to the ``SPP`` field of ``sstatus``;
- The CPU will jump to the instruction pointed to by the ``sepc`` register and continue execution.

These are basically what the hardware has to do. And some remaining finishing work can be handed over to the software, so that the operating system can have greater flexibility.

.. 而当 CPU 完成 Trap 处理准备返回的时候，需要通过一条 S 特权级的特权指令 ``sret`` 来完成，这一条指令具体完成以下功能：

.. - CPU 会将当前的特权级按照 ``sstatus`` 的 ``SPP`` 字段设置为 U 或者 S ；
.. - CPU 会跳转到 ``sepc`` 寄存器指向的那条指令，然后继续执行。

.. 这些基本上都是硬件不得不完成的事情，还有一些剩下的收尾工作可以都交给软件，让操作系统能有更大的灵活性。

.. 用户栈与内核栈

User Stack and Kernel Stack
--------------------------------

The moment the Trap is triggered, the CPU will switch to the S privilege level and jump to the location indicated by ``stvec``. But before officially entering the trap processing at the S privilege level, it was mentioned above that we must save the register state of the original control flow, which is generally saved through the kernel stack. Note that we need to use the kernel stack specially prepared for the operating system, not the user stack used by the application when it is running.

.. 在 Trap 触发的一瞬间， CPU 就会切换到 S 特权级并跳转到 ``stvec`` 所指示的位置。但是在正式进入 S 特权级的 Trap 处理之前，上面提到过我们必须保存原控制流的寄存器状态，这一般通过内核栈来保存。注意，我们需要用专门为操作系统准备的内核栈，而不是应用程序运行时用到的用户栈。

.. 
    chy:我们在一个作为用户栈的特别留出的内存区域上保存应用程序的栈信息，而 Trap 执行流则使用另一个内核栈。

Using two different stacks is mainly for security: if two control flows (that is, the control flow of the application and the control flow of the kernel) use the same stack, the application can read the history information of the trap control flow after returning, such as the address of some kernel functions. This will bring security risks. So, what we have to do is to add a piece of assembly code in the batch operating system to switch from the user stack to the kernel stack, and save the register state of the application control flow on the kernel stack.

.. 使用两个不同的栈主要是为了安全性：如果两个控制流（即应用程序的控制流和内核的控制流）使用同一个栈，在返回之后应用程序就能读到 Trap 控制流的历史信息，比如内核一些函数的地址，这样会带来安全隐患。于是，我们要做的是，在批处理操作系统中添加一段汇编代码，实现从用户栈切换到内核栈，并在内核栈上保存应用程序控制流的寄存器状态。

We declare two types ``KernelStack`` and ``UserStack`` to represent the user stack and the kernel stack respectively, both of which are simply wrappers around byte arrays:

.. 我们声明两个类型 ``KernelStack`` 和 ``UserStack`` 分别表示用户栈和内核栈，它们都只是字节数组的简单包装：

.. code-block:: rust
    :linenos:

    // os/src/batch.rs

    const USER_STACK_SIZE: usize = 4096 * 2;
    const KERNEL_STACK_SIZE: usize = 4096 * 2;

    #[repr(align(4096))]
    struct KernelStack {
        data: [u8; KERNEL_STACK_SIZE],
    }

    #[repr(align(4096))]
    struct UserStack {
        data: [u8; USER_STACK_SIZE],
    }

    static KERNEL_STACK: KernelStack = KernelStack { data: [0; KERNEL_STACK_SIZE] };
    static USER_STACK: UserStack = UserStack { data: [0; USER_STACK_SIZE] };

The constants ``USER_STACK_SIZE`` and ``KERNEL_STACK_SIZE`` specify the size of the kernel stack and user stack to be :math:`8\text{KiB}` respectively. Both types are instantiated as global variables in the ``.bss`` section of the batch operating system.

We implemented the ``get_sp`` method for both types to get the address of the top of the stack. Since the stack grows downward in RISC-V, we only need to return the end address of the wrapped array, taking the user stack type ``UserStack`` as an example:

.. 常数 ``USER_STACK_SIZE`` 和 ``KERNEL_STACK_SIZE`` 指出内核栈和用户栈的大小分别为 :math:`8\text{KiB}` 。两个类型是以全局变量的形式实例化在批处理操作系统的 ``.bss`` 段中的。

.. 我们为两个类型实现了 ``get_sp`` 方法来获取栈顶地址。由于在 RISC-V 中栈是向下增长的，我们只需返回包裹的数组的结尾地址，以用户栈类型 ``UserStack`` 为例：

.. code-block:: rust
    :linenos:

    impl UserStack {
        fn get_sp(&self) -> usize {
            self.data.as_ptr() as usize + USER_STACK_SIZE
        }
    }

So switching the stack is very simple-- just change the value of the ``sp`` register to the return value of ``get_sp``.

.. 于是换栈是非常简单的，只需将 ``sp`` 寄存器的值修改为 ``get_sp`` 的返回值即可。

.. _term-trap-context:

Next is the Trap context (that is, the data structure ``TrapContext``), similar to the function call context mentioned above, that is, the physical resource state that needs to be saved when the Trap occurs. We put them together in a named ``TrapContext`` is defined as follows:

.. 接下来是Trap上下文（即数据结构 ``TrapContext`` ），类似前面提到的函数调用上下文，即在 Trap 发生时需要保存的物理资源内容，并将其一起放在一个名为 ``TrapContext`` 的类型中，定义如下：

.. code-block:: rust
    :linenos:

    // os/src/trap/context.rs

    #[repr(C)]
    pub struct TrapContext {
        pub x: [usize; 32],
        pub sstatus: Sstatus,
        pub sepc: usize,
    }

You can see that it contains all general-purpose registers ``x0~x31``, as well as ``sstatus`` and ``sepc``. So why do you need to save them?

- For general-purpose registers, the two control flows (application control flow and kernel control flow) run at different privilege levels, and the software they belong to may also be written in different programming languages. Although only Trap will be executed in the Trap control flow, there are still many modules that may be called directly or indirectly. So it is difficult or impossible to find out which registers do not need to be saved. In this case, we can only save them all. But there are some exceptions here, such as ``x0`` is hard-coded to 0, it will not change naturally; there is also the ``tp(x4)`` register, unless we manually use it for some special purposes, it is generally will not be used either. Although they don't need to be saved, we still reserve space for them in ``TrapContext``, mainly for the convenience of subsequent implementations.
- For CSR, we know that when entering the Trap, the hardware will immediately overwrite all or part of ``scause/stval/sstatus/sepc``. The case of ``scause/stval`` is: it is always used as soon as the Trap is processed or saved elsewhere, so there is no risk of it being modified and having bad effects. For ``sstatus/sepc``, they will be meaningful throughout the trap processing (they are also used when ``sret`` is at the end of the trap control flow). And it is possible that trap nesting have their values are overwritten. So we need to save them as well and restore them before ``sret``

.. 可以看到里面包含所有的通用寄存器 ``x0~x31`` ，还有 ``sstatus`` 和 ``sepc`` 。那么为什么需要保存它们呢？

.. - 对于通用寄存器而言，两条控制流（应用程序控制流和内核控制流）运行在不同的特权级，所属的软件也可能由不同的编程语言编写，虽然在 Trap 控制流中只是会执行 Trap 处理相关的代码，但依然可能直接或间接调用很多模块，因此很难甚至不可能找出哪些寄存器无需保存。既然如此我们就只能全部保存了。但这里也有一些例外，如 ``x0`` 被硬编码为 0 ，它自然不会有变化；还有 ``tp(x4)`` 寄存器，除非我们手动出于一些特殊用途使用它，否则一般也不会被用到。虽然它们无需保存，但我们仍然在 ``TrapContext`` 中为它们预留空间，主要是为了后续的实现方便。
.. - 对于 CSR 而言，我们知道进入 Trap 的时候，硬件会立即覆盖掉 ``scause/stval/sstatus/sepc`` 的全部或是其中一部分。``scause/stval`` 的情况是：它总是在 Trap 处理的第一时间就被使用或者是在其他地方保存下来了，因此它没有被修改并造成不良影响的风险。而对于 ``sstatus/sepc`` 而言，它们会在 Trap 处理的全程有意义（在 Trap 控制流最后 ``sret`` 的时候还用到了它们），而且确实会出现 Trap 嵌套的情况使得它们的值被覆盖掉。所以我们需要将它们也一起保存下来，并在 ``sret`` 之前恢复原样。


.. Trap 管理

Trap Management
-------------------------------

The core of privilege level switching is the management of traps. This mainly involves the following:

- When the application enters the kernel state through ``ecall``, the operating system saves the Trap context of the interrupted application;
- The operating system completes the dispatching and processing of the system call service according to the contents of the Trap-related CSR register;
- After the operating system finishes the system call service, it needs to restore the trap context of the interrupted application, and let the application continue to execute through ``sret``.

Next, we introduce the above in detail.

.. 特权级切换的核心是对Trap的管理。这主要涉及到如下一些内容：

.. - 应用程序通过 ``ecall`` 进入到内核状态时，操作系统保存被打断的应用程序的 Trap 上下文；
.. - 操作系统根据Trap相关的CSR寄存器内容，完成系统调用服务的分发与处理；
.. - 操作系统完成系统调用服务后，需要恢复被打断的应用程序的Trap 上下文，并通 ``sret`` 让应用程序继续执行。

.. 接下来我们具体介绍上述内容。

.. Trap 上下文的保存与恢复

Saving and Restoring Trap Context
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The first is the assembly code that implements the saving and restoration of the Trap context.

.. 首先是具体实现 Trap 上下文保存和恢复的汇编代码。

.. _trap-context-save-restore: 


When the batch operating system is initialized, we need to modify the ``stvec`` register to point to the correct trap processing entry point.

.. 在批处理操作系统初始化的时候，我们需要修改 ``stvec`` 寄存器来指向正确的 Trap 处理入口点。

.. code-block:: rust
    :linenos:

    // os/src/trap/mod.rs

    global_asm!(include_str!("trap.S"));

    pub fn init() {
        extern "C" { fn __alltraps(); }
        unsafe {
            stvec::write(__alltraps as usize, TrapMode::Direct);
        }
    }

Here we introduce an external symbol ``__alltraps`` and set ``stvec`` to point to its address in Direct mode. We implement the assembly code of Trap context save/restore in ``os/src/trap/trap.S``, marked as functions with external symbols ``__alltraps`` and ``__restore`` respectively. And we pass ``global_asm The !`` macro to insert the assembly code ``trap.S``.

The overall process of trap processing is as follows: first save the trap context on the kernel stack through ``__alltraps``, and then jump to the ``trap_handler`` function written in Rust to complete trap dispatching and processing. After ``trap_handler`` returns, use ``__restore`` to restore the registers from the trap context saved on the kernel stack. Finally, return to the application program execution through a ``sret`` instruction.

The first is the implementation of ``__alltraps`` that holds the Trap context:

.. 这里我们引入了一个外部符号 ``__alltraps`` ，并将 ``stvec`` 设置为 Direct 模式指向它的地址。我们在 ``os/src/trap/trap.S`` 中实现 Trap 上下文保存/恢复的汇编代码，分别用外部符号 ``__alltraps`` 和 ``__restore`` 标记为函数，并通过 ``global_asm!`` 宏将 ``trap.S`` 这段汇编代码插入进来。

.. Trap 处理的总体流程如下：首先通过 ``__alltraps`` 将 Trap 上下文保存在内核栈上，然后跳转到使用 Rust 编写的 ``trap_handler`` 函数完成 Trap 分发及处理。当 ``trap_handler`` 返回之后，使用 ``__restore`` 从保存在内核栈上的 Trap 上下文恢复寄存器。最后通过一条 ``sret`` 指令回到应用程序执行。

.. 首先是保存 Trap 上下文的 ``__alltraps`` 的实现：

.. code-block:: riscv
    :linenos:

    # os/src/trap/trap.S

    .macro SAVE_GP n
        sd x\n, \n*8(sp)
    .endm

    .align 2
    __alltraps:
        csrrw sp, sscratch, sp
        # now sp->kernel stack, sscratch->user stack
        # allocate a TrapContext on kernel stack
        addi sp, sp, -34*8
        # save general-purpose registers
        sd x1, 1*8(sp)
        # skip sp(x2), we will save it later
        sd x3, 3*8(sp)
        # skip tp(x4), application does not use it
        # save x5~x31
        .set n, 5
        .rept 27
            SAVE_GP %n
            .set n, n+1
        .endr
        # we can use t0/t1/t2 freely, because they were saved on kernel stack
        csrr t0, sstatus
        csrr t1, sepc
        sd t0, 32*8(sp)
        sd t1, 33*8(sp)
        # read user stack from sscratch and save it on the kernel stack
        csrr t2, sscratch
        sd t2, 2*8(sp)
        # set input argument of trap_handler(cx: &mut TrapContext)
        mv a0, sp
        call trap_handler

- On line 7, we use ``.align`` to align the address of ``__alltraps`` to 4 bytes, which is the requirement of the RISC-V privilege level specification;
- The prototype of ``csrrw`` in line 9 is :math:`\text{csrrw rd, csr, rs}`, which can load the current CSR value into the general register :math:`\text{rd}`, and then write the general register value :math:`\text{rs}` to this CSR. Effectively, it exchanges sscratch and sp. Before this line sp points to the user stack, sscratch points to the kernel stack (which will be explained later). Now sp points to the kernel stack, and sscratch points to the user stack.
- On line 12, we are going to save the Trap context on the kernel stack, so we pre-allocate a stack frame of :math:`34\times 8` bytes. Here the sp is changed, indicating that it is indeed on the kernel stack.
- Lines 13~24, save the general-purpose registers x0~x31 of the Trap context. We skip x0 and tp(x4), as explained before. We also don't save sp(x2) here, because we need to hinge on it to find the correct location where each register is saved. In fact, after the stack frame is allocated, the address range we can use to save the Trap context is :math:`[\text{sp},\text{sp}+8\times34)`. According to the ``TrapContext`` structure, the memory layout of the kernel stack is: from the kernel stack top (the address pointed by sp), the general-purpose registers x0~x31 are placed in order from the low address to the high address, and finally sstatus and sepc. Therefore the general register xn should be stored in the address range :math:`[\text{sp}+8n,\text{sp}+8(n+1))`. To simplify the code, the 27 general-purpose registers x5~x31 are saved by using the ``SAVE_GP`` macro each time through a loop-like ``.rept``, and the essence is the same. Note that we need to add ``.altmacro`` at the beginning of ``trap.S`` to enable the ``.rept`` command. 

- Lines 25~28, we read the values of CSR sstatus and sepc into registers t0 and t1 respectively and save them to the corresponding positions of the kernel stack. The function of the instruction :math:`\text{csrr rd, csr}` is to read the value of CSR into the register :math:`\text{rd}`. Here we don't need to worry about t0 and t1 being overwritten, because they have just been saved.
- Lines 30~31 deal exclusively with sp. First read the value of sscratch into register t2 and save it on the kernel stack. Note: The value of sscratch is the value of sp before entering Trap, pointing to the user stack. And now sp points to the kernel stack.
- The 33rd line command :math:`\text{a}_0\leftarrow\text{sp}`, let the register a0 point to the stack pointer of the kernel stack, which is the address of the Trap context we just saved. Because we will To call ``trap_handler`` for Trap processing, its first parameter ``cx`` must be obtained from a0 by the calling convention. The reason why the trap handler ``trap_handler`` needs a trap context is that it needs to know the values of some of the registers, such as the syscall ID and corresponding parameters passed by the application when the system call is triggered. We cannot directly use the current values of these registers, because they may have been modified. So we have to find the saved values on the kernel stack.

.. - 第 7 行我们使用 ``.align`` 将 ``__alltraps`` 的地址 4 字节对齐，这是 RISC-V 特权级规范的要求；
.. - 第 9 行的 ``csrrw`` 原型是 :math:`\text{csrrw rd, csr, rs}` 可以将 CSR 当前的值读到通用寄存器 :math:`\text{rd}` 中，然后将通用寄存器 :math:`\text{rs}` 的值写入该 CSR 。因此这里起到的是交换 sscratch 和 sp 的效果。在这一行之前 sp 指向用户栈， sscratch 指向内核栈（原因稍后说明），现在 sp 指向内核栈， sscratch 指向用户栈。
.. - 第 12 行，我们准备在内核栈上保存 Trap 上下文，于是预先分配 :math:`34\times 8` 字节的栈帧，这里改动的是 sp ，说明确实是在内核栈上。
.. - 第 13~24 行，保存 Trap 上下文的通用寄存器 x0~x31，跳过 x0 和 tp(x4)，原因之前已经说明。我们在这里也不保存 sp(x2)，因为我们要基于它来找到每个寄存器应该被保存到的正确的位置。实际上，在栈帧分配之后，我们可用于保存 Trap 上下文的地址区间为 :math:`[\text{sp},\text{sp}+8\times34)` ，按照  ``TrapContext`` 结构体的内存布局，基于内核栈的位置（sp所指地址）来从低地址到高地址分别按顺序放置 x0~x31这些通用寄存器，最后是 sstatus 和 sepc 。因此通用寄存器 xn 应该被保存在地址区间 :math:`[\text{sp}+8n,\text{sp}+8(n+1))` 。

..   为了简化代码，x5~x31 这 27 个通用寄存器我们通过类似循环的 ``.rept`` 每次使用 ``SAVE_GP`` 宏来保存，其实质是相同的。注意我们需要在 ``trap.S`` 开头加上 ``.altmacro`` 才能正常使用 ``.rept`` 命令。
.. - 第 25~28 行，我们将 CSR sstatus 和 sepc 的值分别读到寄存器 t0 和 t1 中然后保存到内核栈对应的位置上。指令 :math:`\text{csrr rd, csr}`  的功能就是将 CSR 的值读到寄存器 :math:`\text{rd}` 中。这里我们不用担心 t0 和 t1 被覆盖，因为它们刚刚已经被保存了。
.. - 第 30~31 行专门处理 sp 的问题。首先将 sscratch 的值读到寄存器 t2 并保存到内核栈上，注意： sscratch 的值是进入 Trap 之前的 sp 的值，指向用户栈。而现在的 sp 则指向内核栈。
.. - 第 33 行令 :math:`\text{a}_0\leftarrow\text{sp}`，让寄存器 a0 指向内核栈的栈指针也就是我们刚刚保存的 Trap 上下文的地址，这是由于我们接下来要调用 ``trap_handler`` 进行 Trap 处理，它的第一个参数 ``cx`` 由调用规范要从 a0 中获取。而 Trap 处理函数 ``trap_handler`` 需要 Trap 上下文的原因在于：它需要知道其中某些寄存器的值，比如在系统调用的时候应用程序传过来的 syscall ID 和对应参数。我们不能直接使用这些寄存器现在的值，因为它们可能已经被修改了，因此要去内核栈上找已经被保存下来的值。


.. _term-atomic-instruction:

.. note::


    **CSR related atomic instructions**

     The instruction to read and write CSR in RISC-V is a kind of instruction that can complete multiple read and write operations without interruption. This instruction that completes multiple operations without being interrupted is called **Atomic Instruction**. The meaning of **atom** here is "the smallest indivisible individual", that is to say, the multiple operations of the instruction are either not completed or all completed without being in some intermediate state.

     In addition, conventional data processing and memory access instructions in the RISC-V architecture can only operate general-purpose registers and cannot operate CSR. Therefore, when you want to operate the CSR, you need to read the CSR into a general-purpose register with the instruction of reading CSR first, then operate the general-purpose register, and finally write the value of the general-purpose register with the instruction of writing CSR into the CSR.

    .. **CSR 相关原子指令**

    .. RISC-V 中读写 CSR 的指令是一类能不会被打断地完成多个读写操作的指令。这种不会被打断地完成多个操作的指令被称为 **原子指令** (Atomic Instruction)。这里的 **原子** 的含义是“不可分割的最小个体”，也就是说指令的多个操作要么都不完成，要么全部完成，而不会处于某种中间状态。

    .. 另外，RISC-V 架构中常规的数据处理和访存类指令只能操作通用寄存器而不能操作 CSR 。因此，当想要对 CSR 进行操作时，需要先使用读取 CSR 的指令将 CSR 读到一个通用寄存器中，而后操作该通用寄存器，最后再使用写入 CSR 的指令将该通用寄存器的值写入到 CSR 中。

When ``trap_handler`` returns, it will be executed from the next instruction that called ``trap_handler``, that is, ``__restore`` to restore from the Trap context on the stack:

.. 当 ``trap_handler`` 返回之后会从调用 ``trap_handler`` 的下一条指令开始执行，也就是从栈上的 Trap 上下文恢复的 ``__restore`` ：

.. _code-restore:

.. code-block:: riscv
    :linenos:

    # os/src/trap/trap.S

    .macro LOAD_GP n
        ld x\n, \n*8(sp)
    .endm

    __restore:
        # case1: start running app by __restore
        # case2: back to U after handling trap
        mv sp, a0
        # now sp->kernel stack(after allocated), sscratch->user stack
        # restore sstatus/sepc
        ld t0, 32*8(sp)
        ld t1, 33*8(sp)
        ld t2, 2*8(sp)
        csrw sstatus, t0
        csrw sepc, t1
        csrw sscratch, t2
        # restore general-purpuse registers except sp/tp
        ld x1, 1*8(sp)
        ld x3, 3*8(sp)
        .set n, 5
        .rept 27
            LOAD_GP %n
            .set n, n+1
        .endr
        # release TrapContext on kernel stack
        addi sp, sp, 34*8
        # now sp->kernel stack, sscratch->user stack
        csrrw sp, sscratch, sp
        sret

- The 10th line is weird, let's ignore it for now, assuming it never happened. Then sp still points to the top of the kernel stack.
- Lines 13~26 are responsible for restoring the general-purpose registers and CSR from the Trap context at the top of the kernel stack. Note that we need to restore the CSR first and then restore the general registers, so that the three temporary registers we use can be restored correctly.
- Before line 28, sp points to the top of the kernel stack after saving the Trap context, and sscratch points to the top of the user stack. We reclaim the memory occupied by the Trap context on the kernel stack on line 28, and return to the top of the kernel stack before entering the Trap. In line 30, sscratch and sp are exchanged again. Now sp points to the top of the user stack again, and sscratch still saves the state before entering Trap and points to the top of the kernel stack.
- After the application control flow state is restored, on line 31 we use the ``sret`` instruction to return to the U privilege level to carry on the application control flow.

.. - 第 10 行比较奇怪我们暂且不管，假设它从未发生，那么 sp 仍然指向内核栈的栈顶。
.. - 第 13~26 行负责从内核栈顶的 Trap 上下文恢复通用寄存器和 CSR 。注意我们要先恢复 CSR 再恢复通用寄存器，这样我们使用的三个临时寄存器才能被正确恢复。
.. - 在第 28 行之前，sp 指向保存了 Trap 上下文之后的内核栈栈顶， sscratch 指向用户栈栈顶。我们在第 28 行在内核栈上回收 Trap 上下文所占用的内存，回归进入 Trap 之前的内核栈栈顶。第 30 行，再次交换 sscratch 和 sp，现在 sp 重新指向用户栈栈顶，sscratch 也依然保存进入 Trap 之前的状态并指向内核栈栈顶。
.. - 在应用程序控制流状态被还原之后，第 31 行我们使用 ``sret`` 指令回到 U 特权级继续运行应用程序控制流。

.. note::

    **Use of sscratch CSR**

     When the privilege level is switched, we need to save the Trap context on the kernel stack, so we need a register to temporarily store the kernel stack address, and use it as the base address pointer to sequentially save the contents of the Trap context. But all general-purpose registers cannot be used as base address pointers, because they all need to be saved. And suppose they were overwritten, it would affect the execution of subsequent application control flow.

     In fact, we are missing an important transit register, and the ``sscratch`` CSR is born for this. It can be seen from the above assembly code that when saving the Trap context, it plays two roles: firstly, it saves the address of the kernel stack, and secondly, it can be used as a transfer station for ``sp`` (currently pointed to the user stack) to temporarily save the value in ``sscratch``. In this way, only one ``csrrw sp, sscratch, sp`` instruction (exchange the contents of ``sp`` and ``sscratch`` two registers) is needed to achieve the switch from the user stack to the kernel stack, an extremely elegant implementation.

    .. **sscratch CSR 的用途**

    .. 在特权级切换的时候，我们需要将 Trap 上下文保存在内核栈上，因此需要一个寄存器暂存内核栈地址，并以它作为基地址指针来依次保存 Trap 上下文的内容。但是所有的通用寄存器都不能够用作基地址指针，因为它们都需要被保存，如果覆盖掉它们，就会影响后续应用控制流的执行。

    .. 事实上我们缺少了一个重要的中转寄存器，而 ``sscratch`` CSR 正是为此而生。从上面的汇编代码中可以看出，在保存 Trap 上下文的时候，它起到了两个作用：首先是保存了内核栈的地址，其次它可作为一个中转站让 ``sp`` （目前指向的用户栈的地址）的值可以暂时保存在 ``sscratch`` 。这样仅需一条 ``csrrw  sp, sscratch, sp`` 指令（交换对 ``sp`` 和 ``sscratch`` 两个寄存器内容）就完成了从用户栈到内核栈的切换，这是一种极其精巧的实现。

.. Trap 分发与处理

Trap Dispatching and Processing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Traps are dispatched and processed in the ``trap_handler`` function implemented in Rust:

.. Trap 在使用 Rust 实现的 ``trap_handler`` 函数中完成分发和处理：

.. code-block:: rust
    :linenos:

    // os/src/trap/mod.rs

    #[no_mangle]
    pub fn trap_handler(cx: &mut TrapContext) -> &mut TrapContext {
        let scause = scause::read();
        let stval = stval::read();
        match scause.cause() {
            Trap::Exception(Exception::UserEnvCall) => {
                cx.sepc += 4;
                cx.x[10] = syscall(cx.x[17], [cx.x[10], cx.x[11], cx.x[12]]) as usize;
            }
            Trap::Exception(Exception::StoreFault) |
            Trap::Exception(Exception::StorePageFault) => {
                println!("[kernel] PageFault in application, kernel killed it.");
                run_next_app();
            }
            Trap::Exception(Exception::IllegalInstruction) => {
                println!("[kernel] IllegalInstruction in application, kernel killed it.");
                run_next_app();
            }
            _ => {
                panic!("Unsupported trap {:?}, stval = {:#x}!", scause.cause(), stval);
            }
        }
        cx
    }

- Line 4 declares that the return value is ``&mut TrapContext`` and actually returns the incoming Trap context ``cx`` in line 25. Hence the ``a0`` register in ``__restore`` remain unchanged before and after calling ``trap_handler`` --  it still points to the top of the kernel stack after the Trap context is allocated, which is the same as the value of ``sp`` at this time, here :math:`\text{sp}\leftarrow\ text{a}_0` will not be a problem;
- Line 7 performs the dispatching according to the Trap reason saved in the ``scause`` register. Here we don't need to manually operate these CSRs, but use Rust's riscv library for more convenience. To import the riscv library, we need:

  .. code-block:: toml

      # os/Cargo.toml
      
      [dependencies]
      riscv = { git = "https://github.com/rcore-os/riscv", features = ["inline-asm"] }  

.. - 第 4 行声明返回值为 ``&mut TrapContext`` 并在第 25 行实际将传入的Trap 上下文 ``cx`` 原样返回，因此在 ``__restore`` 的时候 ``a0`` 寄存器在调用 ``trap_handler`` 前后并没有发生变化，仍然指向分配 Trap 上下文之后的内核栈栈顶，和此时 ``sp`` 的值相同，这里的 :math:`\text{sp}\leftarrow\text{a}_0` 并不会有问题；
.. - 第 7 行根据 ``scause`` 寄存器所保存的 Trap 的原因进行分发处理。这里我们无需手动操作这些 CSR ，而是使用 Rust 的 riscv 库来更加方便的做这些事情。要引入 riscv 库，我们需要：

  .. code-block:: toml

      # os/Cargo.toml
      
      [dependencies]
      riscv = { git = "https://github.com/rcore-os/riscv", features = ["inline-asm"] }  
    
- In lines 8~11, it is found that the trap cause is the Environment Call from the U privilege level, that is, the system call. Here we first modify the sepc in the Trap context saved on the kernel stack to increase it by 4. This is because we know that this is a system call triggered by the ``ecall`` instruction. When entering the Trap, the hardware will set sepc to the address of the ``ecall`` instruction (because it is the address before entering the Trap last executed instruction). And after Trap returns, we want the application control flow to start executing from the next instruction of ``ecall``. Therefore, we only need to modify the sepc in the Trap context to increase the code length of the ``ecall`` instruction, which is 4 bytes. In this way, during ``__restore``, sepc will point to the next instruction of ``ecall`` after restoration, and execute from there after ``sret``.

The a0 register, which is used to hold the return value of the system call, will also change. We take a7 as the syscall ID and the three parameters a0~a2 of the system call from the Trap context, pass them to ``syscall`` function and get the return value. ``syscall`` functions are implemented in the ``syscall`` submodule. This code is the control logic that handles normal system calls.

- Lines 12~20 deal with memory access errors and illegal instruction errors in the application respectively. At this time, you need to print the error message and call ``run_next_app`` to switch and run the next application directly.
- Starting from line 21, when encountering a trap type that is not yet supported, the entire "Dunkleosteus" batch operating system panics and exits.

.. - 第 8~11 行，发现触发 Trap 的原因是来自 U 特权级的 Environment Call，也就是系统调用。这里我们首先修改保存在内核栈上的 Trap 上下文里面 sepc，让其增加 4。这是因为我们知道这是一个由 ``ecall`` 指令触发的系统调用，在进入 Trap 的时候，硬件会将 sepc 设置为这条 ``ecall`` 指令所在的地址（因为它是进入 Trap 之前最后一条执行的指令）。而在 Trap 返回之后，我们希望应用程序控制流从 ``ecall`` 的下一条指令开始执行。因此我们只需修改 Trap 上下文里面的 sepc，让它增加 ``ecall`` 指令的码长，也即 4 字节。这样在 ``__restore`` 的时候 sepc 在恢复之后就会指向 ``ecall`` 的下一条指令，并在 ``sret`` 之后从那里开始执行。

  用来保存系统调用返回值的 a0 寄存器也会同样发生变化。我们从 Trap 上下文取出作为 syscall ID 的 a7 和系统调用的三个参数 a0~a2 传给 ``syscall`` 函数并获取返回值。 ``syscall`` 函数是在 ``syscall`` 子模块中实现的。 这段代码是处理正常系统调用的控制逻辑。
.. - 第 12~20 行，分别处理应用程序出现访存错误和非法指令错误的情形。此时需要打印错误信息并调用 ``run_next_app`` 直接切换并运行下一个应用程序。
.. - 第 21 行开始，当遇到目前还不支持的 Trap 类型的时候，“邓式鱼” 批处理操作系统整个 panic 报错退出。



.. 实现系统调用功能

Implement the System Calls
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For system calls, the ``syscall`` function does not actually process system calls, but only dispatch them to specific processing functions according to the syscall ID:

.. 对于系统调用而言， ``syscall`` 函数并不会实际处理系统调用，而只是根据 syscall ID 分发到具体的处理函数：

.. code-block:: rust
    :linenos:

    // os/src/syscall/mod.rs

    pub fn syscall(syscall_id: usize, args: [usize; 3]) -> isize {
        match syscall_id {
            SYSCALL_WRITE => sys_write(args[0], args[1] as *const u8, args[2]),
            SYSCALL_EXIT => sys_exit(args[0] as i32),
            _ => panic!("Unsupported syscall_id: {}", syscall_id),
        }
    }

Here we will convert the incoming parameter ``args`` into a type that can be accepted by the specific system call handler. Their implementation is very simple:

.. 这里我们会将传进来的参数 ``args`` 转化成能够被具体的系统调用处理函数接受的类型。它们的实现都非常简单：

.. code-block:: rust
    :linenos:

    // os/src/syscall/fs.rs

    const FD_STDOUT: usize = 1;

    pub fn sys_write(fd: usize, buf: *const u8, len: usize) -> isize {
        match fd {
            FD_STDOUT => {
                let slice = unsafe { core::slice::from_raw_parts(buf, len) };
                let str = core::str::from_utf8(slice).unwrap();
                print!("{}", str);
                len as isize
            },
            _ => {
                panic!("Unsupported fd in sys_write!");
            }
        }
    }

    // os/src/syscall/process.rs

    pub fn sys_exit(xstate: i32) -> ! {
        println!("[kernel] Application exited with code {}", xstate);
        run_next_app()
    }

- ``sys_write`` We convert the incoming start address and length of the buffer located in the application into a string ``&str``, and then use the ``print!`` macro already implemented by the batch operating system print it out. Note that we have not checked the security of the incoming parameters here, even if there is a panic when the error is serious, there will still be security risks. Here we do not make repairs for the convenience of implementation.
- ``sys_exit`` prints the return value of the exited application and also calls ``run_next_app`` to switch to the next application.

.. - ``sys_write`` 我们将传入的位于应用程序内的缓冲区的开始地址和长度转化为一个字符串 ``&str`` ，然后使用批处理操作系统已经实现的 ``print!`` 宏打印出来。注意这里我们并没有检查传入参数的安全性，即使会在出错严重的时候 panic，还是会存在安全隐患。这里我们出于实现方便暂且不做修补。
.. - ``sys_exit`` 打印退出的应用程序的返回值并同样调用 ``run_next_app`` 切换到下一个应用程序。

.. _ch2-app-execution:

.. 执行应用程序

Execute the Application
-------------------------------------

When the initialization of the batch operating system is completed, or when an application ends or an error occurs, we need to call the ``run_next_app`` function to switch to the next application. At this point the CPU is running at the S privilege level, and it wants to be able to switch to the U privilege level. In the RISC-V architecture, the only way to lower the CPU privilege level is to execute the privileged instructions returned by Trap, such as ``sret``, ``mret``, etc. In fact, before returning from the operating system kernel to running applications, the following tasks must be completed:

.. 当批处理操作系统初始化完成，或者是某个应用程序运行结束或出错的时候，我们要调用 ``run_next_app`` 函数切换到下一个应用程序。此时 CPU 运行在 S 特权级，而它希望能够切换到 U 特权级。在 RISC-V 架构中，唯一一种能够使得 CPU 特权级下降的方法就是执行 Trap 返回的特权指令，如 ``sret`` 、``mret`` 等。事实上，在从操作系统内核返回到运行应用程序之前，要完成如下这些工作：

- Construct the Trap context needed for the application to start executing;
- Through the ``__restore`` function, restore part of the registers executed by the application from the newly constructed Trap context;
- Set the value of the ``sepc`` CSR to the application entry point ``0x80400000``;
- switch ``scratch`` and ``sp`` registers, set ``sp`` to point to the application user stack;
- Execute ``sret`` to switch from S privilege level to U privilege level.

.. - 构造应用程序开始执行所需的 Trap 上下文；
.. - 通过 ``__restore`` 函数，从刚构造的 Trap 上下文中，恢复应用程序执行的部分寄存器；
.. - 设置 ``sepc`` CSR的内容为应用程序入口点 ``0x80400000``；
.. - 切换 ``scratch`` 和 ``sp`` 寄存器，设置 ``sp`` 指向应用程序用户栈；
.. - 执行 ``sret`` 从 S 特权级切换到 U 特权级。


They can achieve the above work more easily by reusing the code of ``__restore``. We only need to push a Trap context specially constructed for starting the application on the kernel stack, and then pass the ``__restore`` function. So that these registers can reach the context state required to start the application.

.. 它们可以通过复用 ``__restore`` 的代码来更容易的实现上述工作。我们只需要在内核栈上压入一个为启动应用程序而特殊构造的 Trap 上下文，再通过 ``__restore`` 函数，就能让这些寄存器到达启动应用程序所需要的上下文状态。

.. code-block:: rust
    :linenos:

    // os/src/trap/context.rs

    impl TrapContext {
        pub fn set_sp(&mut self, sp: usize) { self.x[2] = sp; }
        pub fn app_init_context(entry: usize, sp: usize) -> Self {
            let mut sstatus = sstatus::read();
            sstatus.set_spp(SPP::User);
            let mut cx = Self {
                x: [0; 32],
                sstatus,
                sepc: entry,
            };
            cx.set_sp(sp);
            cx
        }
    }

Implement the ``app_init_context`` method for ``TrapContext``-- modify the sepc register as the application entry point ``entry``. The sp register is a stack pointer we set, and set the ``SPP`` of the sstatus register field is set to User.

.. 为 ``TrapContext`` 实现 ``app_init_context`` 方法，修改其中的 sepc 寄存器为应用程序入口点 ``entry``， sp 寄存器为我们设定的一个栈指针，并将 sstatus 寄存器的 ``SPP`` 字段设置为 User 。

.. 在 ``run_next_app`` 函数中我们能够看到：

In the ``run_next_app`` function we can see:

.. code-block:: rust
    :linenos:
    :emphasize-lines: 14,15,16,17,18

    // os/src/batch.rs

    pub fn run_next_app() -> ! {
        let mut app_manager = APP_MANAGER.exclusive_access();
        let current_app = app_manager.get_current_app();
        unsafe {
            app_manager.load_app(current_app);
        }
        app_manager.move_to_next_app();
        drop(app_manager);
        // before this we have to drop local variables related to resources manually
        // and release the resources
        extern "C" { fn __restore(cx_addr: usize); }
        unsafe {
            __restore(KERNEL_STACK.push_context(
                TrapContext::app_init_context(APP_BASE_ADDRESS, USER_STACK.get_sp())
            ) as *const _ as usize);
        }
        panic!("Unreachable in batch::run_current_app!");
    }

What is done in the highlighted line is to push a Trap context on the kernel stack -- its ``sepc`` is the application entry address ``0x80400000``, its ``sp`` register points to the user stack, and its ``SPP`` field of ``sstatus`` is set to User. The return value of ``push_context`` is the top of the stack after the kernel stack is pushed into the Trap context, and it will be used as the parameter of ``__restore`` (see back :ref:`__restore code <code-restore>`. At this point we can understand why the initial part of the ``__restore`` function will be completed :math:`\text{sp}\leftarrow\text{a}_0` ), which makes the ``__restore`` function ``sp`` can still point to the top of the kernel stack. After that, it's the same as doing a normal ``__restore`` function call.

.. 在高亮行所做的事情是在内核栈上压入一个 Trap 上下文，其 ``sepc`` 是应用程序入口地址 ``0x80400000`` ，其 ``sp`` 寄存器指向用户栈，其 ``sstatus`` 的 ``SPP`` 字段被设置为 User 。``push_context`` 的返回值是内核栈压入 Trap 上下文之后的栈顶，它会被作为 ``__restore`` 的参数（回看 :ref:`__restore 代码 <code-restore>` ，这时我们可以理解为何 ``__restore`` 函数的起始部分会完成 :math:`\text{sp}\leftarrow\text{a}_0` ），这使得在 ``__restore`` 函数中 ``sp`` 仍然可以指向内核栈的栈顶。这之后，就和执行一次普通的 ``__restore`` 函数调用一样了。

.. note::

    Interested students can think: When was sscratch set to the top of the kernel stack?

    .. 有兴趣的同学可以思考： sscratch 是何时被设置为内核栈顶的？



.. 
   马老师发生甚么事了？
   --
   这里要说明目前只考虑从 U Trap 到 S ，而实际上 Trap 的要素就有：Trap 之前在哪个特权级，Trap 在哪个特权级处理。这个对于中断和异常
   都是如此，只不过中断可能跟特权级的关系稍微更紧密一点。毕竟中断的类型都是跟特权级挂钩的。但是对于 Trap 而言有一点是共同的，也就是触发 
   Trap 不会导致优先级下降。从中断/异常的代理就可以看出从定义上就不允许代理到更低的优先级。而且代理只能逐级代理，目前我们能操作的只有从 
   M 代理到 S，其他代理都基本只出现在指令集拓展或者硬件还不支持。中断的情况是，如果是属于某个特权级的中断，不能在更低的优先级处理。事实上
   这个中断只可能在 CPU 处于不会更高的优先级上收到（否则会被屏蔽），而 Trap 之后优先级不会下降（Trap 代理机制决定），这样就自洽了。
   --
   之前提到异常是说需要执行环境功能的原因与某条指令的执行有关。而 Trap 的定义更加广泛一些，就是在执行某条指令之后发现需要执行环境的功能，
   如果是中断的话 Trap 回来之后默认直接执行下一条指令，如果是异常的话硬件会将 sepc 设置为 Trap 发生之前最后执行的那条指令，而异常发生
   的原因不一定和这条指令的执行有关。应该指出的是，在大多数情况下都是和最后这条指令的执行有关。但在缓存的作用下也会出现那种特别极端的情况。
   --
   然后是 Trap 到 S，就有 S 模式的一些相关 CSR，以及从 U Trap 到 S，硬件会做哪些事情（包括触发异常的一瞬间，以及处理完成调用 sret 
   之后）。然后指出从用户的视角来看，如果是 ecall 的话， Trap 回来之后应该从 ecall 的下一条指令开始执行，且执行现场不能发生变化。
   所以就需要将应用执行环境保存在内核栈上（还需要换栈！）。栈存在的原因可能是 Trap handler 是一条新的运行在 S 特权级的执行流，所以
   这个可以理解成跨特权级的执行流切换，确实就复杂一点，要保存的内容也相对多一点。而下一章多任务的任务切换是全程发生在 S 特权级的执行流
   切换，所以会简单一点，保存的通用寄存器大概率更少（少在调用者保存寄存器），从各种意义上都很像函数调用。从不同特权级的角度来解释换栈
   是出于安全性，应用不应该看到 Trap 执行流的栈，这样做完之后，虽然理论上可以访问，但应用不知道内核栈的位置应该也有点麻烦。
   --
   然后是 rust_trap 的处理，尤其是奇妙的参数传递，内部处理逻辑倒是非常简单。
   --
   最后是如何利用 __restore 初始化应用的执行环境，包括如何设置入口点、用户栈以及保证在 U 特权级执行。





