.. 为内核支持函数调用

Support Function Calls for Kernel
================================================

.. toctree::
   :hidden:
   :maxdepth: 5

.. 本节导读

Section Guide
--------------------------------------------------

In the previous section, we successfully executed the first instruction of the kernel on Qemu, by handwriting the assembly code in ``entry.asm``. However, we don't want to write our kernel by handwriting assembly code anyway, and we want to use Rust language to implement most of the functions. However, in order to transfer control to our kernel entry function written in Rust language, we do need to hand-write several lines of assembly code to perform certain initialization work. As before, these assembly codes are placed in ``entry.asm`` and are executed first before control is passed to kernel-related functions. But these functions will be more complicated. First, you need to set up the stack space to enable function calls in the kernel, and then directly call the kernel entry function written in Rust, so that the control is handed over to the Rust code. This is the third step in building the "Trilobite" operating system.

Before the specific operation, we will first introduce the background knowledge of many function calls and stacks. This knowledge is important, and there are some ideas that carry over into later chapters. But at the same time, this knowledge is relatively basic, so we gave a list of knowledge points before the introduction. Readers with a certain foundation can refer to this list for selective reading.

.. 上一节我们成功在 Qemu 上执行了内核的第一条指令，它是我们在 ``entry.asm`` 中手写汇编代码得到的。然而，我们无论如何也不想仅靠手写汇编代码的方式编写我们的内核，绝大部分功能我们都想使用 Rust 语言来实现。不过为了将控制权转交给我们使用 Rust 语言编写的内核入口函数，我们确实需要手写若干行汇编代码进行一定的初始化工作。和之前一样，这些汇编代码放在 ``entry.asm`` 中，并在控制权被转交给内核相关函数前最先被执行，但它们的功能会更加复杂。首先需要设置栈空间，来在内核内使能函数调用，随后直接调用使用 Rust 编写的内核入口函数，从而控制权便被移交给 Rust 代码。这就是构建“三叶虫”操作系统的第三个步骤。

.. 在具体操作之前，我们首先会介绍很多函数调用和栈的背景知识。这些知识很重要，而且有一些思想会一直延续到后面的章节。但同时这些知识相对比较基础，因此我们在正式开始介绍之前给出了一个知识点清单，有一定基础的读者可以参照此清单进行选读。

.. note::

    **Knowledge list in this section**

    Please try to answer the following questions, and if you are confident enough with your answer, you can go directly to the :ref:`practice part <jump-practice>` of this section.

    - How to jump to the next instruction that called the function when the function returns, even if the function is called in multiple places in the code?
    - WWhat is the rationale to guarantee that the value of (some) general-purpose registers remains unchanged before and after it calls a function? 
    - How does the caller function and the callee function cooperate to ensure that the register states remain unchanged before and after calling the sub-function? Who is responsible for saving and restoring caller-saved and callee-saved registers? Where are they temporarily stored? When are they saved and restored (eg function prologue/epilogue)?
    - On the RISC-V architecture, how are caller-saved and callee-saved registers divided?
    - `sp` and `ra` are caller- or callee-saved registers, why this convention?
    - How to use registers to pass parameters and return values ​​of function calls? If the number of registers is not enough, how to pass the parameters of the function call?

    .. **本节知识清单**

    .. 请尝试回答以下问题，如果对于自己的答案足够自信的话则可以直接进入本节的 :ref:`实践部分 <jump-practice>` 。

    .. - 如何使得函数返回时能够跳转到调用该函数的下一条指令，即使该函数在代码中的多个位置被调用？
    .. - 对于一个函数而言，保证它调用某个子函数之前，以及该子函数返回到它之后（某些）通用寄存器的值保持不变有何意义？
    .. - 调用者函数和被调用者函数如何合作保证调用子函数前后寄存器内容保持不变？调用者保存和被调用者保存寄存器的保存与恢复各自由谁负责？它们暂时被保存在什么位置？它们于何时被保存和恢复（如函数的开场白/退场白）？
    .. - 在 RISC-V 架构上，调用者保存和被调用者保存寄存器如何划分的？ 
    .. - `sp` 和 `ra` 是调用者还是被调用者保存寄存器，为什么这样约定？
    .. - 如何使用寄存器传递函数调用的参数和返回值？如果寄存器数量不够用了，如何传递函数调用的参数？

.. _term-function-call-and-stack:

.. 函数调用与栈

Function calls and the stack
--------------------------------

Looking at the execution of a program from the level of assembly instructions, if the physical address sequence of the instructions executed by the CPU in sequence is :math:`\{a_n\}`, then what kind of pattern will this sequence conform to?

.. 从汇编指令的级别看待一段程序的执行，假如 CPU 依次执行的指令的物理地址序列为 :math:`\{a_n\}`，那么这个序列会符合怎样的模式呢？

.. _term-control-flow:

The simplest of them is undoubtedly that the CPU executes instructions one by one continuously, that is, it satisfies the recursive formula: math:`a_{n+1}=a_n+L`. Here we assume that the instructions of the platform are fixed-length and uniform is :math:`L` bytes (2/4 bytes in the common case). But the execution sequence doesn't always follow this pattern, and the pattern can be broken when the instruction at physical address :math:`a_n` is a jump instruction. Jump instructions correspond to many different structures of the **Control Flow** (Control Flow) we construct in the program, such as branch structures (such as if/switch statements) and loop structures (such as for/while statements). The jump instruction used to realize the above two structures only needs to realize the jump function, that is, set the pc register to a specified address.

.. 其中最简单的无疑就是 CPU 一条条连续向下执行指令，也即满足递推公式 :math:`a_{n+1}=a_n+L`，这里我们假设该平台的指令是定长的且均为 :math:`L` 字节（常见情况为 2/4 字节）。但是执行序列并不总是符合这种模式，当位于物理地址 :math:`a_n` 的指令是一条跳转指令的时候，该模式就有可能被破坏。跳转指令对应于我们在程序中构造的 **控制流** (Control Flow) 的多种不同结构，比如分支结构（如 if/switch 语句）和循环结构（如 for/while 语句）。用来实现上述两种结构的跳转指令，只需实现跳转功能，也就是将 pc 寄存器设置到一个指定的地址即可。

.. _term-function-call:

