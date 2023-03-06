.. 练习参考答案

Reference Answers to Exercises
=====================================================

.. toctree::
     :hidden:
     :maxdepth: 4


.. 编程题

Programming Tasks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. `*` Install and configure the experimental environment in your daily operating system environment. Briefly describe the problem/difficulty you encountered and how it was resolved.

2. `*` Write an application program that will generate exceptions in the Linux environment, and briefly explain the processing results of the operating system.

   For example, consider a C program that includes a division by zero:

   .. code-block:: c
      :linenos:

      #include <stdio.h>

      int main() {
          printf("1 / 0 = %d", 1 / 0);
          return 0;
      }
   
   The results of compiling and running in an x86-64 Linux-based environment are as follows:

   .. code-block:: console

      $ gcc divzero
      $ ./divzero
      Floating point exception (core dumped)

   The program received a "floating point exception" and terminated abnormally. Use ``strace`` to see more detailed signal information:

   .. code-block:: console

   $ strace ./divzero
   [... Syscall trace output omitted here]
   --- SIGFPE {si_signo=SIGFPE, si_code=FPE_INTDIV, si_addr=0x401131} ---
   +++ killed by SIGFPE (core dumped) +++


   Use the ``disassemble`` command of gdb to see the abnormal instruction

   .. code-block:: console

      $ gdb divzero
      Reading symbols from divzero...
      (gdb) r
      Starting program: [...]/divzero

      Program received signal SIGFPE, Arithmetic exception.
      0x0000000000401131 in main ()
      (gdb) disassemble
      Dump of assembler code for function main:
         0x0000000000401122 <+0>:     push   %rbp
         0x0000000000401123 <+1>:     mov    %rsp,%rbp
         0x0000000000401126 <+4>:     mov    $0x1,%eax
         0x000000000040112b <+9>:     mov    $0x0,%ecx
         0x0000000000401130 <+14>:    cltd
      => 0x0000000000401131 <+15>:    idiv   %ecx
         0x0000000000401133 <+17>:    mov    %eax,%esi
         0x0000000000401135 <+19>:    mov    $0x402004,%edi
         0x000000000040113a <+24>:    mov    $0x0,%eax
         0x000000000040113f <+29>:    call   0x401030 <printf@plt>
         0x0000000000401144 <+34>:    mov    $0x0,%eax
         0x0000000000401149 <+39>:    pop    %rbp
         0x000000000040114a <+40>:    ret
   
   It can be seen that when the application executes the `idiv` instruction (signed division instruction), a division by zero exception occurs, and it jumps to the operating system for processing. The operating system converts it to a signal `SIGFPE` and uses the signal handling mechanism to handle the exception. What the program should do when it receives `SIGFPE` is to abort, so the operating system terminates it and reports the abort to the shell process.

   It should be noted that exception handling and correspondence with signals are architecture-dependent. For example, division by zero under the RISC-V architecture is not an anomaly, but has a deterministic result. In addition, the corresponding relationship between specific exceptions and signals under different architectures is also different, and even somewhat confusing (for example, here it is clearly an integer division by zero error, but a "floating point number" exception is reported).

.. 1. `*` 在你日常使用的操作系统环境中安装并配置好实验环境。简要说明你碰到的问题/困难和解决方法。

.. 2. `*` 在Linux环境下编写一个会产生异常的应用程序，并简要解释操作系统的处理结果。

..    例如，对于这样一段 C 程序，其中包含一个除以零的操作：

..    .. code-block:: c
..       :linenos:

..       #include <stdio.h>

..       int main() {
..           printf("1 / 0 = %d", 1 / 0);
..           return 0;
..       }

..    在基于 x86-64 Linux 的环境下编译运行结果如下：

..    .. code-block:: console

..       $ gcc divzero
..       $ ./divzero
..       Floating point exception (core dumped)

..    程序接收到了一个“浮点数异常”而异常终止。使用 ``strace`` 可以看到更详细的信号信息：

..    .. code-block:: console

..       $ strace ./divzero
..       [... 此处省略系统调用跟踪输出]
..       --- SIGFPE {si_signo=SIGFPE, si_code=FPE_INTDIV, si_addr=0x401131} ---
..       +++ killed by SIGFPE (core dumped) +++

..    用 gdb 的 ``disassemble`` 命令可以看到发生异常的指令

..    .. code-block:: console

..       $ gdb divzero
..       Reading symbols from divzero...
..       (gdb) r
..       Starting program: [...]/divzero

