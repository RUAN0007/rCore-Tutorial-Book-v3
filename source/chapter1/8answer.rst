.. 练习参考答案

Exercise Reference Answers
=====================================================

.. toctree::
   :hidden:
   :maxdepth: 4


.. 课后练习

Homework
-------------------------------

.. 编程题

Programming Tasks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. 1. `*` 实现一个linux应用程序A，显示当前目录下的文件名。（用C或Rust编程）

..    References:

1. `*` Implement a linux application program A, which displays the file names in the current directory. (code in C or Rust)

   Reference implementation:


   .. code-block:: c

      #include <dirent.h>
      #include <stdio.h>

      int main() {
          DIR *dir = opendir(".");

          struct dirent *entry;

          while ((entry = readdir(dir))) {
              printf("%s\n", entry->d_name);
          }

          return 0;
      }

   Possible Output:

   .. code-block:: console

      $ ./ls
      .
      ..
      .git
      .dockerignore
      Dockerfile
      LICENSE
      Makefile
      [...]


2. `***` Implement a linux application program B that can print out the call stacktrace. (programming in C or Rust)

    Taking the C language program compiled with GCC as an example, when the compilation parameter ``-fno-omit-frame-pointer`` is used, the stack frame pointer ``fp`` will be saved.

    Two values ​​are stored at the negative offset of the stack location pointed to by ``fp``:

    * ``-8(fp)`` is saved by ``ra``
    * ``-16(fp)`` is the last ``fp`` saved

    So we can start from the value of the current ``fp`` register like a linked list, find the previous ``fp`` each time, and restore our call stack frame by frame:

    .. 以使用 GCC 编译的 C 语言程序为例，使用编译参数 ``-fno-omit-frame-pointer`` 的情况下，会保存栈帧指针 ``fp`` 。

    .. ``fp`` 指向的栈位置的负偏移量处保存了两个值：

    .. * ``-8(fp)`` 是保存的 ``ra``
    .. * ``-16(fp)`` 是保存的上一个 ``fp``

    .. 因此我们可以像链表一样，从当前的 ``fp`` 寄存器的值开始，每次找到上一个 ``fp`` ，逐帧恢复我们的调用栈：

    .. code-block:: c

       #include <inttypes.h>
       #include <stdint.h>
       #include <stdio.h>

       // Compile with -fno-omit-frame-pointer
       void print_stack_trace_fp_chain() {
           printf("=== Stack trace from fp chain ===\n");

           uintptr_t *fp;
           asm("mv %0, fp" : "=r"(fp) : : );

           // When should this stop?
           while (fp) {
               printf("Return address: 0x%016" PRIxPTR "\n", fp[-1]);
               printf("Old stack pointer: 0x%016" PRIxPTR "\n", fp[-2]);
               printf("\n");

               fp = (uintptr_t *) fp[-2];
           }
           printf("=== End ===\n\n");
       }

    But there will be a problem here. Because our standard library does not save the stack frame pointer, so when we find the call stack to the standard library, it will break our assumption of the stack frame format and an exception will occur.

    We can also not make assumptions about how the stack frame is saved, but explicitly let the compiler tell us how the call stack at each instruction is restored. Adding ``-funwind-tables`` when compiling will enable this function, and store the call stack recovery information in the executable file.

    There is a library called `libunwind <https://www.nongnu.org/libunwind>`_ that can help us read these information and generate call stack information, and it can correctly find that some stack frames do not know how to recover and avoid exceptions quit.

    After installing libunwind correctly, we can also generate call stack information in this way:

    .. 但是这里会遇到一个问题，因为我们的标准库并没有保存栈帧指针，所以找到调用栈到标准的库时候会打破我们对栈帧格式的假设，出现异常。

    .. 我们也可以不做关于栈帧保存方式的假设，而是明确让编译器告诉我们每个指令处的调用栈如何恢复。在编译的时候加入 ``-funwind-tables`` 会开启这个功能，将调用栈恢复的信息存入可执行文件中。

    .. 有一个叫做 `libunwind <https://www.nongnu.org/libunwind>`_ 的库可以帮我们读取这些信息生成调用栈信息，而且它可以正确发现某些栈帧不知道怎么恢复，避免异常退出。

    .. 正确安装 libunwind 之后，我们也可以用这样的方式生成调用栈信息：

    .. code-block:: c

       #include <inttypes.h>
       #include <stdint.h>
       #include <stdio.h>

       #define UNW_LOCAL_ONLY
       #include <libunwind.h>

       // Compile with -funwind-tables -lunwind
       void print_stack_trace_libunwind() {
           printf("=== Stack trace from libunwind ===\n");

           unw_cursor_t cursor; unw_context_t uc;
           unw_word_t pc, sp;

           unw_getcontext(&uc);
           unw_init_local(&cursor, &uc);

           while (unw_step(&cursor) > 0) {
               unw_get_reg(&cursor, UNW_REG_IP, &pc);
               unw_get_reg(&cursor, UNW_REG_SP, &sp);

               printf("Program counter: 0x%016" PRIxPTR "\n", (uintptr_t) pc);
               printf("Stack pointer: 0x%016" PRIxPTR "\n", (uintptr_t) sp);
               printf("\n");
           }
           printf("=== End ===\n\n");
       }


3. `**` Implement an application C based on rcore/ucore tutorial, and use the sleep system call to sleep for 5 seconds (in rcore/ucore tutorial v3: Branch ch1)

Note: Try to use GDB (or other debugging tools) and output strings to debug the above programs, etc. Or, you can set breakpoints, execute single steps while displaying variable values, in order to understand the correspondence between assembly codes and source programs.