Another control flow structure is more complicated: **Function Call** (Function Call). We probably know the order of code execution in the entire process of calling a function. If we look at it from the source code level, we will execute the code of the called function. After it returns, we will return to the next line of the corresponding statement of the calling function. Continue to execute. So how do we use assembly instructions to achieve this process? First of all, when calling, there needs to be an instruction to jump to the position of the called function, which looks no different from other control structures; but when the called function returns, we need to jump back to the instruction next to the one that initiates the jump and carry on the execution. Where to jump back is not known until the corresponding function call occurs. For example, if we call the same function in two different places, obviously the function will return to different addresses after returning. This is a big difference: other control flow only needs to jump to a *compile-time fixed* address, while the return jump of a function call is a jump to a *run-time fixed* (exactly at address when the function call occurs).

.. 另一种控制流结构则显得更为复杂： **函数调用** (Function Call)。我们大概清楚调用函数整个过程中代码执行的顺序，如果是从源代码级的视角来看，我们会去执行被调用函数的代码，等到它返回之后，我们会回到调用函数对应语句的下一行继续执行。那么我们如何用汇编指令来实现这一过程？首先在调用的时候，需要有一条指令跳转到被调用函数的位置，这个看起来和其他控制结构没什么不同；但是在被调用函数返回的时候，我们却需要返回那条跳转过来的指令的下一条继续执行。这次用来返回的跳转究竟跳转到何处，在对应的函数调用发生之前是不知道的。比如，我们在两个不同的地方调用同一个函数，显然函数返回之后会回到不同的地址。这是一个很大的不同：其他控制流都只需要跳转到一个 *编译期固定下来* 的地址，而函数调用的返回跳转是跳转到一个 *运行时确定* （确切地说是在函数调用发生的时候）的地址。


.. image:: function-call.png
   :align: center
   :name: function-call

For this, the instruction set has to give some extra capability to the jump instructions used for function calls, not just jumps. On the RISC-V architecture, there are two instructions that meet such characteristics:

.. 对此，指令集必须给用于函数调用的跳转指令一些额外的能力，而不只是单纯的跳转。在 RISC-V 架构上，有两条指令即符合这样的特征：

.. list-table:: RISC-V 函数调用跳转指令
   :widths: 20 30
   :header-rows: 1
   :align: center


   * - Instruction
     - Instruction Functionality
   * - :math:`\text{jal}\ \text{rd},\ \text{imm}[20:1]`
     - :math:`\text{rd}\leftarrow\text{pc}+4`

       :math:`\text{pc}\leftarrow\text{pc}+\text{imm}`
   * - :math:`\text{jalr}\ \text{rd},\ (\text{imm}[11:0])\text{rs}`
     - :math:`\text{rd}\leftarrow\text{pc}+4`
       
       :math:`\text{pc}\leftarrow\text{rs}+\text{imm}`

.. _term-source-register:
.. _term-immediate:
.. _term-destination-register:

.. note::

    **The meaning of each part of the RISC-V instruction**

   In most instructions that only deal with general-purpose registers, rs stands for **Source Register** (Source Register), and imm stands for **immediate number** (Immediate), which is a constant, and the two constitute the input part of the instruction; And rd stands for **Target Register** (Destination Register), which is the output part of the instruction. rs and rd can be selected from 32 general-purpose registers x0~x31. But none of these three parts is necessary, some instructions have only one input type, and others have no output part.

..    **RISC-V 指令各部分含义**

..    在大多数只与通用寄存器打交道的指令中， rs 表示 **源寄存器** (Source Register)， imm 表示 **立即数** (Immediate)，是一个常数，二者构成了指令的输入部分；而 rd 表示 **目标寄存器** (Destination Register)，它是指令的输出部分。rs 和 rd 可以在 32 个通用寄存器 x0~x31 中选取。但是这三个部分都不是必须的，某些指令只有一种输入类型，另一些指令则没有输出部分。

.. _term-pseudo-instruction:

It can be seen from the figure that these two instructions save the address of the next instruction of the current jump instruction in the rd register before setting the pc register to complete the jump function, namely: :math:`\text{rd}\leftarrow\text {pc}+4` is the meaning of this instruction. (It is assumed here that the length of all instructions is 4 bytes). In the RISC-V architecture, the ``ra`` register (that is, the ``x1`` register) is usually used as the specific register corresponding to ``rd``. So when the function returns, just jump back to the address saved by ``ra``. In fact, when the function returns, we often use a **assembly pseudo-instruction** (Pseudo Instruction) to jump back to the position before the call: ``ret``. It will be translated by the assembler as ``jalr x0, 0(x1)``, meaning to jump to the physical address saved by the register ``ra``. Because ``x0`` is a constant ``0`` registers, the step of saving in ``rd`` is omitted.

.. 从中可以看出，这两条指令在设置 pc 寄存器完成跳转功能之前，还将当前跳转指令的下一条指令地址保存在 rd 寄存器中，即 :math:`\text{rd}\leftarrow\text{pc}+4` 这条指令的含义。（这里假设所有指令的长度均为 4 字节）在 RISC-V 架构中，通常使用 ``ra`` 寄存器（即 ``x1`` 寄存器）作为其中的 ``rd`` 对应的具体寄存器，因此在函数返回的时候，只需跳转回 ``ra``  所保存的地址即可。事实上在函数返回的时候我们常常使用一条 **汇编伪指令** (Pseudo Instruction) 跳转回调用之前的位置： ``ret`` 。它会被汇编器翻译为 ``jalr x0, 0(x1)``，含义为跳转到寄存器 ``ra`` 保存的物理地址，由于 ``x0`` 是一个恒为 ``0`` 的寄存器，在 ``rd`` 中保存这一步被省略。

To sum up, when making a function call, we use the ``jalr`` instruction to save the return address and implment the jump; and when the function is about to return, we use the ``ret`` pseudo-instruction to return to the next address before the jump to carry on the execution. In this way, these two instructions of RISC-V realize the core mechanism of the function call process.

Since we save the return address in the ``ra`` register, we need to ensure that it does not change throughout the execution of the function, otherwise it will jump to the wrong location after ``ret``. In fact, the compiler does basically not use the ``ra`` register except for the related instructions of function calls. That is to say, if no other functions are called in the function, the value of ``ra`` will not change, and the function call process can work normally. But unfortunately, when we actually write code, we often encounter the situation of function **multi-level nested call**. We can easily imagine how complicated programming would be if functions did not support nested calls. If we try to call a sub-function in a function: :math:`f`, while jumping to the sub-function: :math:`g`, ra will be overwritten with the address of the next jump instruction, and The return address of function :math:`f` previously saved by ra will be `permanently lost`.