..       Program received signal SIGFPE, Arithmetic exception.
..       0x0000000000401131 in main ()
..       (gdb) disassemble
..       Dump of assembler code for function main:
..          0x0000000000401122 <+0>:     push   %rbp
..          0x0000000000401123 <+1>:     mov    %rsp,%rbp
..          0x0000000000401126 <+4>:     mov    $0x1,%eax
..          0x000000000040112b <+9>:     mov    $0x0,%ecx
..          0x0000000000401130 <+14>:    cltd
..       => 0x0000000000401131 <+15>:    idiv   %ecx
..          0x0000000000401133 <+17>:    mov    %eax,%esi
..          0x0000000000401135 <+19>:    mov    $0x402004,%edi
..          0x000000000040113a <+24>:    mov    $0x0,%eax
..          0x000000000040113f <+29>:    call   0x401030 <printf@plt>
..          0x0000000000401144 <+34>:    mov    $0x0,%eax
..          0x0000000000401149 <+39>:    pop    %rbp
..          0x000000000040114a <+40>:    ret

..    可以看出，应用程序在执行 `idiv` 指令（有符号除法指令）时发生了除以零异常，跳转至操作系统处理。操作系统把它转换为一个信号 `SIGFPE`，使用信号处理机制处理这个异常。该程序收到 `SIGFPE` 时应发生的行为是异常终止，于是操作系统将其终止，并将异常退出的信息报告给 shell 进程。

..    需要注意的是，异常的处理和与信号的对应是与架构相关的。例如，RISC-V 架构下除以零不是异常，而是有个确定的结果。此外，不同架构下具体异常和信号的对应关系也是不同的，甚至有些混乱（例如，这里明明是整数除以零错误，却报告了“浮点数”异常）。


3. `**` In the Linux environment, write an application A that can print out a string after sleeping for 5 seconds, and store the string content in a file. (based on C or Rust language)

   A sample implementation is as follows (error handling not included)

   .. code-block:: c

      #include <stdio.h>
      #include <unistd.h>

      int main() {
          sleep(5);

          const char* hello_string = "Hello Linux!\n";

          printf(hello_string);

          FILE *output_file = fopen("output.txt", "w");
          fputs(hello_string, output_file);
          fclose(output_file);

          return 0;
      }


   Compile and run, and view the output and files of the program. The results are as follows:

   .. code-block:: console

      $ gcc -o program-a program-a.c
      $ ./program-a
      Hello Linux!        [This line waits for 5s before outputting]
      $ cat output.txt
      Hello Linux!

.. 3. `**`  在Linux环境下编写一个可以睡眠5秒后打印出一个字符串，并把字符串内容存入一个文件中的应用程序A。(基于C或Rust语言)

..    样例实现如下（未包含错误处理）

..    .. code-block:: c

..       #include <stdio.h>
..       #include <unistd.h>

..       int main() {
..           sleep(5);

..           const char* hello_string = "Hello Linux!\n";

..           printf(hello_string);

..           FILE *output_file = fopen("output.txt", "w");
..           fputs(hello_string, output_file);
..           fclose(output_file);

..           return 0;
..       }


..    编译运行，查看程序的输出和文件，结果如下：

..    .. code-block:: console

..       $ gcc -o program-a program-a.c
..       $ ./program-a
..       Hello Linux!        [这一行等待 5s 后才输出]
..       $ cat output.txt
..       Hello Linux!

4. `***` Write an application program B in the Linux environment, and briefly explain that this program can reflect the concurrency, asynchrony, sharing and persistence of the operating system. (based on C or Rust language)

Note: Write in a Linux-like environment and try to debug application A with debugging tools such as GDB, which can set breakpoints, execute single steps, and display variable information.


.. 4. `***`  在Linux环境下编写一个应用程序B，简要说明此程序能够体现操作系统的并发性、异步性、共享性和持久性。(基于C或Rust语言)

.. 注： 在类Linux环境下编写尝试用GDB等调试工具调试应用程序A，能够设置断点，单步执行，显示变量信息。

.. 问答题

Essay Questions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. 1. `*`  什么是操作系统？操作系统的主要目标是什么？

1. `*` What is an operating system? What is the main goal of an operating system?