.. 3. `**` 实现一个基于rcore/ucore tutorial的应用程序C，用sleep系统调用睡眠5秒（in rcore/ucore tutorial v3: Branch ch1）

.. 注： 尝试用GDB等调试工具和输出字符串的等方式来调试上述程序，能设置断点，单步执行和显示变量，理解汇编代码和源程序之间的对应关系。


.. 问答题

Essay Questions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. `*` What computer resources will an application use during its execution?
2. `*` Please analyze and give the address space range of the code segment/data segment/heap/stack of application A with relevant software tools. 
3. `*` Please analyze and give the address space range of the code segment/data segment/heap/stack of the application program C.
4. `*` Please combine the knowledge of the compiler and the application program B to explain how the application program B establishes the call stacktrace.
5. `*` Please briefly describe the similarities and differences between the application and the operating system.
6. `**` By referring to how QEMU emulates RSIV-v and QEMU source codes, explain where are the instructions after the RISC-V hardware is powered on? What functions have been completed?

    In the QEMU source code [#qemu_bootrom]_, you can find several instructions just executed when "power on", as follows:

   .. code-block:: c

      uint32_t reset_vec[10] = {
          0x00000297,                   /* 1:  auipc  t0, %pcrel_hi(fw_dyn) */
          0x02828613,                   /*     addi   a2, t0, %pcrel_lo(1b) */
          0xf1402573,                   /*     csrr   a0, mhartid  */
      #if defined(TARGET_RISCV32)
          0x0202a583,                   /*     lw     a1, 32(t0) */
          0x0182a283,                   /*     lw     t0, 24(t0) */
      #elif defined(TARGET_RISCV64)
          0x0202b583,                   /*     ld     a1, 32(t0) */
          0x0182b283,                   /*     ld     t0, 24(t0) */
      #endif
          0x00028067,                   /*     jr     t0 */
          start_addr,                   /* start: .dword */
          start_addr_hi32,
          fdt_load_addr,                /* fdt_laddr: .dword */
          0x00000000,
                                        /* fw_dyn: */
      };


    The completed job is:

   - Read current Hart ID CSR ``mhartid`` Write register ``a0``
   - (We haven't used it yet: write the address of FDT (Flatten device tree) in physical memory to ``a1``)
   - Jump to ``start_addr``, which in our experiments is the address of RustSBI

..    完成的工作是：

..    - 读取当前的 Hart ID CSR ``mhartid`` 写入寄存器 ``a0``
..    - （我们还没有用到：将 FDT (Flatten device tree) 在物理内存中的地址写入 ``a1``）
..    - 跳转到 ``start_addr`` ，在我们实验中是 RustSBI 的地址

7. `*` What is the meaning and function of SBI in RISC-V?
8. `**` What agreements need to be made between the operating system and the compiler in order for the application to execute on the computer?
9. `**` Please briefly explain the execution process from the power-on of the RISC-V computer simulated by QEMU to the execution of the first instruction of the application program.
10. `**` Why do application programmers need not to create stack space and specify address space when writing applications?
11. `***` The code generated by many modern compilers no longer strictly saves/restores the stack frame pointer by default. In this case, as long as the compiler provides enough information, we can also restore the call stack.

    * First, our current ``pc`` is at the beginning of the ``flip`` function, which is the function we are running. The address returned to the caller is in the ``ra`` register, which is ``0x10742``. Since we haven't started manipulating the stack pointer, the ``sp`` at the call site is the same as ours, which is ``0x40007f1310``.
    * ``0x10742`` is inside the ``flap`` function. According to the beginning of the ``flap`` function, the stack frame size of this function is 16 bytes, so the stack pointer at the caller should be ``sp + 16 = 0x40007f1320``. The return address of the caller calling ``flap`` is saved on the stack ``8(sp)``, which can be read as ``0x10750``, and it is still in the ``flap`` function.
    * By analogy, as long as the function code corresponding to the known address can be understood, the recovery operation can be completed.

    .. * 首先，我们当前的 ``pc`` 在 ``flip`` 函数的开头，这是我们正在运行的函数。返回给调用者处的地址在 ``ra`` 寄存器里，是 ``0x10742`` 。因为我们还没有开始操作栈指针，所以调用处的 ``sp`` 与我们相同，都是 ``0x40007f1310`` 。
    .. * ``0x10742`` 在 ``flap`` 函数内。根据 ``flap`` 函数的开头可知，这个函数的栈帧大小是 16 个字节，所以调用者处的栈指针应该是 ``sp + 16 = 0x40007f1320``。调用 ``flap`` 的调用者返回地址保存在栈上 ``8(sp)`` ，可以读出来是 ``0x10750`` ，还在 ``flap`` 函数内。
    .. * 依次类推，只要能理解已知地址对应的函数代码，就可以完成恢复操作。


Experimental Exercises
-------------------------------

Question-and-Answer Assignments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Please learn how to use the gdb debugging tool (this is very important for subsequent debugging), and simply trace the simple process from powering on the machine to jumping to 0x80200000 through gdb. Only important jumps need to be described, and only the situation on qemu needs to be described.

.. 问答作业
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. 1. 请学习 gdb 调试工具的使用(这对后续调试很重要)，并通过 gdb 简单跟踪从机器加电到跳转到 0x80200000 的简单过程。只需要描述重要的跳转即可，只需要描述在 qemu 上的情况。

.. [#qemu_bootrom] https://github.com/qemu/qemu/blob/0ebf76aae58324b8f7bf6af798696687f5f4c2a9/hw/riscv/boot.c#L300