.. 总结一下，在进行函数调用的时候，我们通过 ``jalr`` 指令保存返回地址并实现跳转；而在函数即将返回的时候，则通过 ``ret`` 伪指令回到跳转之前的下一条指令继续执行。这样，RISC-V 的这两条指令就实现了函数调用流程的核心机制。

.. 由于我们是在 ``ra`` 寄存器中保存返回地址的，我们要保证它在函数执行的全程不发生变化，不然在 ``ret`` 之后就会跳转到错误的位置。事实上编译器除了函数调用的相关指令之外确实基本上不使用 ``ra`` 寄存器。也就是说，如果在函数中没有调用其他函数，那 ``ra`` 的值不会变化，函数调用流程能够正常工作。但遗憾的是，在实际编写代码的时候我们常常会遇到函数 **多层嵌套调用** 的情形。我们很容易想象，如果函数不支持嵌套调用，那么编程将会变得多么复杂。如果我们试图在一个函数 :math:`f` 中调用一个子函数，在跳转到子函数 :math:`g` 的同时，ra 会被覆盖成这条跳转指令的下一条的地址，而 ra 之前所保存的函数 :math:`f` 的返回地址将会 `永久丢失` 。 

.. _term-function-context:
.. _term-activation-record:

Therefore, if we want to correctly implement the control flow of nested function calls, we must ensure in some way: before and after a function calls a sub-function, the value of the ``ra`` register cannot change. But in fact, this is not limited to a single ``ra`` register, but applies to all general-purpose registers. This is because the compiler compiles each function independently, so a function cannot know which registers are modified by the subfunction it calls. From the perspective of a function, the value of some registers being overwritten in the process of calling a sub-function will indeed affect its subsequent execution. So this is necessary. We refer to the set of registers that need to remain unchanged before and after the control flow transfer due to the function call as **Function Call Context** (Function Call Context).

.. 因此，若想正确实现嵌套函数调用的控制流，我们必须通过某种方式保证：在一个函数调用子函数的前后，``ra`` 寄存器的值不能发生变化。但实际上，这并不仅仅局限于 ``ra`` 一个寄存器，而是作用于所有的通用寄存器。这是因为，编译器是独立编译每个函数的，因此一个函数并不能知道它所调用的子函数修改了哪些寄存器。而站在一个函数的视角，在调用子函数的过程中某些寄存器的值被覆盖的确会对它接下来的执行产生影响。因此这是必要的。我们将由于函数调用，在控制流转移前后需要保持不变的寄存器集合称之为 **函数调用上下文** (Function Call Context) 。

.. _term-save-restore:

Since each CPU has only one set of registers, if we want to keep the function call context unchanged before and after the subfunction call, we need the help of physical memory. To be precise, before calling a subfunction, we need to **Save** the registers in the function call context in an area in physical memory; after the function is executed, we will read from the same area in memory and **Restore** (Restore) registers in the context of a function call. In fact, this work is done cooperatively by the caller of the subfunction and the callee (that is, the subfunction itself). The registers in the function call context are divided into the following two categories:

.. 由于每个 CPU 只有一套寄存器，我们若想在子函数调用前后保持函数调用上下文不变，就需要物理内存的帮助。确切的说，在调用子函数之前，我们需要在物理内存中的一个区域 **保存** (Save) 函数调用上下文中的寄存器；而在函数执行完毕后，我们会从内存中同样的区域读取并 **恢复** (Restore) 函数调用上下文中的寄存器。实际上，这一工作是由子函数的调用者和被调用者（也就是子函数自身）合作完成。函数调用上下文中的寄存器被分为如下两类：

.. _term-callee-saved:
.. _term-caller-saved:

- **Callee-Saved Registers**: The called function may overwrite these registers. These registers  need to be saved by the called function. Equivalently, the called function ensures that these registers  remain unchanged before and after the call. 
- **Caller-Saved Registers**: The called function may overwrite these registers. These registers need to be saved by the caller function. Equivalently, the caller function ensures that these registers  remain unchanged before and after the call. 

As can be seen from the name, the function call context is saved separately by the caller and the callee, and the specific processes are as follows:

- Calling a function: first save the **caller-saved registers** that do not want to change during the function call, then call the sub-function through the jal/jalr instruction, and restore these registers after returning.
- Called function: At the beginning of the called function, first save the **callee-saved registers** used during function execution, then execute the function, and finally restore these registers before the function exits.

.. - **被调用者保存(Callee-Saved) 寄存器** ：被调用的函数可能会覆盖这些寄存器，需要被调用的函数来保存的寄存器，即由被调用的函数来保证在调用前后，这些寄存器保持不变；
.. - **调用者保存(Caller-Saved) 寄存器** ：被调用的函数可能会覆盖这些寄存器，需要发起调用的函数来保存的寄存器，即由发起调用的函数来保证在调用前后，这些寄存器保持不变。

.. 从名字中可以看出，函数调用上下文由调用者和被调用者分别保存，其具体过程分别如下：

.. - 调用函数：首先保存不希望在函数调用过程中发生变化的 **调用者保存寄存器** ，然后通过 jal/jalr 指令调用子函数，返回之后恢复这些寄存器。
.. - 被调用函数：在被调用函数的起始，先保存函数执行过程中被用到的 **被调用者保存寄存器** ，然后执行函数，最后在函数退出之前恢复这些寄存器。

.. _term-prologue:
.. _term-epilogue:

We found that whether it is the calling function or the called function, a pair of corresponding  assembly codes to save and restore registers are required due to the calling behavior, which can be called Prologue and Epilogue. They will be automatically inserted by the compiler for us to complete the preservation and restoration of related registers. A function may call other functions as a caller, and may be called by other functions as a callee.

.. 我们发现无论是调用函数还是被调用函数，都会因调用行为而需要两段匹配的保存和恢复寄存器的汇编代码，可以分别将其称为 **开场** (Prologue) 和 **结尾** (Epilogue)，它们会由编译器帮我们自动插入，来完成相关寄存器的保存与恢复。一个函数既有可能作为调用者调用其他函数，也有可能作为被调用者被其他函数调用。