2. What are the functional similarities and differences between `*` server-oriented operating systems and mobile phone-oriented operating systems?

   * Example of the same part:

     * Need to manage the process
     * Need to manage basic resources such as CPU and memory
     * All need security mechanisms such as isolation protection
     * (the operating systems of servers and mobile phones have a lot in common, and even the most basic parts can be shared, such as using the Linux kernel)

   * Examples of different parts:

     * Supported peripherals are different:

       * Server: high-performance network card (such as optical fiber), storage device (such as HBA), etc.
       * Mobile phone: screen, sound and microphone, baseband (mobile network), WiFi, etc.

     * There are differences in the management of resources that both the server and the mobile phone have:

       * Server: a lot of memory, a lot of CPU cores, may need to support more complex hardware topologies, such as NUMA architecture
       * Mobile phone: The requirement for low power consumption requires the operating system to take power management into account when scheduling, and balance power saving and performance

     * Different applications lead to different security requirements:

       * Server: Thousands of users, a few applications, need various functions to prevent users from exploiting vulnerabilities to manipulate programs on the server
       * Mobile phone: one or a few users, dozens of applications, need privacy protection function

.. 2. `*`  面向服务器的操作系统与面向手机的操作系统在功能上有何异同？

..    * 相同部分举例：

..      * 都需要进行进程的管理
..      * 都需要管理 CPU，内存这些基本资源
..      * 都需要隔离保护等安全机制
..      * （服务器和手机的操作系统共通之处很多，甚至可以共用最基础的部分，比如都用 Linux 内核）

..    * 不同部分举例：

..      * 支持的外设不同：

..        * 服务器：高性能网卡（如光纤）、存储设备（如 HBA）等
..        * 手机：屏幕，音响和话筒，基带（移动网络）、WiFi 等

..      * 服务器和手机都有的资源，管理起来也有区别：

..        * 服务器：内存很多，CPU 核数很多，可能需要支持更复杂的硬件拓扑，如 NUMA 架构
..        * 手机：对低功耗的要求，需要操作系统在调度时考虑到电源管理，平衡省电和性能

..      * 应用不同，导致对安全性要求不同：

..        * 服务器：上千用户，少数几个应用，需要各种阻止用户利用漏洞操控服务器上的程序的功能
..        * 手机：一个或少数几个用户，几十个应用，需要隐私保护功能

3. `*` For current mobile or desktop operating systems, should the operating system include a web browser? Please explain why.

4. `*` What are the core abstractions of the operating system? What are they dealing with?

   * Process/Thread -- CPU time
   * Address space -- memory
   * Execution environment -- the complex environment on the CPU (with interrupt exceptions, etc.), and the functions provided by the operating system (such as system calls)
   * Files and file descriptors -- storage and I/O devices

.. 3. `*`  对于目前的手机或桌面操作系统而言，操作系统是否应该包括网络浏览器？请说明理由。

.. 4. `*`  操作系统的核心抽象有哪些？它们应对的对象是啥？

..    * 进程/线程 -- CPU 时间
..    * 地址空间 -- 内存
..    * 执行环境 -- CPU 上复杂的环境（有中断异常等），和操作系统提供的功能（如系统调用）
..    * 文件和文件描述符 -- 存储和输入输出设备

5. `*` What is the interoperability and data exchange between the operating system and the application program?

   * The way of interoperability:

     * The application program calls the system call to actively let the operating system operate
     * The operating system forcibly suspends the application to perform related operations when an interrupt exception occurs

   * The way of data exchange:

     * System calls pass arguments in (for example) registers according to the ABI
     * Copy data: copy data between the space occupied by the kernel and the space occupied by the user, such as copying the written data from the buffer given by the application when reading and writing files, or copying the read data to the buffer
     * (shared memory space: such as `io_uring`)

.. 5. `*`  操作系统与应用程序之间通过什么来进行互操作和数据交换？

..    * 互操作的方式：

..      * 应用程序调用系统调用主动让操作系统进行操作
..      * 操作系统在中断异常发生时强制暂停应用程序进行相关操作

..    * 数据交换的方式：

..      * 系统调用时根据 ABI 规定在（比如）寄存器中传递参数
..      * 复制数据：在内核占用的空间和用户占用的空间之间互相复制数据，如读写文件的时候从应用程序给出的缓冲区复制写的数据，或者复制读的数据到缓冲区
..      * （共享内存空间：如 `io_uring`）