.. chyyuu

  对于这个函数而言，如果在执行的时候需要修改被调用者保存寄存器，而必须在函数开头的开场和结尾处进行保存；对于调用者保存寄存器则可以没有任何顾虑的随便使用，因为它在约定中本就不需要承担保证调用者保存寄存器保持不变的义务。

.. note::


    **Register saving and compiler optimization**

   It's worth noting here that the caller and callee actually only need to save a subset of the caller-saved registers and callee-saved registers, respectively, as needed. For the caller, when the sub-function is called, even if the sub-function modifies the caller-saved registers, the code inserted by the compiler in the calling function will restore these registers; for the callee function, during its execution, some callee-saved registers do not need to be saved if they are not used at all. When the compiler performs back-end code generation, it knows which registers are worth saving in these two scenarios. From this point of view, we can also understand why the function call context is divided into two categories: the compiler can optimize some useless register saving and restoration operations as early as possible, and improve the execution performance of the program.

..    **寄存器保存与编译器优化**

..    这里值得说明的是，调用者和被调用者实际上只需分别按需保存调用者保存寄存器和被调用者保存寄存器的一个子集。对于调用函数而言，在调用子函数的时候，即使子函数修改了调用者保存寄存器，编译器在调用函数中插入的代码会恢复这些寄存器；而对于被调用函数而言，在其执行过程中没有使用到的被调用者保存寄存器也无需保存。编译器在进行后端代码生成时，知道在这两个场景中分别有哪些值得保存的寄存器。从这一角度也可以理解为何要将函数调用上下文分成两类：可以让编译器尽可能早地优化掉一些无用的寄存器保存与恢复操作，提高程序的执行性能。

.. chyyuu 最好有个例子说明

.. _term-calling-convention:


.. 调用规范

Calling Convention
------------------------

**Calling Convention** agrees on how to implement a function call in a programming language on a certain instruction set architecture. It includes the following:

1. How to pass the input parameters and return value of the function;
2. The division of caller/callee saved registers in the function call context;
3. Other methods of using registers in the function call process.

The calling convention is for a certain programming language, because the function call in the general sense will only be performed inside the programming language. When a language wants to call a function interface written in another programming language, the compiler needs to understand the calling specifications of the two languages ​​at the same time, and adjust the use of registers.

.. **调用规范** (Calling Convention) 约定在某个指令集架构上，某种编程语言的函数调用如何实现。它包括了以下内容：

.. 1. 函数的输入参数和返回值如何传递；
.. 2. 函数调用上下文中调用者/被调用者保存寄存器的划分；
.. 3. 其他的在函数调用流程中对于寄存器的使用方法。

.. 调用规范是对于一种确定的编程语言来说的，因为一般意义上的函数调用只会在编程语言的内部进行。当一种语言想要调用用另一门编程语言编写的函数接口时，编译器就需要同时清楚两门语言的调用规范，并对寄存器的使用做出调整。

.. note::

    **C language call specification on RISC-V architecture**

   The C language calling specification on RISC-V architecture can be found `here <https://riscv.org/wp-content/uploads/2015/01/riscv-calling.pdf>`_.
   It makes the following conventions for the use of general-purpose registers:

   .. list-table:: RISC-V register function classification
      :widths: 20 20 40
      :align: center
      :header-rows: 1

      * - registers
        - saver
        - functionalities
      * - a0~a7 ( ``x10~x17`` )
        - caller save
        - Used to pass input parameters. Among them, a0 and a1 are also used to save the return value.
      * - t0~t6( ``x5~x7,x28~x31`` )
        - caller save
        - Used as a temporary register, it can be used freely in the called function without saving.
      * - s0~s11( ``x8~x9,x18~x27`` )
        - callee save
        - Used as a temporary register, the called function can only be used in the called function after it is saved.


   The remaining 5 general-purpose registers are as follows:

   - zero( ``x0`` ) As mentioned before, it is always zero, and the function call will not affect it;
   - ra( ``x1`` ) is callee-saved. The callee function may also call the function, and ``ra`` needs to be modified before the call so that the call can return correctly. Therefore, each function needs to save ``ra`` in its own stack frame at the beginning, and restore it before returning with ``ret`` at the end. A stack frame is a memory structure used by the currently executing function to store local variables and function return information.
   - sp( ``x2`` ) is callee-saved. This is the stack pointer (Stack Pointer) register that will be mentioned later, which points to the next top of the stack to be stored.
   - fp( ``s0`` ), it can be used as both the s0 temporary register and the stack frame pointer (Frame Pointer) register. If latter, it indicates the starting position of the current stack frame and is a callee-saved register. The starting position of the stack frame pointed to by fp and the current top position of the stack frame pointed to by sp delimit the space range of the corresponding function stack frame.
   - Both gp( ``x3`` ) and tp( ``x4`` ) do not change during a program run, so they do not have to be placed in a function call context. Their use will be mentioned in later chapters.

   For more detailed content, please refer to the `CS 3410: Computer System Organization and Programming courseware content of Cornell University <http://www.cs.cornell.edu/courses/cs3410/2019sp/schedule/slides/10-calling-notes-bw .pdf>`_.


..    **RISC-V 架构上的 C 语言调用规范**

..    RISC-V 架构上的 C 语言调用规范可以在 `这里 <https://riscv.org/wp-content/uploads/2015/01/riscv-calling.pdf>`_ 找到。
..    它对通用寄存器的使用做出了如下约定：

..    .. list-table:: RISC-V 寄存器功能分类
..       :widths: 20 20 40
..       :align: center
..       :header-rows: 1

..       * - 寄存器组
..         - 保存者
..         - 功能
..       * - a0~a7（ ``x10~x17`` ）
..         - 调用者保存
..         - 用来传递输入参数。其中的 a0 和 a1 还用来保存返回值。
..       * - t0~t6( ``x5~x7,x28~x31`` )
..         - 调用者保存
..         - 作为临时寄存器使用，在被调函数中可以随意使用无需保存。
..       * - s0~s11( ``x8~x9,x18~x27`` )
..         - 被调用者保存
..         - 作为临时寄存器使用，被调函数保存后才能在被调函数中使用。

..    剩下的 5 个通用寄存器情况如下：

..    - zero( ``x0`` ) 之前提到过，它恒为零，函数调用不会对它产生影响；
..    - ra( ``x1`` ) 是被调用者保存的。被调用者函数可能也会调用函数，在调用之前就需要修改 ``ra`` 使得这次调用能正确返回。因此，每个函数都需要在开头保存 ``ra`` 到自己的栈帧中，并在结尾使用 ``ret`` 返回之前将其恢复。栈帧是当前执行函数用于存储局部变量和函数返回信息的内存结构。
..    - sp( ``x2`` ) 是被调用者保存的。这个是之后就会提到的栈指针 (Stack Pointer) 寄存器，它指向下一个将要被存储的栈顶位置。
..    - fp( ``s0`` )，它既可作为s0临时寄存器，也可作为栈帧指针（Frame Pointer）寄存器，表示当前栈帧的起始位置，是一个被调用者保存寄存器。fp 指向的栈帧起始位置 和 sp 指向的栈帧的当前栈顶位置形成了所对应函数栈帧的空间范围。 
..    - gp( ``x3`` ) 和 tp( ``x4`` ) 在一个程序运行期间都不会变化，因此不必放在函数调用上下文中。它们的用途在后面的章节会提到。

..    更加详细的内容可以参考 Cornell 大学的 `CS 3410: Computer System Organization and Programming 课件内容 <http://www.cs.cornell.edu/courses/cs3410/2019sp/schedule/slides/10-calling-notes-bw.pdf>`_ 。

.. _term-stack:
.. _term-stack-pointer:
.. _term-stack-frame:

Before we discussed the timing of saving/restoring the function call context and the selection of registers. But we didn't detail where these registers are saved -- we just used "an area in memory" to briefly mention it. Actually, its more accurate name is **Stack** (Stack). The ``sp`` register is often used to save the **Stack Pointer** (Stack Pointer), which points to the top address of the stack in memory. In the RISC-V architecture, the stack grows from high addresses to low addresses. In a function, the starting code is responsible for allocating a new stack space, that is, reducing the value of ``sp`` by the corresponding number of bytes, so the physical address range: math:`[\text{new sp},\text{old sp})` can be used by this function to save/restore the function call context. This piece of physical memory is called the **stack frame** of this function. Similarly, the tail code in the function is responsible for reclaiming the stack frame allocated by the opening code, which only needs to increase the value of ``sp`` by the same number of bytes to return to the state before allocation. This would also explain why ``sp`` is a callee-saved register.


.. 之前我们讨论了函数调用上下文的保存/恢复时机以及寄存器的选择，但我们并没有详细说明这些寄存器保存在哪里，只是用“内存中的一块区域”草草带过。实际上，它更确切的名字是 **栈** (Stack) 。  ``sp`` 寄存器常用来保存 **栈指针** (Stack Pointer)，它指向内存中栈顶地址。在 RISC-V 架构中，栈是从高地址向低地址增长的。在一个函数中，作为起始的开场代码负责分配一块新的栈空间，即将 ``sp`` 的值减小相应的字节数即可，于是物理地址区间 :math:`[\text{新sp},\text{旧sp})` 对应的物理内存的一部分便可以被这个函数用来进行函数调用上下文的保存/恢复，这块物理内存被称为这个函数的 **栈帧** (Stack Frame)。同理，函数中的结尾代码负责将开场代码分配的栈帧回收，这也仅仅需要将 ``sp`` 的值增加相同的字节数回到分配之前的状态。这也可以解释为什么 ``sp`` 是一个被调用者保存寄存器。

.. note::

  **Stack frame**
 
  We know that when a program executes a function call, the caller function and the called function use the same stack. Under normal circumstances, we don't need to distinguish which part of the stack is used by the caller function and the called function. However, this information is important when we need to debug or backtrace function calls during execution. Simply put, a stack frame is a part of the stack used by a function, and the stack frames of all functions are strung together to form a complete function call stack. Generally speaking, the two boundaries of the stack frame of the currently executing function are defined by the Stack Pointer register and the stack Frame Pointer register respectively.

..   **栈帧 stack frame**
 
..   我们知道程序在执行函数调用时，调用者函数和被调用函数使用的是同一个栈。在通常的情况下，我们并不需要区分调用者函数和被调用函数分别使用了栈的哪个部分。但是，当我们需要在执行过程中对函数调用进行调试或backtrace的时候，这一信息就很重要了。简单的说，栈帧（stack frame）就是一个函数所使用的栈的一部分区域，所有函数的栈帧串起来就组成了一个完整的函数调用栈。一般而言，当前执行函数的栈帧的两个边界分别由栈指针 (Stack Pointer)寄存器和栈帧指针（frame pointer）寄存器来限定。


.. figure:: CallStack.png
   :align: center

   Function call and stack frame: As shown in the figure, we can see the allocation/recycling of the stack frame and the change of ``sp`` register during the whole process of calling a, calling b, calling c, c returning, and b returning in the whole process .

    The blocks marked a/b/c in the figure represent the stack frames of functions a/b/c respectively.

..    函数调用与栈帧：如图所示，我们能够看到在程序依次调用 a、调用 b、调用 c、c 返回、b 返回整个过程中栈帧的分配/回收以及 ``sp`` 寄存器的变化。
..    图中标有 a/b/c 的块分别代表函数 a/b/c 的栈帧。

.. _term-lifo:

.. note::

    **The stack in the data structure and the stack for implementing the function call**

   From the perspective of data structure, the stack is a **Last In First Out** (Last In First Out, LIFO) linear table, which supports two operations of pushing an element to the top of the stack and popping an element from the top of the stack, called ``push`` and ``pop`` respectively. From the interface it provides, it only supports accessing elements near the top of the stack. Therefore, it is necessary to maintain a pointer to the top of the stack to indicate the current state of the stack during implementation.

    The principle of the stack here is the same as the stack in the data structure, and it can be one-to-one in many aspects. The stack pointer ``sp`` can correspond to the pointer pointing to the top of the stack, and the allocation/recycling of stack frames can correspond to ``push`` / ``pop`` operations respectively. If we regard our stack as a memory allocator, the reason why it can be so simple is that the memory it reclaims must be the *last allocated* memory. So only two operations like ``push`` / ``pop`` are needed.


..    **数据结构中的栈与实现函数调用所需要的栈**

..    从数据结构的角度来看，栈是一个 **后入先出** (Last In First Out, LIFO) 的线性表，支持向栈顶压入一个元素以及从栈顶弹出一个元素两种操作，分别被称为 push 和 pop。从它提供的接口来看，它只支持访问栈顶附近的元素。因此在实现的时候需要维护一个指向栈顶的指针来表示栈当前的状态。