6. `*` What are the characteristics of the operating system? Please further explain the characteristics of the operating system in combination with the specific operating conditions of the operating system you use daily.

   Take, for example, what might happen when experimenting with an operating system running a Linux-based platform on an ordinary desktop:

   * Virtualization: If the physical memory is relatively insufficient, Linux's swap mechanism will dump the infrequently used memory content to the hard disk, giving priority to ensuring that active processes can access physical memory at high speed.
   * Concurrency: The number of running programs (including `gcc` or `rustc` running concurrently, and other programs) can exceed the number of CPU cores, and the CPU time is scheduled and allocated by the operating system.
   * Asynchrony: Too many compilers running in parallel can affect the performance of the text editor used to write code, because the operating system schedules more time to run the compiler instead of the text editor process.
   * Sharing: Writing code and compiling can share the same file system, read and write on the same hard disk, and the file system related module (VFS) of the operating system will call the file system to implement and arrange these read and write operations with the hard disk drive.
   * Persistence: The files used for operating system experiments are saved in the file system. If the dormitory is powered off at night and the computer is turned off, you can continue working from the state of the files saved yesterday when you wake up tomorrow morning

.. 6. `*`  操作系统的特征是什么？请结合你日常使用的操作系统的具体运行情况来进一步说明操作系统的特征。

..    以在普通的桌面机上运行基于 Linux 的平台上做操作系统实验时可能发生的事情为例：

..    * 虚拟性：如果物理内存相对不足，Linux 的 swap 机制会将不常用的内存内容转存到硬盘上，优先保证活跃的进程可以高速访问物理内存。
..    * 并发性：正在运行的程序数量（包括并发运行的 `gcc` 或 `rustc`，和其它程序）可以超过 CPU 核心数，由操作系统来调度分配使用 CPU 时间。
..    * 异步性：并行运行的编译器太多，可能会影响写代码用的文本编辑器性能，因为操作系统安排了更多时间运行编译器而不是文本编辑器进程。
..    * 共享性：写代码和编译可以共享同一个文件系统，在同一块硬盘上读写，操作系统的文件系统相关模块（VFS）会调用文件系统实现和硬盘驱动安排这些读写操作。
..    * 持久性：操作系统实验用的文件保存在文件系统中，晚上宿舍断电关机，明天早上起来可以从昨天保存的文件的状态继续工作。

7. `*` Please explain the similarities and differences between the execution environment based on C language application and the execution environment based on Java language application.

.. 7. `*`  请说明基于C语言应用的执行环境与基于Java语言应用的执行环境的异同。

8. `**` Please briefly list the functions of the system calls of the operating system, and briefly explain the general interface and meaning of the Linux system calls related to program execution, memory allocation, and file reading and writing.

   * The role of the system call:

     * Separate the operating system implementation from the user program calling method, as an abstraction layer to facilitate development and use
     * Server as a unified application program requests the operating system services, which is convenient for security inspection and auditing, when user programs cannot directly access the address space of other user programs and the address space of the operating system, 
     * Allows user programs to have limited access to computer resources and operating system objects that they cannot directly access

   * In Linux:

     * `clone` (`fork` used in the past) creates a process; `execve` passes in the file path, command line parameters, and environment variables, and loads a new program to replace the current process.
     * `brk` modifies or obtains the current position of the top of the heap, increases the allocation space of the top address of the heap, and reduces the release space of the bottom address of the heap; `mmap` is used to map the address space, and can also allocate physical memory (`MAP_ANONYMOUS`),` munmap` releases the corresponding address space and the corresponding memory.
     * File reading and writing are related to `openat` (`open` used in the past) to open the file, `read` to read the file, `write` to write the file, etc.

.. 8. `**`  请简要列举操作系统的系统调用的作用，以及简要说明与程序执行、内存分配、文件读写相关的Linux系统调用的大致接口和含义。

..    * 系统调用的作用：

..      * 将操作系统实现和用户程序调用方式分开，作为一个抽象层方便开发和使用
..      * 在让用户程序没法直接访问别的用户程序的地址空间和操作系统地址空间的情况下，作为一个统一的应用程序请求操作系统服务的入口，方便安全检查和审计。
..      * 让用户程序能有限访问其不能直接访问的计算机资源和操作系统对象

..    * 在 Linux 中：