..    我们这里的栈与数据结构中的栈原理相同，在很多方面可以一一对应。栈指针 ``sp`` 可以对应到指向栈顶的指针，对于栈帧的分配/回收可以分别对应到 ``push`` / ``pop`` 操作。如果将我们的栈看成一个内存分配器，它之所以可以这么简单，是因为它回收的内存一定是 *最近一次分配* 的内存，从而只需要类似 ``push`` / ``pop`` 的两种操作即可。

Under the appropriate compilation option settings, the stack frame content of a function may be as shown in the following figure:

.. 在合适的编译选项设置之下，一个函数的栈帧内容可能如下图所示：

.. figure:: StackFrame.png
   :align: center

   The contents of the function stack frame

..    函数栈帧中的内容


It starts and ends at the addresses pointed to by sp(x2) and fp(s0) respectively. According to the address from high to low, there are the following contents, and they are all accessed through ``sp`` plus an offset:

- The ``ra`` register holds the jump address after it returns, and is a caller-saved register;
- The end address ``fp`` of the parent stack frame is a callee-saved register;
- other callee-saved registers ``s1`` ~ ``s11``;
- Local variables used by the function.

Therefore, multiple ``fp`` information on the stack actually saves a complete function call chain, and we can track the relationship of function calls in an appropriate way.

``ra``, ``sp`` and ``fp`` are registers that are closely related to function calls. We use an example to show how the assembly code generated by a real compiler will use these registers. First, we modify ``.cargo/config`` for both the kernel itself and the applications that appear after Chapter 2:


.. 它的开头和结尾分别在 sp(x2) 和 fp(s0) 所指向的地址。按照地址从高到低分别有以下内容，它们都是通过 ``sp`` 加上一个偏移量来访问的：

.. - ``ra`` 寄存器保存其返回之后的跳转地址，是一个调用者保存寄存器；
.. - 父亲栈帧的结束地址 ``fp`` ，是一个被调用者保存寄存器；
.. - 其他被调用者保存寄存器 ``s1`` ~ ``s11`` ；
.. - 函数所使用到的局部变量。

.. 因此，栈上多个 ``fp`` 信息实际上保存了一条完整的函数调用链，通过适当的方式我们可以实现对函数调用关系的跟踪。

.. ``ra`` 、 ``sp`` 和 ``fp`` 是和函数调用紧密相关的寄存器，我们用一个例子来展示真实编译器生成的汇编代码会如何使用这些寄存器。首先，无论对于内核本身还是第二章后出现的应用程序，我们修改 ``.cargo/config`` :

.. code-block::

    // .cargo/config

    [build]
    target = "riscv64gc-unknown-none-elf"

    [target.riscv64gc-unknown-none-elf]
    rustflags = [
        "-Clink-args=-Tsrc/linker.ld", "-Cforce-frame-pointers=yes"
    ]

This can set our default compilation target, adjust the compilation options, set the link script and force the ``fp`` option, so that the ``fp`` related instructions will not be optimized out by the compiler. We can then use the ``rust-objdump`` tool to disassemble the kernel or application executable and find the entry point for a function. We can then see that at the beginning and end of the function, the compiler generates assembly code like this:

.. 这可以设置我们的默认编译目标，同时调整编译选项，设置链接脚本以及强制打开 ``fp`` 选项，这样才会避免 ``fp`` 相关指令被编译器优化掉。随后，我们可以使用 ``rust-objdump`` 工具反汇编内核或者应用程序可执行文件，并找到某个函数的入口。然后，我们能够看到在函数的开场和结尾阶段，编译器会生成类似的汇编代码：

.. code-block:: riscv

    # Prologue
    # Allocate a 64-byte stack frame for the current function
    addi	sp, sp, -64
    # Push ra and fp onto the stack and save
    sd	ra, 56(sp)
    sd	s0, 48(sp)
    # Update fp to the top address of the current function stack frame
    addi	s0, sp, 64


    # function execution
    # If other functions are called in the middle, ra will be modified

    # end
    # restore ra and fp
    ld	ra, 56(sp)
    ld	s0, 48(sp)
    # Deallocate the stack
    addi	sp, sp, 64
    # return , using ret instruction or other likewise approaches
    ret    

So far, we have basically explained how function calls are implemented based on the stack. However, we can ignore these details for the time being, because we only need to configure the stack in the initialization phase, that is, set the stack pointer ``sp`` register, and the compiler will help us automatically complete the code generation of the following function call-related mechanisms. But the trouble is, the value of ``sp`` cannot be set casually. At least we need to ensure that it points to legal physical memory, and it cannot intersect with other code and data segments of the program. It is because during the function call, the stack area content will be modified. How to ensure this?

.. 至此，我们基本上说明了函数调用是如何基于栈来实现的。不过我们可以暂时先忽略掉这些细节，因为我们现在只是需要在初始化阶段完成栈的设置，也就是设置好栈指针 ``sp`` 寄存器，编译器会帮我们自动完成后面的函数调用相关机制的代码生成。麻烦的是， ``sp`` 的值也不能随便设置，至少我们需要保证它指向合法的物理内存，而且不能与程序的其他代码、数据段相交，因为在函数调用的过程中，栈区域里面的内容会被修改。如何保证这一点呢？

.. _jump-practice:

.. 分配并使用启动栈

Allocate and Use Bootstrap Stack
-------------------------------------------

We allocate the boostrap stack space in ``entry.asm`` and set the stack pointer ``sp`` to the location of the top of the stack before control is passed to the Rust entry.

.. 我们在 ``entry.asm`` 中分配启动栈空间，并在控制权被转交给 Rust 入口之前将栈指针 ``sp`` 设置为栈顶的位置。

.. code-block:: asm
    :linenos:

    # os/src/entry.asm
        .section .text.entry
        .globl _start
    _start:
        la sp, boot_stack_top
        call rust_main

        .section .bss.stack
        .globl boot_stack_lower_bound
    boot_stack_lower_bound:
        .space 4096 * 16
        .globl boot_stack_top
    boot_stack_top:

We reserve a space of 4096 * 16 bytes in the memory layout of the kernel in line 11, which is :math:`64\text{KiB}`, which is used as the stack space for the program to run. On the RISC-V architecture, the stack grows from high addresses to low addresses. Therefore, the stack is empty at the beginning, and the top and bottom of the stack are at the same location. We use the higher address symbol ``boot_stack_top`` to identify the location of the stack top. At the same time, we use the lower address symbol ``boot_stack_lower_bound`` to identify the lower limit where the stack can grow, and they are all set as global symbols for use by other object files. As shown below:

.. 我们在第 11 行在内核的内存布局中预留了一块大小为 4096 * 16 字节也就是 :math:`64\text{KiB}` 的空间用作接下来要运行的程序的栈空间。在 RISC-V 架构上，栈是从高地址向低地址增长。因此，最开始的时候栈为空，栈顶和栈底位于相同的位置，我们用更高地址的符号 ``boot_stack_top`` 来标识栈顶的位置。同时，我们用更低地址的符号 ``boot_stack_lower_bound`` 来标识栈能够增长到的下限位置，它们都被设置为全局符号供其他目标文件使用。如下图所示：

.. image:: boot_stack.png
    
From line 8, we can see that we put this space in a section called ``.bss.stack``. And from the linker script ``linker.ld``, you can see ``.bss.stack`` is included into the ``.bss`` section:

.. 第 8 行可以看到我们将这块空间放置在一个名为 ``.bss.stack`` 的段中，在链接脚本 ``linker.ld`` 中可以看到 ``.bss.stack`` 段最终会被汇集到 ``.bss`` 段中：

.. code-block::

    .bss : {
        *(.bss.stack)
        sbss = .;
        *(.bss .bss.*)
        *(.sbss .sbss.*)
    }
    ebss = .;

We mentioned earlier that the ``.bss`` section generally holds data that needs to be initialized to zero. However, the stack does not need to be initialized to zero before use, because when the function is called, we will insert the stack frame to overwrite the existing data. We tried and failed to place it in the global data ``.data`` section, so we decided to put it in the ``.bss`` section. The global symbols ``sbss`` and ``ebss`` respectively point to the start and end addresses of the ``.bss`` section except ``.bss.stack``. So we need to initialize them before using this part of data to zero, this process will be carried out in the next section.

.. 前面我们提到过 ``.bss`` 段一般放置需要被初始化为零的数据。然而栈并不需要在使用前被初始化为零，因为在函数调用的时候我们会插入栈帧覆盖已有的数据。我们尝试将其放置到全局数据 ``.data`` 段中但最后未能成功，因此才决定将其放置到 ``.bss`` 段中。全局符号 ``sbss`` 和 ``ebss`` 分别指向 ``.bss`` 段除 ``.bss.stack`` 以外的起始和终止地址，我们在使用这部分数据之前需要将它们初始化为零，这个过程将在下一节进行。

Back to ``entry.asm``, we can find that two instructions are executed before the control is transferred to the Rust entry, which are located on lines 5 and 6 of ``entry.asm``. In line 5, we set the stack pointer ``sp`` to the previously allocated boostrap stack top address, so that the Rust code can normally allocate and reclaim stack frames on the bootstrap stack when making function calls and returns. In our designed memory layout, the memory used by the boostrap stack will not conflict with other code and data segments of the kernel, and they are physically isolated. However, if the boostrap stack is overflowed (for example, too many function calls appear in the kernel code), then the allocated stack frame may overwrite the code and data of other parts of the kernel, resulting in very strange errors. At present, we can only try to avoid the occurrence of stack overflow. In Chapter 4, with the help of address space abstraction and MMU hardware, we can completely prohibit stack overflow. On line 6, we transfer control to the Rust code by calling the kernel entry point ``rust_main`` written in Rust with the pseudo-instruction ``call``, which is implemented in ``main.rs``:

.. 回到 ``entry.asm`` ，可以发现在控制权转交给 Rust 入口之前会执行两条指令，它们分别位于 ``entry.asm`` 的第 5、6 行。第 5 行我们将栈指针 ``sp`` 设置为先前分配的启动栈栈顶地址，这样 Rust 代码在进行函数调用和返回的时候就可以正常在启动栈上分配和回收栈帧了。在我们设计好的内存布局中，这块启动栈所用的内存并不会和内核的其他代码、数据段产生冲突，它们是从物理上隔离的。然而如果启动栈溢出（比如在内核代码中出现了太多的函数调用），那么分配的栈帧将有可能覆盖内核其他部分的代码、数据从而出现十分诡异的错误。目前我们只能尽量避免栈溢出的情况发生，到了第四章，借助地址空间抽象和 MMU 硬件的帮助，我们可以做到完全禁止栈溢出。第 6 行我们通过伪指令 ``call`` 调用 Rust 编写的内核入口点 ``rust_main`` 将控制权转交给 Rust 代码，该入口点在 ``main.rs`` 中实现：

.. code-block:: rust

    // os/src/main.rs
    #[no_mangle]
    pub fn rust_main() -> ! {
        loop {}
    }

It should be noted here that ``rust_main`` needs to be marked as ``#[no_mangle]`` through a macro to avoid the compiler from confusing its name. Otherwise when linking, ``entry.asm`` will be unable to find external symbol ``rust_main`` provided by ``main.rs``, causing link failure. In the prologue of the ``rust_main`` function, we will allocate a stack frame on the stack for the first time and save the function call context, which is also the lowest stack frame in the whole process of the kernel running.

In the kernel initialization, it is necessary to clear the ``.bss`` section first. This is an important part of the initialization of the kernel. Before using any global variables allocated to the ``.bss`` section, we need to ensure that the ``.bss`` section has been cleared. We do this right at the beginning of ``rust_main``. And since control has been transferred to Rust, we can finally implement this functionality in Rust instead of writing assembly code by hand:


.. 这里需要注意的是需要通过宏将 ``rust_main`` 标记为 ``#[no_mangle]`` 以避免编译器对它的名字进行混淆，不然在链接的时候， ``entry.asm`` 将找不到 ``main.rs`` 提供的外部符号 ``rust_main`` 从而导致链接失败。在 ``rust_main`` 函数的开场白中，我们将第一次在栈上分配栈帧并保存函数调用上下文，它也是内核运行全程中最底层的栈帧。