..      * `clone`（曾用的有 `fork`）创建进程；`execve` 传入文件路径、命令行参数、环境变量，加载新的程序替换当前进程。
..      * `brk` 修改或获取当前的堆顶的位置，增加堆顶地址分配空间，减小堆底地址释放空间；`mmap` 用于映射地址空间，也可以分配物理内存（`MAP_ANONYMOUS`），`munmap` 释放对应的地址空间和对应的内存。
..      * 文件读写相关有 `openat`（曾用的有 `open`）打开文件，`read` 读文件，`write` 写文件等。

9. `**` Take the application A you wrote that can sleep for 5 seconds and print out a string as an example, what is control flow? What is exception control flow? What are processes, address spaces and files? And briefly describe how the operating system supports this application to complete its work and end.

   Operating system-related concepts embodied in application A:

   * Control flow: The ``main`` function is executed line by line, which is a sequential control flow; the standard library function called is the control flow of the function call.
   * Exceptional control flow: During ``sleep``, the system call `clock_nanosleep` is executed. At this time, the control flow jumps out of the program and enters the system call implementation code of the operating system, and then the operating system puts application A into a sleep state , and turn to run other processes. It reflects the Exceptional control flow. A similar situation occurs when executing other system calls.
   * Process: The entire application A is running in a new process, and the shell that started it.
   * Address space: The address of the string ``hello_string`` is valid in the address space of the process itself, and has nothing to do with other processes.
   * File: Application A opens the file named ``output.txt`` and writes a string to it. In addition, ``printf`` writes to standard output, which at this point is a file corresponding to the current terminal.

   The operating system supports the process that application A runs:

   * The operating system loads the program and the C language standard library into memory
   * The operating system sets up some initial virtual memory configuration and some data, then switches to the user state and jumps to the entry point of the program to start running.
   * The application program calls the C language standard library, and the library makes a system call, enters the kernel, and the kernel processes the related system calls and then resumes the application program, specifically:

     * For ``sleep``, the operating system switches to other tasks within 5 seconds, does not run application A, and continues to run after 5 seconds
     * For files, the operating system finds the corresponding file to write in the file system, or finds the corresponding terminal and sends the written content to it

   * Finally, the C language standard library will call the `exit_group` system call to exit the process. After the operating system receives this system call, it will not return to the application program and release the resources related to the process.

.. 9. `**`  以你编写的可以睡眠5秒后打印出一个字符串的应用程序A为例，说明什么是控制流？什么是异常控制流？什么是进程、地址空间和文件？并简要描述操作系统是如何支持这个应用程序完成其工作并结束的。

..    应用程序A中体现的操作系统相关概念：

..    * 控制流： ``main`` 函数内一行一行往下执行，是一个顺序控制流；其中调用的标准库函数，是函数调用的控制流。
..    * 异常控制流： 在 ``sleep`` 的时候，执行系统调用 `clock_nanosleep`，此时控制流跳出了该程序，进入了操作系统的系统调用实现代码，之后操作系统将应用程序A进入睡眠状态，转而运行别的进程，这里体现了异常控制流。在执行其它系统调用的时候也会有类似的情况。
..    * 进程：整个应用程序A是在一个新的进程中运行的，与启动它的 shell。
..    * 地址空间：字符串 ``hello_string`` 所在的地址，是在这个进程自己的地址空间内有效的，和别的进程无关。
..    * 文件：应用程序A打开文件名为 ``output.txt`` 的文件，向其中写入了一个字符串。此外， ``printf`` 是向标准输出写入，标准输出在此时是一个对应当前终端的文件。

..    操作系统支持应用程序A运行的流程：

..    * 操作系统加载程序和 C 语言标准库到内存中
..    * 操作系统设置一些初始的虚拟内存配置和一些数据，然后切换到用户态跳转到程序的入口点开始运行。
..    * 应用程序调用 C 语言标准库，库再进行系统调用，进入内核，内核处理相关的系统调用然后恢复应用程序运行，具体来说：

..      * 对于 ``sleep`` 来说，操作系统在 5 秒时间内切换到别的任务，不运行应用程序A，5 秒过后再继续运行
..      * 对于文件来说，操作系统在文件系统内找到对应的文件写入，或者找到对应的终端将写入的内容发送过去

..    * 最后 C 语言标准库会调用 `exit_group` 系统调用退出进程。操作系统接收这个系统调用后，不再回到应用程序，释放该进程相关资源。

10. `*` Please briefly describe the characteristics of OS supporting a single application, batch OS, multiprogramming OS, and time-sharing OS.

.. 10. `*`  请简要描述支持单个应用的OS、批处理OS、多道程序OS、分时共享OS的特点。