.. 在内核初始化中，需要先完成对 ``.bss`` 段的清零。这是内核很重要的一部分初始化工作，在使用任何被分配到 ``.bss`` 段的全局变量之前我们需要确保 ``.bss`` 段已被清零。我们就在 ``rust_main`` 的开头完成这一工作，由于控制权已经被转交给 Rust ，我们终于不用手写汇编代码而是可以用 Rust 来实现这一功能了：

.. code-block:: rust
    :linenos:

    // os/src/main.rs
    #[no_mangle]
    pub fn rust_main() -> ! {
        clear_bss();
        loop {}
    }

    fn clear_bss() {
        extern "C" {
            fn sbss();
            fn ebss();
        }
        (sbss as usize..ebss as usize).for_each(|a| {
            unsafe { (a as *mut u8).write_volatile(0) }
        });
    }

In the function ``clear_bss``, we will try to find the global symbols ``sbss`` and ``ebss`` from other places, which are given by the linker script ``linker.ld``. And we indicate the starting and ending address of the ``.bss`` section, in between data to clear. Next, we only need to traverse the address range and clear byte by byte.

.. 在函数 ``clear_bss`` 中，我们会尝试从其他地方找到全局符号 ``sbss`` 和 ``ebss`` ，它们由链接脚本 ``linker.ld`` 给出，并分别指出需要被清零的 ``.bss`` 段的起始和终止地址。接下来我们只需遍历该地址区间并逐字节进行清零即可。

.. note::

    **Rust Tips: External Symbol References**

    extern "C" can refer to an external C function interface (this means that when calling it, it must follow the C language calling convention of the target platform). But here we just refer to the location flag and turn it into usize to get its address. From this, we can know the addresses of both ends of the ``.bss`` segment.

    .. **Rust Tips：外部符号引用**

    .. extern "C" 可以引用一个外部的 C 函数接口（这意味着调用它的时候要遵从目标平台的 C 语言调用规范）。但我们这里只是引用位置标志并将其转成 usize 获取它的地址。由此可以知道 ``.bss`` 段两端的地址。

.. note::

    **Rust Tips: Iterators and Closures**

    Line 13 of the code uses Rust's iterator and closure syntax, which can improve development efficiency in many cases. This can also be rewritten as an equivalent for loop implementation if the reader is interested.

    .. **Rust Tips：迭代器与闭包**

    .. 代码第 13 行用到了 Rust 的迭代器与闭包的语法，它们在很多情况下能够提高开发效率。如读者感兴趣的话也可以将其改写为等价的 for 循环实现。

.. _term-raw-pointer:
.. _term-dereference:

.. warning::

    **Rust Tips：Unsafe**

    On line 14 of the code, we convert an address in the ``.bss`` segment into a **raw pointer**, and modify the value it points to to 0. This is a commonplace operation in C, but in Rust we need to wrap it in an unsafe block. This is because Rust considers **Dereference** (Dereference) for raw pointers to be an unsafe behavior.

    .. 代码第 14 行，我们将 ``.bss`` 段内的一个地址转化为一个 **裸指针** (Raw Pointer)，并将它指向的值修改为 0。这在 C 语言中是一种司空见惯的操作，但在 Rust 中我们需要将他包裹在 unsafe 块中。这是因为，Rust 认为对于裸指针的 **解引用** (Dereference) 是一种 unsafe 行为。

    Compared with C language, Rust implements more semantic constraints to ensure safety (memory safety/type safety/concurrency safety), which is reflected in both compilation and runtime. But at some point, especially when dealing with the underlying hardware, our needs cannot be met within the semantic constraints of Rust. At this time, we need to wrap the behavior beyond the semantic constraints of Rust in an unsafe block and tell the compiler that it does not need to perform a complete constraint check on it. But it is the programmer's responsibility to ensure its safety. When the code does not run normally, we are often the first to check the code in the unsafe block, because it is not protected by the compiler, and the probability of error is greater.

    .. 相比 C 语言，Rust 进行了更多的语义约束来保证安全性（内存安全/类型安全/并发安全），这在编译期和运行期都有所体现。但在某些时候，尤其是与底层硬件打交道的时候，在 Rust 的语义约束之内没法满足我们的需求，这个时候我们就需要将超出了 Rust 语义约束的行为包裹在 unsafe 块中，告知编译器不需要对它进行完整的约束检查，而是由程序员自己负责保证它的安全性。当代码不能正常运行的时候，我们往往也是最先去检查 unsafe 块中的代码，因为它没有受到编译器的保护，出错的概率更大。

    The pointer in C language is equivalent to the raw pointer in Rust. It is omnipotent but too flexible. Programmers often use it carelessly to cause many memory insecurity problems, the most common ones dangling pointers and multiple recycling. The Rust compiler cannot confirm whether programmers are safe to use it, so it is classified as unsafe Rust. In safe Rust, we have references ``&/&mut`` and various smart pointers ``Box<T>/RefCell<T>/Rc<T>`` that can be used. As long as they follow the rules of Rust To use them, the compiler can solve many potential memory insecurity problems at compile time.

    .. C 语言中的指针相当于 Rust 中的裸指针，它无所不能但又太过于灵活，程序员对其不谨慎的使用常常会引起很多内存不安全问题，最常见的如悬垂指针和多次回收的问题，Rust 编译器没法确认程序员对它的使用是否安全，因此将其划到 unsafe Rust 的领域。在 safe Rust 中，我们有引用 ``&/&mut`` 以及各种功能各异的智能指针 ``Box<T>/RefCell<T>/Rc<T>`` 可以使用，只要按照 Rust 的规则来使用它们便可借助编译器在编译期就解决很多潜在的内存不安全问题。

In this section, we introduced the background knowledge of function calls and stacks. By allocating stack space and setting the stack pointer correctly, we enabled function calls in the kernel and successfully transferred control to Rust code. From then on, we can finally use the powerful Rust language to write the various functions of the kernel. In the next section we will proceed to the last step of building the "Trilobite" operating system: that is, the service provided by RustSBI successfully prints ``Hello, world!`` on the screen.

.. 本节我们介绍了函数调用和栈的背景知识，通过分配栈空间并正确设置栈指针在内核中使能了函数调用并成功将控制权转交给 Rust 代码，从此我们终于可以利用强大的 Rust 语言来编写内核的各项功能了。下一节中我们将进行构建“三叶虫”操作系统的最后一个步骤：即基于 RustSBI 提供的服务成功在屏幕上打印 ``Hello, world!`` 。