.. 什么是操作系统

What is an Operating System
================================================

.. toctree::
   :hidden:
   :maxdepth: 5

.. 站在一万米的代码空间维度看

Standing on the 10,000-meter altitude of code space and see from above
------------------------------------------------------------------------------

.. chyyuu What is an Operating System? A historical investigation (1954–1964) Maarten Bullynck

An operating system (OS) is software that helps users and applications use and manage a computer's resources. An operating system may not be visible to the end user, but it controls a variety of computer systems such as embedded devices, more general systems such as smartphones, desktop computers, and servers, and supercomputers.

.. 一个操作系统（OS）是一个软件，它帮助用户和应用程序使用和管理计算机的资源。操作系统可能对最终用户不可见，但它控制着嵌入式设备、更通用的系统（如智能手机、台式计算机和服务器）以及巨型机等各种计算机系统。

Today, it's hard to imagine using a computer without an operating system, which shapes and builds the way we interact with accessing a computer and its peripherals. When the first electronic computers were developed after World War II, there was no such software as an operating system. The first attempt at an operating system of some kind came about a decade after the computer was born. It took another decade before the operating system became widely accepted.

.. 今天，我们很难想象在没有操作系统的情况下使用计算机，它塑造和构建了我们访问计算机及其外围设备的交互方式。当第一批电子计算机在第二次世界大战后被开发出来时，还没有操作系统这种软件。在计算机诞生十年后，才首次出现某种操作系统的尝试。又过了十年，操作系统才被大家广泛接受。

Our discussion will focus on general-purpose operating systems because they require a superset of the technologies required for embedded systems, providing a more comprehensive coverage of operating system principles, concepts, and techniques. The current general-purpose operating system is a complex system software. For example, the source code of Linux operating system has tens of millions of lines in C language. In the early stage of learning the operating system, if you analyze and understand such a large-scale software, you will have to pay a heafy price, so we simplify it and only discuss the most basic functions.

.. 我们的讨论将集中在通用操作系统上，因为它们需要的技术是嵌入式系统所需技术的超集，对操作系统原理、概念和技术的覆盖更加全面。现在的通用操作系统是一个复杂的系统软件，比如 Linux 操作系统达到了千万行的 C 源码量级。在学习操作系统的初期，如果去分析了解这样大规模的软件，要付出巨大的代价，因此我们对其进行简化，只讨论最基本的功能。

.. 系统软件

System Software
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

System software is software that provides basic functions for the computer system and is used within the scope of the computer system, and its role may involve the entire computer system. System software includes operating system kernel, driver, tool software, user interface, software library, etc. The operating system kernel is the core part of the system software, responsible for controlling the computer's hardware resources and providing services for users and applications. Drivers are software used by the operating system to control hardware devices, such as graphics card drivers, sound card drivers, and printer drivers. Typically, drivers are part of the operating system kernel.

.. 系统软件是为计算机系统提供基本功能，并在计算机系统范围内使用的软件，其作用可涉及到整个计算机系统。系统软件包括操作系统内核、驱动程序、工具软件、用户界面、软件库等。操作系统内核是系统软件的核心部分，负责控制计算机的硬件资源并为用户和应用程序提供服务。驱动程序是操作系统用于控制硬件设备的软件，如显卡驱动、声卡驱动和打印机驱动等。一般情况下，驱动程序是操作系统内核的一部分。

Tool software is software provided by the operating system for maintaining, debugging, and optimizing computer systems, such as disk defragmentation tools, system information tools, and virus killing tools. The user interface can be a graphical user interface (GUI) or a command line interface (CLI). A GUI is a common user interface for an operating system that uses graphical elements such as icons, menus, and buttons to help users use the operating system. Typically, a graphical user interface provides a desktop environment with windows that can be opened and closed in which the user can run applications and perform other operations. A GUI is a common user interface for an operating system that uses graphical elements such as icons, menus, and buttons to help users use the operating system. Typically, a graphical user interface provides a desktop environment with windows that can be opened and closed in which the user can run applications and perform other operations.

.. 工具软件是操作系统提供的用于维护、调试和优化计算机系统的软件，如磁盘碎片整理工具、系统信息工具和病毒查杀工具等。用户界面可以是图形用户界面 (GUI) 或命令行界面 (CLI)。图形用户界面是操作系统的一种常见用户界面，它使用图形元素（如图标、菜单和按钮）来帮助用户使用操作系统。通常，图形用户界面提供了一个桌面环境，其中包含可以打开和关闭的窗口，用户可以在其中运行应用程序和执行其他操作。图形用户界面是操作系统的一种常见用户界面，它使用图形元素（如图标、菜单和按钮）来帮助用户使用操作系统。通常，图形用户界面提供了一个桌面环境，其中包含可以打开和关闭的窗口，用户可以在其中运行应用程序和执行其他操作。

.. chyyuu    
   如果这样来看，编辑类软件，如 Vi、Emacs、MS Word等，只涉及到对文本文件的编辑，它们就不能算是系统软件。


The C language standard library libc (similar to the Rust standard library, etc.) provides a system call interface for interacting with the OS. Its functions cover the entire computer system and will be accessed and called by many different software.

From this point of view, the operating system is a kind of system software.

.. C 语言标准库 libc（类似的有 Rust 标准库 等）提供了与 OS 交互的系统调用接口，其功能覆盖了整个计算机系统，会被许多不同的软件访问和调用。

.. 从这个角度来看，操作系统是一种系统软件。

.. 执行环境

Runtime Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From the perspective of applications, we can find that common applications are actually running in an :ref:`Execution Environment (Execution Environment) <exec-env>` wrapped by hardware, operating system kernel, runtime library, graphical interface support library, etc., as shown in the figure below. The execution environment provides runtime services required to run application software, including memory management, file system access, network connections, etc. Most of these services are provided by the operating system. The application program only needs to request various services or functions provided by the execution environment according to the Application Binary Interface (ABI, Application Binary Interface) agreed with the system software, so as to complete the application program's own functions. Based on this observation, we can simplify the definition of an operating system to: **A software execution environment for applications**. This general description can be applied to different historical periods of operating system development. From this point of view, the operating system can include system software such as runtime libraries and graphical interface support libraries. In :ref:`subsequent section "Execution Environment" <term-exec-env-define>`, the meaning of execution environment will be further elaborated.

.. 站在应用程序的角度来看，我们可以发现常见的应用程序其实是运行在由硬件、操作系统内核、运行时库、图形界面支持库等所包起来的一个 :ref:`执行环境 (Execution Environment) <exec-env>` 中，如下图所示。执行环境提供了运行应用软件所需的运行时服务，包括内存管理、文件系统访问、网络连接等，这些服务大部分是由操作系统来提供的。应用程序只需根据与系统软件约定好的应用程序二进制接口 (ABI, Application Binary Interface) 来请求执行环境提供的各种服务或功能，从而完成应用程序自己的功能。基于这样的观察，我们可以把操作系统的定义简化为： **应用程序的软件执行环境** 。这种概括性描述可以适用于操作系统发展的不同历史时期。从这个角度出发，操作系统可以包括运行时库、图形界面支持库等系统软件。在 :ref:`后续小节“执行环境” <term-exec-env-define>` 中会对执行环境相关含义进行进一步的阐述。

.. image:: EE.png
   :align: center
   :name: exec-env



.. 操作系统的定义与组成

Definition and Constitution of Operating System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If we stand at an altitude of 10,000 meters and see from above :ref:`operating system<computer-hw-sw>`, we can find that the system software of the operating system mainly fulfill two tasks: one is to manage and control the computer hardware and various peripherals underneath, and the second is to upwards manage application software and provide various services. We can further define it as: operating system is a kind of system software whose main function is to manage hardware resources such as CPU, memory and various peripherals underneath, and form a software execution environment to manage and serve application software upwards. Such a description also conforms to the definition of operating system in most operating system textbooks. In order to complete these tasks, the operating system needs to know how to deal with hardware and how to provide services to application software. There are a series of operating system-related theories, abstractions, designs, etc. to support how to do and do these two things well.
s

.. 如果我们站在一万米的高空来看 :ref:`操作系统 <computer-hw-sw>` ，可以发现操作系统这个系统软件干的事主要有两件：一是向下管理并控制计算机硬件和各种外设，二是向上管理应用软件并提供各种服务。我们可对其进一步定义为：操作系统是一种系统软件，主要功能是向下管理CPU、内存和各种外设等硬件资源，并形成软件执行环境来向上管理和服务应用软件。这样的描述也符合大多数操作系统教材上对操作系统的定义。为了完成这些工作，操作系统需要知道如何与硬件打交道，如何给应用软件提供服务。这就有一系列与操作系统相关的理论、抽象、设计等来支持如何做和做得好这两件事情。

.. image:: computer-hw-sw.png
   :align: center
   :scale: 50 %
   :name: computer-hw-sw

.. 如果看看我们的身边， Android 应用运行在 ARM 处理器上的 Android 操作系统执行环境中；微软的 Office 应用运行在 x86-64 处理器上的 Windows 操作系统执行环境中；Web Server应用运行在 x86-64 处理器上的 Linux 操作系统执行环境中；Web app 应用运行在 x86-64 或 ARM 处理器上的 Chrome OS 操作系统执行环境中。而在一些嵌入式环境中，操作系统以运行时库的形式与应用程序紧密结合在一起，形成一个可在嵌入式硬件上执行的嵌入式应用。所以，在不同的应用场景下，操作系统的边界也是不同的，我们可以把运行时库、图形界面支持库等这些可支持不同应用的系统软件 (System Software) 也看成是操作系统的一部分。

If you look around us, Android applications run on the Android operating system execution environment on the ARM processor; Microsoft Office applications run on the Windows operating system execution environment on the x86-64 processor; Web Server applications run on the x86- Linux operating system execution environment on 64 processors; Web app application runs on Chrome OS operating system execution environment on x86-64 or ARM processors. In some embedded environments, the operating system closely works with the application program in the form of a runtime library to form an embedded application that can be executed on the embedded hardware. Therefore, in different application scenarios, the boundaries of the operating system are also different. We can regard runtime libraries, graphical interface support libraries and other system softwares that can support different applications as part of the operating system.


What are the components of the operating system? In general, the main components of an operating system include:

1. Operating system kernel: The core part of the operating system, responsible for controlling the computer's hardware resources and providing services for users and applications.
2. System tools and software libraries: software that provides basic functions for the operating system, including tool software and system software libraries.
3. User interface: It is the shell of the operating system and the way for the user to interact with the operating system. The user interface includes a graphical user interface (GUI) and a command line interface (CLI).

.. 那操作系统的组成部分包含哪些内容呢？在一般情况下，操作系统的主要组成包括：

.. 1. 操作系统内核：操作系统的核心部分，负责控制计算机的硬件资源并为用户和应用程序提供服务。
.. 2. 系统工具和软件库：为操作系统提供基本功能的软件，包括工具软件和系统软件库等。
.. 3. 用户接口：是操作系统的外壳，是用户与操作系统交互的方式。用户接口包括图形用户界面（GUI）和命令行界面（CLI）等。

The object that this book focuses on is the operating system kernel, and its main components include:

1. Process/thread management: The kernel is responsible for managing processes or threads in the system, creating, destroying, scheduling and switching processes or threads.
2. Memory management: The kernel is responsible for managing system memory, allocating and reclaiming memory space, and ensuring memory isolation between processes.
3. File system: The kernel provides a file system interface, is responsible for managing files and directories on storage devices, and allows applications to access the file system.
4. Network communication: The kernel provides a network communication interface, which is responsible for managing network connections and allowing applications to communicate over the network.
5. Device driver: The kernel provides a device driver interface, which is responsible for managing hardware devices and allowing applications and other parts of the kernel to access devices.
6. Synchronized mutual exclusion: The kernel is responsible for coordinating access to shared resources among multiple processes or threads. The synchronization function is mainly used to solve the cooperation problem between processes or threads, and the mutual exclusion function is mainly used to solve the competition problem between processes or threads.
7. System call interface: The kernel provides an entry point for applications to access system services, and applications use the system call interface to call services provided by the operating system, such as file systems, network communications, and process management.


.. 而本书重点讲述的对象是操作系统内核，它的主要组成部分包括：

.. 1. 进程/线程管理：内核负责管理系统中的进程或线程，创建、销毁、调度和切换进程或线程。
.. 2. 内存管理：内核负责管理系统的内存，分配和回收内存空间，并保证进程之间的内存隔离。
.. 3. 文件系统：内核提供文件系统接口，负责管理存储设备上的文件和目录，并允许应用访问文件系统。
.. 4. 网络通信：内核提供网络通信接口，负责管理网络连接并允许应用进行网络通信。
.. 5. 设备驱动：内核提供设备驱动接口，负责管理硬件设备并允许应用和内核其他部分访问设备。
.. 6. 同步互斥：内核负责协调多个进程或线程之间对共享资源的访问。同步功能主要用于解决进程或线程之间的协作问题，互斥功能主要用于解决进程或线程之间的竞争问题。
.. 7. 系统调用接口：内核提供给应用程序访问系统服务的入口，应用程序通过系统调用接口调用操作系统提供的服务，如文件系统、网络通信、进程管理等。

The following figure is a schematic diagram of the composition of a typical UNIX operating system: 

.. 下图就是一个典型的UNIX操作系统的组成示意图：

.. image:: ../../os-lectures/lec1/figs/ucorearch.png
   :align: center
   :scale: 100 %
   :name: unix-arch



.. 站在计算机发展的时间尺度看

See from the historical lens of computer evolutions
--------------------------------------------------------------

Although electronic computers only emerge about seventy years ago, computer technology and operating systems have undergone tremendous changes. From the perspective of the short history of computer development, the operating system has also gradually developed from scratch about ten years after the birth of the computer. The operating system mainly completes the essential functions of hardware control and providing services for application programs. Its history is inseparable from the development history of computers. The connotation and functions of the operating system are constantly changing and improving along with the historical development. In the eyes of the public at the beginning of the 21st century, the operating system is the software system on their mobile phones/terminals, including a collection of various applications, of which graphical interfaces and web browsers are important components.

.. 虽然电子计算机的出现距今才仅仅七十年左右，但计算机技术和操作系统已经发生了巨大的变化。从计算机发展的短暂的历史角度看，操作系统也是从计算机诞生大约十年后，从无到有地逐步发展起来的。操作系统主要完成硬件控制和为应用程序提供服务这些必不可少的功能，它的历史与计算机的发展史密不可分。操作系统的内涵和功能随着历史的发展也在一直变化、改进中。如今在二十一世纪初期的大众眼中，操作系统就是他们的手机/终端上的软件系统，包括各种应用程序集合，图形界面和网络浏览器是其中重要的组成部分。

In fact, the connotation and extension of the operating system have been changing with the development of history, and there is no clear definition like "1+1=2". Referring to the evolutionary history of life on earth, we also give a brief overview of the evolutionary history of the operating system, from which we can see what the operating system contains and what characteristics it has in various time periods. But no matter how the internal implementations and specific goals of the operating system change, it remains unchanged of its core that manages computer hardware and provides services to applications. 

.. 其实，操作系统的内涵和外延随着历史的发展也一直在变化，并没有类似于“1+1=2”这样的明确定义。参考地球生物的进化史，我们也给操作系统的进化历史做一个简单的概述，从中可以看到操作系统在各个时间段上包含什么，具有什么样的特征。但无论操作系统的内在实现和具体目标如何变化，其管理计算机硬件，给应用提供服务的核心定位没有变化。

Cambrian Explosion Era [#Cambrian]_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


.. chyyuu https://en.wikipedia.org/wiki/Timeline_of_operating_systems  https://en.wikipedia.org/wiki/History_of_operating_systems https://computerhistory.org/ https://en.wikipedia.org/wiki/Comparison_of_operating_systems

When electronic computers appeared in 1946, only human operators managed and operated machines, and there was no such thing as an operating system. The hard work of starting up, flipping switches, loading cards/tapes, etc. is done by computer operators or users themselves. The operator/user goes to the computer with a card (Punch Card) or punched paper tape on which programs and data are recorded. After loading the card/tape, start the card/tape reader, read the program and data into the computer memory, the computer starts to work, and outputs the results to the card/tape or display, and finally the program stops .

.. 在 1946 年出现电子计算机的时候，只有人类操作员（Operator）来管理和操作机器，还没有操作系统（Operating System）这种事物 。启动，扳开关，装卡片/纸带等比较辛苦的工作都是计算机操作员或者用户自己完成。操作员/用户带着记录有程序和数据的卡片 (Punch Card) 或打孔纸带去操作计算机。装好卡片/纸带后，启动卡片/纸带阅读器，把程序和数据读入计算机内存中之后，计算机就开始工作，并把结果也输出到卡片/纸带或显示屏上，最后程序停止。

With the advancement of programming language and compilation technology, programmers are encouraged to develop translation symbol programs (i.e. compilers) to automatically convert codes into machine codes, replacing the previous inefficient manual machine coding methods and improving the efficiency of program development.  But the efficiency of program execution is still very low. And as computers and I/O devices become more powerful, the time it takes for a program to run decreases, while it takes longer to get the computer ready to run, making the computer's overall performance inefficient.

.. 随着程序设计语言和编译技术的进步，推动了程序员开发翻译符号程序（即编译器）来自动把代码转换成机器代码，代替了以前低效的手工机器编码的方式，提高了程序开发的效率。但程序执行的效率还很低。而且随着计算机和 I/O 设备变得更强大，程序运行的时间减少了，相比之下，让计算机运行的准备时间变得更长了，使得计算机的整体执行效率很低。

In general, computers in the early 1950s could only do one task at a time, with the CPU spending most of the time waiting for the slow action of a human operator. Because the low efficiency of manual operation wastes precious computer time, a monitor program is introduced to assist in the completion of input, output, loading, and running programs, thereby improving the efficiency of using computers. The monitoring program is the initial prototype of the operating system, similar to the famous creature in the Cambrian explosion - "trilobite". Around 1951-1954, Swinnerton-Dwyer and others developed a monitoring program on the EDSAC (Electronic Delay Storage Automatic Calculator) computer, which may be the earliest recorded operating system prototype [#UNIX25Y]_ . This rudimentary monitoring program-like "assistant operation" process continued until the mid to late 1950s.


.. 一般情况下，五十年代初期的计算机每次只能执行一个任务， CPU 大部分时间都在等待人类操作员的缓慢操作。由于过低的人工操作效率浪费了计算机的宝贵机时，所以就引入监控程序（Monitor）辅助完成输入、输出、加载、运行程序等工作，从而提高了使用计算机的效率。监控程序就是操作系统最开始的雏形，类似寒武纪生物大爆发中的著名生物--“三叶虫”。在 1951-1954 年前后，Swinnerton-Dwyer 等在 EDSAC（Electronic Delay Storage Automatic Calculator）计算机上研制了监控程序，这也许是有记录的最早操作系统雏形 [#UNIX25Y]_ 。这种类似监控程序的初级“辅助操作”过程一直持续到 20 世纪 50 年代中后期。


.. chyyuu http://www.ict.ac.cn/jssgk/lsyg/ http://www.wyzxwk.com/Article/chanye/2020/09/424229.html  https://www.ccf.org.cn/Computing_history/Full_List/2017/Second_class/2018-09-12/652328.shtml https://www.ccf.org.cn/Computing_history/Full_List/2020/First_class/2021-01-20/722002.shtml


In China, technologies related to computers and operating systems are also developing rapidly. On August 1, 1958, under the guidance of Soviet experts, the Institute of Computing Technology of the Chinese Academy of Sciences, the Fourth Ministry of Machinery Industry (Ministry of Electronics Industry) and other units imitated the Soviet Union's M-3 small digital electronic computer, and completed the design and completion of China's first electronic tube computer. The main small digital electronic computer - 103 (namely DJS-1) computer, this computer does not have a monitoring program. On October 1, 1959, the Institute of Computing Technology of the Chinese Academy of Sciences, the Fourth Ministry of Machinery and other units cooperated to create China's first large-scale general-purpose digital electronic computer based on electronic tubes - 104 machines. The software on the 104 machine is relatively rich, including the self-test program with the prototype of the early operating system, the standard subroutine library, the automatic address replacement program, and the application-oriented algorithm language and compilation system.

.. 在中国，计算机与操作系统相关技术的发展也很快。1958 年 8 月 1 日，在苏联专家的指导下，中科院计算所、第四机械工业部（电子工业部）等单位，仿制苏联 M-3 小型数字电子计算机，设计完成了中国第一台以电子管为主的小型数字电子计算机 -- 103 型（即 DJS-1 型）计算机，这个计算机没有监控程序。1959 年 10 月 1 日，中科院计算所、四机部等单位合作，仿照苏联 БЭCM-Ⅱ 大型机，制成了中国第一台以电子管为主的大型通用数字电子计算机 -- 104 机。在 104 机上的软件已经比较丰富了，包括具有前期操作系统雏形的自检程序、标准子程序库、自动更换地址程序，以及面向应用的算法语言与编译系统。

.. note::

   .. **历史的缩影 -- 寒武纪“三叶虫”操作系统 -- LibOS**

   .. 可以在 :ref:`本书第一章 <link-chapter1>` 看到初级的“三叶虫”操作系统 -- LibOS 其实就是一个给应用提供各种服务（比如输出字符串）的库，方便了单一应用程序的开发与运行。

   **The epitome of history -- the Cambrian "trilobite" operating system -- LibOS**
   You can see the primary "trilobite" operating system in :ref:`Chapter 1 <link-chapter1>` -- LibOS is actually a library that provides various services (such as outputting strings) for applications, which is convenient development and operation of a single application.



.. 泥盆纪 [#泥盆纪]_ 鱼类时代和二叠纪  [#二叠纪]_ 两栖动物时代

Devonian [#Devonian]_ The age of fishes and the Permian [#Permian]_ The age of amphibians
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the 1950s and 1960s, computers developed to the stage of mainframes, and the computing power and I/O processing capabilities were further enhanced, the storage space was further expanded, and the application fields were gradually expanded, which made the corresponding early operating systems more diversified and specialized. Computer manufacturers design dedicated operating systems for their respective hardware, and most of them are written in assembly language. This leads to low development efficiency of the operating system, lack of portability across hardware platforms, and slow evolution. Moreover, processor time was very precious at that time, and it was a great waste to limit the use of the computer system to one user at a time. For example, in early computer systems, when a user loaded an application program through the monitor program, the processor remained idle and all other user programs could not execute. In order to improve the efficiency of computer execution, the evolution of the operating system continues, from the stage of "manual operation" and "monitoring program" to the stage of "batch processing" and "multiprogramming".

.. 在 20 世纪 50~60 年代，计算机发展到大型机阶段，计算能力和 I/O 处理能力进一步加强，存储空间进一步扩展，应用领域逐步扩大，这使得所对应的各种早期操作系统具有多样化、专用化的特点。计算机生产厂商设计出针对各自硬件的专用操作系统，且大部分用汇编语言编写。这导致操作系统的开发效率不高，不具有跨硬件平台的可移植性，进化比较缓慢。而且当时的处理器时间很宝贵，将计算机系统的一次使用过程限制在一个用户范围内是一种很大的浪费。例如，在早期计算机系统中，当用户通过监控程序加载应用程序时，处理器保持空闲状态，所有其他用户的程序都不能执行。为提高计算机执行效率，操作系统进化在持续进行，从“手工操作”和“监控程序”阶段进化到了“批处理 Batch”和“多道程序 Multiprogramming”阶段。

.. 批处理是指把一批作业（英文： **Job** ，古老的术语，可理解为现在的应用程序）以脱机方式（offline mode）输入到磁带上，并使这批作业能一个接一个地连续处理，流程如下：

Batch processing refers to inputting a batch of jobs (**Job**, an ancient term, which can be understood as the current application program) to the tape in offline mode, and enabling these jobs to be processed one by one. Continuous processing one by one, the process is as follows:

1. Load a job on tape into memory;
2. The operating system hands over the execution control to the job;
3. When the job is processed, control is returned to the operating system;
4. Repeat steps 1-3 to process the next job until all jobs are processed.

.. 1. 将磁带上的一个作业装入内存；
.. 2. 操作系统把运行控制权交给该作业；
.. 3. 当该作业处理完成后，控制权被交还给操作系统；
.. 4. 重复 1-3 的步骤处理下一个作业直到所有作业处理完毕。



.. note::

   Offline mode
   .. 脱机方式（offline mode）

   Offline-based operations are operations that are not associated with a computer. For example, put the card containing the program on the card machine, install the printing paper on the printer, etc. The opposite is the operation of the online mode, that is, the operation completed by connecting with the computer, such as the computer printing the result calculated by the running application program to the paper through the printer.

   Now, the offline mode refers to the operation performed under the disconnected network, and the online mode refers to the operation performed under the network.

   .. 基于脱机方式的操作是指没有与计算机进行关联所完成的操作。比如把包含程序的卡片放到卡片机上，把打印纸安装到打印机上等。与此相反的是联机方式（online mode）的操作，即通过与计算机相联所完成的操作，比如计算机把正在运行的应用程序所计算出来的结果通过打印机打印到纸上。

   .. 现在，脱机方式的操作更多的是指断网下进行的操作，联机方式是指在联网下进行的操作。


This can make full use of the computer system: that is, try to make the system run continuously and reduce the idle time of the CPU. Batch operating systems are divided into single-channel batch processing systems and multi-channel batch processing systems. The single-channel batch operating system can only manage one (channel) job in the memory, and cannot fully utilize all the resources in the large-scale computer system, resulting in poor overall system performance. It's like the prehistoric fish of the Devonian period [#Devonian]_ - Dunkleosteus, with a hard head armor, very strong, but slow in movement, low in sensitivity, and inseparable from water.


.. 这样能充分地利用计算机系统：即尽量使该系统连续运行，减少 CPU 的空闲时间。批处理操作系统分为单道批处理系统和多道批处理系统。单道批处理操作系统只能管理内存中的一个（道）作业，无法充分利用大型计算机系统中的所有资源，致使系统整体性能较差。这就像泥盆纪 [#泥盆纪]_ 的史前鱼类--邓氏鱼，有着坚硬的头部铠甲，很强壮，但运动缓慢，灵敏度低，离不开水。

.. note::

   .. **历史的缩影 -- 泥盆纪“邓氏鱼”操作系统 -- BatchOS**

   **Microcosm of History -- the Devonian "Deng's fish" operating system -- BatchOS**
   
   You can see the "Dunkleosteus" operating system -- BatchOS in :ref:`Chapter 2 of this book<link-chapter2>`. Although only one application can be loaded and run at a time, BatchOS allows the applications and operating system to be connected through the hardware isolation mechanism. Isolation strengthens system security and improves execution efficiency to a certain extent.

   .. 可以在 :ref:`本书第二章 <link-chapter2>` 看到“邓氏鱼”操作系统 -- BatchOS ，虽然每次只能加载运行一个应用，但BatchOS通过硬件隔离机制让 APP 与 OS 隔离，加强了系统安全，并在一定程度上提高了执行效率。


In 1956, Bob Patrick (Bob Patrick) designed the earliest batch processing operating system on the IBM 704 computer for General Motors and North American Airlines based on the system monitor of General Motors of the United States -- GM-NAA I/O [#OSHISTORY]_ . This earliest operating system already had the basic functions of a single-channel batch system.

.. 1956 年，鲍勃.帕特里克（Bob Patrick）在美国通用汽车的系统监督程序（system monitor）的基础上，为美国通用汽车和北美航空公司在 IBM 704 计算机上设计了最早的批处理操作系统--GM-NAA I/O [#OSHISTORY]_ 。这个最早的操作系统已经具有了单道批处理系统的基本功能。

In 1964, IBM Corporation developed OS/360 [#MAN]_, a unified and compatible operating system for the System/360 series of mainframe computers, which is a multiprogramming batch operating system. The multi-channel batch operating system can manage multiple (channel) jobs in the memory, which can make full use of all resources in the computer system and improve the overall performance of the system. The "multi-channel" here refers to multiple programs, which allow multiple programs to be put into the memory at the same time, and allow them to run alternately in the CPU, and they share various hardware and software resources in the system. When a program is halted by an I/O request, the CPU immediately switches to another program. This is like the amphibians in the Permian [#Permian]_, when there is temporary danger in the water or when there is not much food, they can leave the water and come to the land, and enjoy the animal and plant resources on the land.


.. 1964 年， IBM 公司开发了面向 System/360 系列大型计算机的统一可兼容的操作系统—— OS/360  [#MAN]_ ，它是一种多道批处理操作系统。多道批处理操作系统能管理内存中的多个（道）作业，可比较充分地利用计算机系统中的所有资源，提升系统整体性能。这里的“多道”是指多个程序，即允许同时把多个程序放入内存，并允许它们交替在 CPU 中运行，它们共享系统中的各种硬、软件资源。当一个程序因 I/O 请求而暂停运行时， CPU 便立即转去运行另一个程序。这就像二叠纪  [#二叠纪]_ 的两栖动物，当水中暂时有危险或食物不多的时候，可以离开水面到陆地上来，并享用陆地上的动植物资源。

.. chyyuu  https://blog.csdn.net/weixin_39716043/article/details/118742378 http://www.wyzxwk.com/Article/lishi/2021/12/445916.html http://www.wyzxwk.com/Article/chanye/2020/09/424229.html  https://www.zhihu.com/question/20984050 https://new.qq.com/omn/20210305/20210305A0G8VW00.html

However, processor sharing creates a need for program-to-program isolation, lest a bug in one program crash or corrupt the other program. For this reason, a hardware-level memory protection mechanism is further added to the computer, which limits the address space that a program can access when it is running, improves the ability of fault isolation, and reduces the isolation overhead. In addition, although the batch processing operating system improves the execution efficiency of the system, its disadvantage is poor human-computer interaction. For example, a practical challenge of batch processing computing is how to debug the application program and the operating system itself. If an error occurs in the programmer's code, it must be recoded, uploaded to memory, and then executed. This takes hours and days of time overhead, making it inconvenient for programmers to modify and debug programs. Essentially turning the computer back into a single-user system.

.. 然而，处理器共享产生了程序间隔离的需求，以避免出现一个程序中的运行错误使其他程序崩溃或损坏的情况。为此计算机中进一步添加了硬件级的内存保护机制，限制了一个程序在运行时能访问的地址空间，提高了故障隔离的能力，并减少了隔离的开销。另外，虽然批处理操作系统提高了系统的执行效率，但其缺点是人机交互性差，比如，批处理计算的一个实际挑战是如何调试应用程序和操作系统本身。如果程序员的代码出现错误，必须重新编码，上传内存，再执行。这需要花费以小时和天为单位的时间开销，使得程序员修改和调试程序很不方便。实质上是将计算机重新变成单用户系统。

In February 1965, Professor Ci Yungui of Harbin Military Engineering Institute (the predecessor of National University of Defense Technology) and others successfully developed my country's first transistor general-purpose electronic computer 441B-I. 441B-I is the second-generation computer in China. It has a prototype of the operating system with batch processing function, assembly language, FORTRAN language and standard program library and other rich software.

.. 1965 年 2 月，哈尔滨军事工程学院（国防科技大学前身）慈云桂教授等成功研制我国第一台晶体管通用电子计算机 441B-I。441B-I 是中国第二代计算机，具有批处理功能的操作系统雏形、汇编语言、FORTRAN 语言及标准程序库等丰富的软件。

.. note::

   .. **历史的缩影 -- 二叠纪“锯齿螈”、三叠纪“始初龙”和三叠纪“腔骨龙”操作系统**

   **Microcosm of History-- the Permian "Prionosuchus", the Triassic "Eoraptor", the Triassic "Coelophysis" Operating System**

   In :ref:`Chapter 3 of this book<link-chapter3>`, you can see that the Permian "Prionosuchus" operating system supports multiple applications residing in memory, forming a multiprogramming operating system -- Multiprog OS; 3 The Triassic "Eoraptor" operating system -- Coop OS has further evolved to support cooperative multi-programming, that is, it supports applications to voluntarily give up the CPU and switch to another application to continue execution, thereby improving the overall execution efficiency of the system; Triassic " "Coelophysis" operating system -- Timesharing OS can preempt the execution of applications, so that multiple applications can be executed in a fair and efficient time-sharing manner, improving the overall efficiency of the system.

   .. 在 :ref:`本书第三章 <link-chapter3>` 可以看到二叠纪“锯齿螈”操作系统支持在内存中驻留多个应用，形成多道程序操作系统 -- Multiprog OS；三叠纪“始初龙”操作系统 -- Coop OS 进一步进化，支持协作式多道程序，即支持应用程序主动放弃 CPU 并切换到另一个应用继续执行，从而提高系统整体执行效率；三叠纪“腔骨龙”操作系统 -- Timesharing OS 则可以抢占应用的执行，从而可以公平和高效地分时执行多个应用，提高系统的整体效率。

.. 侏罗纪 [#侏罗纪]_ 与白垩纪 [#白垩纪]_ 的恐龙时代

Jurassic [#Jurassic]_ and Cretaceous [#Cretaceous]_ the age of dinosaurs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In 1965, when Gordon Moore, then director of research and development at Fairchild Semiconductor Corporation, was asked to predict the future development trend of the semiconductor industry, he proposed: "When the price remains constant, the number of transistors that can be accommodated on an integrated circuit will double every year." Known as Moore's Law. In 1975, Gordon Moore's colleague David House argued that, according to Moore's Law, the performance of computer chips would also double every 18 months. This cumulative effect has led to a gradual decline in computing costs, so that software developers do not have to devote all their energy to improving processor utilization, but can start working hard to improve the user experience. For example, UNIX was developed in the early 70s on spare computers that no one was using at the time. At the end of the 1950s, time-sharing operating systems, which improved the way humans interact with computers, came to the fore. Time-sharing refers to the sharing of CPU and other hardware resources on the same computer by multiple users and programs at small intervals of time. In 1961, Fernando Corbató of the Massachusetts Institute of Technology led a team to successfully develop the CTSS (Compatible Time-Sharing System, Compatible Time-Sharing System) operating system [#UNIX25Y]_ on the IBM 709 computer, which has the necessary time-sharing system Features: Support multiple users to share and use the same computer, that is, multiple human-computer interaction tasks can be completed within the same time period macroscopically.

.. 1965 年，时任仙童半导体公司研发总监的 Gordon Moore 在被要求预测未来半导体产业发展趋势时提出：“当价格不变时，集成电路上可容纳的晶体管数目每年便会增加一倍”，这被称为摩尔定律。1975 年，Gordon Moore 的同事 David House 认为根据摩尔定律，计算机芯片的性能也将每 18 个月提升一倍。这种累积效应使得计算成本逐渐下降，使得软件开发者不必将全部精力用于提高处理器利用率，而是可以开始努力提升用户的使用体验。例如，UNIX 是在 70 年代初在当时没有人使用的备用计算机上开发的。20 世纪 50 年代末，提高人机交互方式的分时操作系统越来越崭露头角。分时是指多个用户和多个程序以很小的时间间隔来共享使用同一台计算机上的 CPU 和其他硬件资源。1961 年，麻省理工学院的 Fernando Corbató 带领团队成功研发了在 IBM 709 计算机上的 CTSS（Compatible Time-Sharing System， 兼容时间共享系统）操作系统 [#UNIX25Y]_ ，它拥有分时系统必须有的特征：支持多个用户分享使用同一台计算机，即宏观上的同一时间段内能完成多个人机交互工作。

Inspired by CTSS, in 1964, MIT, Bell Labs and General Electric jointly developed an ambitious operating system: MULTICS (MULTiplexed Information and Computing System), which is a set of an operating system that supports multiplayer and multitasking. Based on Compatible Time Sharing System (CTSS), MULTICS is built on the mainframe GE-645 of General Electric Company in the United States. The goal is to connect 1000 terminals and support 300 users to go online at the same time. Because the goal of MULTICS was too ambitious and the progress of R&D work was too slow, AT&T's Bell Labs withdrew from MULTICS R&D in 1969. CTSS and MULTICS It's like Tyrannosaurus, the huge carnivorous dinosaur of the Jurassic period, which dominated for a while, but evolved slowly and eventually became extinct.

.. 在 CTSS 的鼓舞下，1964 年，麻省理工学院、贝尔实验室及美国通用电气公司共同研发一个目标远大的操作系统：MULTICS（MULTiplexed Information and Computing System），它是一套安装在大型主机上、支持多人多任务的操作系统。 MULTICS 以兼容分时系统（CTSS）做基础，建置在美国通用电力公司的大型机 GE-645 ，目标是连接 1000 部终端机，支持 300 位用户同时上线。因 MULTICS 的目标太宏大，而研发工作进度过于缓慢，1969 年 AT&T 的 Bell 实验室从 MULTICS 研发中撤出。CTSS 和 MULTICS 这就像侏罗纪时期体型庞大的食肉恐龙--霸王龙，称霸一时，但进化缓慢，最终灭绝。

.. 但贝尔实验室的两位软件工程师 Ken Thompson 与 Dennis Ritchie 借鉴了一些重要的 MULTICS 设计思想和理念，以 C 语言为基础发展出小巧灵活的 UNIX 操作系统 [#UNIX]_ 。UNIX 操作系统的早期版本是完全免费的，可以轻易获得并随意修改，所以它得到了广泛的接受。后来，它成为开发小型机操作系统的起点。由于早期的广泛应用，它已经成为分时操作系统的典范。这好像一种生活在侏罗纪晚期的小型恐龙--始祖鸟，它可能是鸟类的祖先，最终进化为可以展翅高飞的飞鸟。

But Ken Thompson and Dennis Ritchie, two software engineers at Bell Labs, drew on some important MULTICS design ideas and concepts, and developed a small and flexible UNIX operating system [#UNIX]_ based on the C language. Early versions of the UNIX operating system were completely free and could be easily obtained and modified at will, so it gained wide acceptance. Later, it became the starting point for the development of minicomputer operating systems. Due to its early widespread use, it has become a model for time-sharing operating systems. This seems to be a small dinosaur living in the late Jurassic period - Archeopteryx, which may be the ancestor of birds, and eventually evolved into flying birds that can spread their wings and fly high.


.. note::

   Ken Thompson and Dennis Ritchie published "The UNIX Time Sharing System" in the Communications of the ACM journal in July 1974, which aroused widespread interest in the academic community and asked them for source code analysis and learning, so Unix v5 is named "only The open protocol for educational purposes" was provided to various universities as operating system teaching purposes, and became an important learning material in operating system courses at that time, and various universities and companies began to further study Unix, and improved and expanded Unix, thus making Unix widely popular in the world.

   .. Ken Thompson 与 Dennis Ritchie 于 1974 年 7 月在 the Communications of the ACM 期刊上发表 “The UNIX Time Sharing System”，引起了学术界的广泛兴趣并向他们要源码进行分析和学习，所以 Unix v5 以“仅用于教育目的”的开放协议，提供给各个大学作为操作系统教学之用，成为当时操作系统课程中的重要学习资料，而且各大学和公司开始进一步研究 Unix，并对 Unix 进行改进和扩展，从而使得 Unix 在世界上广泛流行。



.. chyyuu https://new.qq.com/omn/20210305/20210305A0G8VW00.html

In 1973, Xu Jiafu from Nanjing University, Zhong Cuihao from the Software Institute of the Chinese Academy of Sciences, and Yang Fuqing from Peking University jointly developed the system programming language XCY. XCY is composed of one letter from the Chinese pinyin of the names of Xu (Xu) Jiafu, Zhongcui (Cui) Hao, and Yang (Yang) Fuqing. The development team of Nanjing University also used the XCY language to write and develop general-purpose operating systems with time-sharing and process management capabilities such as DJS200/XT I, DJS200/XT II, ​​and XW for 240 machines.


.. 1973 年，南京大学徐家福、中科院软件所仲萃豪、北京大学杨芙清合作，研制了系统程序设计语言 XCY。XCY 由徐(Xu)家福、仲萃(Cui)豪、杨(Yang)芙清的姓名汉语拼音各取一个字母组成。南京大学的开发小组还用 XCY 语言编写开发了 240 机的 DJS200/XT I、DJS200/XT II、XW 等具有分时和进程管理能力的通用操作系统。

.. note::

   .. **历史的缩影 -- 侏罗纪和白垩纪的操作系统**

   **Microcosm of History-- The Jurassic and Cretaceous Operating Systems**

   The Jurassic and Cretaceous periods were the last prosperous period when dinosaurs dominated the earth. Jurassic "Ankylosaurus" operating system, Cretaceous "Troodont" operating system, Cretaceous "Tyrannosaurus" operating system, Cretaceous "Veloraptor" operating system, Cretaceous "Dakotaraptor" operating system , Cretaceous "Mother Dragon" operating system can be seen in :ref:`Chapter 4 of this book <link-chapter4>` to :ref:`Chapter 8 of this book<link-chapter8>`.

   .. 侏罗纪和白垩纪是恐龙称霸地球的最后繁荣时期。侏罗纪“头甲龙”操作系统、白垩纪“伤齿龙”操作系统、白垩纪“霸王龙”操作系统、白垩纪“迅猛龙”操作系统、白垩纪“达科塔盗龙”操作系统、白垩纪“慈母龙”操作系统可以在 :ref:`本书第四章 <link-chapter4>` 到  :ref:`本书第八章 <link-chapter8>` 中看到。

   :ref: `Chapter 4 <link-chapter4>` Jurassic "Ankylosaurus" operating system -- "Address Space OS" mainly solves the problem of memory isolation of multiple applications, so as to ensure that applications will not interact with each other Destroy their respective memory spaces, and further develop the core concepts of operating systems such as address space and virtual memory.

   .. :ref:`第四章 <link-chapter4>` 的侏罗纪“头甲龙”操作系统 -- “Address Space OS”主要解决多个应用所在内存隔离的问题，从而确保应用之间不会相互破坏各自的内存空间，并进一步发展出地址空间、虚拟内存等操作系统核心概念。

   :ref: `Chapter 5 <link-chapter5>` of the Cretaceous "Troodont" operating system -- "Process OS" mainly solves the problem of flexible creation and execution of multiple applications, and further develops processes, scheduling and other operating system core concepts.

   .. :ref:`第五章 <link-chapter5>` 的白垩纪“伤齿龙”操作系统 -- “Process OS”主要解决多个应用灵活创建与执行的问题，并进一步发展出进程、调度等操作系统核心概念。

   :ref:`Chapter 6<link-chapter6>` The Cretaceous "Tyrannosaurus" operating system -- "Filesystem OS" mainly solves the application requirements for persistent data storage, and further proposes the concept of files, and integrates files and address spaces. It is managed as a resource of a process, which integrates the three core concepts of the operating system.

   .. :ref:`第六章 <link-chapter6>` 的白垩纪“霸王龙”操作系统-- “Filesystem OS”主要解决数据持久保存的应用需求，并进一步提出文件的概念，并且把文件与地址空间作为进程的资源来进行管理，综合了操作系统三大核心概念。

   :ref: `Chapter 7 <link-chapter7>` of the Cretaceous "Velocus" operating system -- "IPC OS" mainly solves the application requirements of data exchange and information notification between processes, and builds part of the IPC functions in the file. Under the abstraction, the complexity of application development is simplified.

   .. :ref:`第七章 <link-chapter7>` 的白垩纪“迅猛龙”操作系统 -- “IPC OS”主要解决进程间的数据交换与信息通知的应用需求，并把部分IPC功能建立在文件的抽象之下，简化了应用开发的复杂性。

   :ref: `Chapter 8 <link-chapter8>` of the Cretaceous "Dakotaraptor" operating system -- "Thread OS" supports thread abstraction, takes threads as the scheduling unit of processors, and becomes the core of process resource management part of the content. The basis for sharing data and exchanging data between threads is established by sharing the address space of the process among threads. The Cretaceous "Mother Dragon" operating system -- "SyncMutex OS" establishes a synchronization mutual exclusion mechanism to allow threads to access shared resources in an orderly and mutually exclusive manner, so that applications based on multithreading can complete tasks efficiently and correctly.

   .. :ref:`第八章 <link-chapter8>` 的白垩纪“达科塔盗龙”操作系统 -- “Thread OS”支持线程抽象，把线程作为处理器的调度单位，并成为进程资源管理的一部分内容。线程间通过共享进程的地址空间建立了线程间共享数据和进行数据交换的基础。而白垩纪“慈母龙”操作系统 -- “SyncMutex OS”通过建立同步互斥机制来让线程间能够有序、互斥地访问共享资源，从而让基于多线程的应用 能够高效正确地完成任务。


.. 古近纪 [#古近纪]_ 哺乳动物时代

Paleogene [#Paleogene]_ The Age of Mammals
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the middle and late 1970s, the rapid development of microprocessors made the application of computers popular to small and medium-sized enterprises and individual enthusiasts, which promoted the development of PC (Personal Computer) and further promoted the operating system for the general public. These are exeplified by the DOS operating system developed (and actually purchased) by Microsoft Corporation for personal computers in the 1980s and characterized by its simplicity and ease of use. Later, MS Windows, an operating system with a graphical user interface (GUI), was developed, which greatly simplified the difficulty for ordinary users to use computers, and made personal computers popular and widely used. It should be noted here that the first personal computer prototype with a GUI interface originated from the great but lamentable Xerox Palo Alto Research Center (PARC, Palo Alto Research Center). The graphical user interface (GUI, Graphical User Interface) with pop-up menus and overlapping windows can be manipulated by clicking the mouse, which is the basis of the GUI system we use today. Supporting a convenient graphical interface has also become one of the main features of operating systems since the 1970s. This is like the mammals of the Paleogene [#Paleogene]_, which can run on land, fly in the air and swim in water, and have strong adaptability and survivability.

.. 20 世纪 70 年代中后期，微型处理器的快速发展使计算机的应用普及至中小企业及个人爱好者，推动了PC（Personal Computer，个人计算机）的发展，也进一步推动了面向一般大众使用的操作系统的出现。其代表是由微软公司在 20 世纪 80 年代为个人计算机开发（实际是购买）的 DOS 操作系统，其特点是简单易用。后来又开发了有图形用户界面（GUI）的操作系统--MS Windows，极大地简化了一般用户使用计算机的难度，使得个人计算机得到了快速的普及和广泛的使用。这里需要注意的是，第一个带 GUI 界面的个人计算机原型起源于伟大却又让人扼腕叹息的施乐帕洛阿图研究中心（PARC, Palo Alto Research Center），PARC 研发出的带有图标、弹出式菜单和重叠窗口的图形交互界面 (GUI, Graphical User Interface)，可利用鼠标的点击动作来进行操控，这是当今我们所使用的 GUI 系统的基础。支持便捷的图形交互界面也成为自 20 世纪 70 年代以来操作系统的主要特征之一。这就像古近纪 [#古近纪]_ 的哺乳动物，能在陆上跑，空中飞和水里游，有很强的适应性和生存能力。


.. chyyuu http://news.newhua.com/news/2009/1224/82274.shtml https://baike.baidu.com/item/CCDOS/2782730 https://zhuanlan.zhihu.com/p/161170166

Around 1980, the multi-machine real-time operating system GX-73 for the survey ship Yuanwang was born. In 1983, the National University of Defense Technology developed China's first 100 million times supercomputer - Galaxy-I, and developed a supporting multi-program batch processing operating system - YHOS. In the same year, the Sixth Research Institute of the Ministry of Electronics Industry designed and implemented the microcomputer-oriented operating system CCDOS (Chinese Characters Disk Operation System, English: Chinese Characters Disk Operation System), which is a Chinese version based on Microsoft's DOS microcomputer operating system. From the 1980s to the 1990s, Microsoft's DOS operating system and subsequent Windows operating system supported each other with Intel's x86 processor, forming the Wintel Alliance, which was gradually widely used in the desktop computer market. China's operating system is mainly concentrated in the scientific research field with a small range of use. During the period from 1989 to 1997, China Computer Services Corporation and China Software Technology Corporation, in conjunction with domestic universities and research institutes, jointly undertook the scientific and technological research plan to develop the COSIX operating system compatible with UNIX. COSIX has achieved a lot of scientific research results, but due to various factors such as lack of research and development based on new hardware, it has not formed a broad operating system industry ecology in the end.

.. 1980 年左右，用于远望号测量船的多机实时操作系统 GX-73 诞生了。1983 年，国防科技大学研制了中国第一台亿次巨型机 -- 银河-I，并研制了配套的多道批处理操作系统 -- YHOS。同年，电子工业部第六研究所设计实现了面向微型计算机的操作系统 CCDOS（汉字磁盘操作系统，英语：Chinese Characters Disk Operation System），这是一个基于微软公司 DOS 微机操作系统的汉化版本。80 年代到 90 年代，微软的 DOS 操作系统和后继的 Windows 操作系统和 Intel 公司的 x86 处理器相互支持，形成了 Wintel 联盟，逐渐在桌面计算机市场上被广泛使用。中国的操作系统主要集中在使用范围较小的科研领域。1989 - 1997 年这段时间，中国计算机服务总公司与中国软件技术公司联合国内高校和科研院所，共同承担了开发与 UNIX 兼容的 COSIX 操作系统的科技攻关计划。COSIX 取得了很多科研成果，但由于没基于新的硬件进行研发等多种因素，最终没有形成广泛的操作系统产业生态。

.. note::

   .. **历史的缩影 -- “侏罗猎龙”操作系统**

   **Microcosm of history -- "Juravenator" operating system**

   Creatures with dexterous perception have appeared before mammals, and this is the Juravenator of the Late Jurassic. It is small in size, has big eyes adapted to low-light environments and magical scales with pressure sensitivity, which can detect small changes in the external environment. This "small and sensitive" feature has opened up a direction for the biological evolution of subsequent mammals. :ref: `Chapter 9 <link-chapter9>`'s "Jurassic Hunter" operating system -- "Device OS" supports the interaction with the newly added keyboard, mouse, GPU and other Virtio peripherals, which can form a primary Human-computer interaction GUI capability, and support to respond to interrupts in the kernel to reduce the overall delay of I/O response. To some extent, this reflects the basic characteristics of the currently widely used desktop/mobile terminal operating systems with graphical interfaces.

   .. 在哺乳动物之前已经出现了具有灵巧感知能力的生物，这就是晚侏罗纪的侏罗猎龙。它体型小巧，有着适应微光环境的大眼睛和具备压力感应能力的神奇鳞片，能够发现外部环境的微小变化，这种“小巧灵敏”的特征为后续哺乳动物的生物进化开辟了一个方向。 :ref:`第九章 <link-chapter9>` 的“侏罗猎龙”操作系统 -- “Device OS”支持与新增的键盘、鼠标、GPU 等多种 Virtio 外设的交互，可以形成初级的人机交互 GUI 能力，而且支持在内核中响应中断来降低 I/O 响应的总体延迟。这在某种程度上体现了当前广泛使用的有图形界面的桌面/移动终端操作系统的基本特征。

第四纪智人时代 [#人类简史]_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

21 世纪以来， Internet 和移动互联网的迅猛发展，使得在服务器领域和个人终端的应用与需求大增，数据中心和个人终端已经进入了人们的日常生活中。现在我们拥有种类繁多的计算设备，在这些设备上运行着许多不同的操作系统，操作系统设计者面临的功能权衡取决于硬件的物理能力以及应用程序和用户需求。下面是一些目前常见类型的操作系统：

- 面向服务器的操作系统：当前大家常用的主流搜索引擎、新媒体、电子商务网站和大数据处理系统等一般都托管在数据中心的计算机上。每台计算机都是强大的服务器，运行着服务器操作系统，典型的例子是 Linux。通常每台服务器只运行一个应用服务程序，例如数据库服务器或 Web 服务器等，用于处理成千上万个用户传入的网络服务请求，所以吞吐量（每秒处理的请求数量）是一个关键的优化目标。而安全和可靠也是服务器操作性系统需要重点关注的目标。服务器操作系统的一种形态是虚拟机（Virtual Machine Monitor，VMM），它可在一台物理机上虚拟出多台虚拟计算机，可以像运行应用程序一样运行另一个操作系统。通过虚拟机可以充分利用数据中心中资源利用率不高的物理服务器，提高整个数据中心的运行效率。典型的服务器操作系统有：FreeBSD、微软的Windows Server、基于 Linux 系的 RHLS、Ubuntu、openEuler、龙蜥操作系统、麒麟服务器操作系统等。

- 面向台式机/笔记本电脑和上网本的操作系统：典型的例子是：Windows、Mac OS X、Linux、Chrome OS等。这些操作系统主要面向单个用户，运行许多应用程序，并具有各种 I/O 设备。有人可能认为只有一个用户，就没有必要将系统设计为支持多用户共享，这会使得操作系统设计更加简单，但安全性相对弱一些。

- 面向智能移动终端（手机/平板）的操作系统：智能移动终端是一种带有强大处理器的手机或平板，能够运行第三方应用程序。智能移动终端操作系统的典型例子包括 两个霸主 iOS、Android，和曾经辉煌过的 Symbian，还有几乎快消失的 WebOS、Blackberry OS 和 Windows Phone 等。智能移动终端一般只有一个用户，对交互响应能力、长效的电池使用时间和各种应用的支持有着迫切的需求。

- 嵌入式操作系统：随着物联网的发展，处理器芯片可以集成到各种的消费设备中，从机顶盒、手表到机器人等，形成各种物联网中的嵌入式设备。这些嵌入式设备的功能相对单一，通常运行嵌入式操作系统。典型的嵌入式操作系统有 嵌入式 Linux、VxWorks、FreeRTOS、RT-Thread、SylixOS 等。预计在下一个十年，通过面向物联网设备的操作系统，可以把各种嵌入式设备连接在一起，实现灵活多变的功能协同与组合，形成万物互联的新应用场景和生态。

从上面的简介，我们可以看到，iOS 和 Android 操作系统是 21 世纪个人终端操作系统的代表，Linux 在巨型机到数据中心服务器操作系统中占据了统治地位。以 Android 系统为例，Android 操作系统是一个包括 Linux 操作系统内核、基于 Java 的中间件、用户界面和关键应用软件的移动设备软件栈集合。这里介绍一下广泛用在服务器领域、智能移动终端和嵌入式系统中的操作系统内核--Linux 操作系统内核。1991 年 8 月，芬兰学生 Linus Torvalds \(林纳斯·托瓦兹\) 在 comp.os.minix 新闻组贴上了以下这段话： 


  ＂你好，所有使用 minix 的人 -我正在为 386 ( 486 ) AT 做一个免费的操作系统（只是为了爱好）...″


而他所说的“爱好″成为了大家都知道的 Linux 操作系统内核。这个时代的操作系统的特征是联网，提高网络的吞吐量，并降低传输延迟是这个时代的网络操作系统追求的目标。 Linux 就像是第四纪出现的智人，横扫陆地上的各种强大生物，出现在生物界的顶端，统治了整个地球。


.. chyyuu  https://www.163.com/dy/article/FIIMV82I0538KQKE.html https://blog.csdn.net/itmaster/article/details/27901

中国对 Linux 系统的引进源于在芬兰读博士的宫敏。1994 年，他回国休假，随手带了 20 张磁盘、存储了 80GB 的自由软件，其中就有 Linux。由于 Linux 基于 GPL 协议开放源代码， Linux 在国内的高校中被小范围传播。从 1999 年起，国内出现了很多基于 Linux 的操作系统公司，出现了Xteam、蓝点、中科红旗、银河麒麟、中软 Linux 等几十种发行版。但这些发行版大多数基于 Fedora/CentOS/Debian/Ubuntu 进行二次开发，并没有形成桌面计算机的应用生态，在二十年左右的时间内，大部分发行版都退出了。目前（2019 年之后）在桌面计算机领域，麒麟操作系统和统信 UOS 操作系统目前有比较好的应用发展趋势。在服务器领域，华为的 openEuler 操作系统和阿里的龙蜥操作系统借助于云计算的快速发展，形成了较好的云应用生态。在嵌入式操作系统领域，国内有不少有技术特色的操作系统，主要代表是 RT-Thread、SylixOS、LiteOS 等。

.. chyyuu note::

   目前支持联网的操作系统设计实现在本书中还没有对应的章节。但其操作系统的内核其实与分时操作系统的设计实现思路基本是一致的。在本书设计的简单分时操作系统的基础上，添加一个网卡外设的驱动和一个简单的网络协议栈，也许是另一个有趣的操作系统实验内容。

二十一世纪神人时代 [#未来简史]_
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当前，大数据、人工智能、机器学习、高速移动互联网络、AR/VR 对操作系统等系统软件带来了新的挑战。如何有效支持和利用这些技术是未来操作系统的方向。我们看到了华为逐步推出的 OpenHarmony 系统；小米也推出了物联网软件平台小米 Vela ；阿里推出了 AliOS Thing；腾讯推出了Tencent OS；苹果公司接连推出 A14、M1 等基于 ARM 的 CPU，逐步开始淘汰 X86 CPU；微软推出 Windows 10 IoT，Google 推出 Fuchsia OS。大家都在做着各种位于云、边、端操作系统的技术调整和创新，构建多种形态的网络基础设施。可以发现操作系统的外延在放大，位于云、边、端的操作系统通过多种形态的网络基础设施，跳出了传统单机为主的运行模式，支持应用程序在分布式环境下的互联、互通以及互操作，从而进一步延伸为分布式操作系统。


另外，随着人工智能和机器学习的快速发展，下一个与人工智能充分融合并带有分布式特征的操作系统即将到来，并试图通过这种操作系统带来的连贯用户体验，打通从数据中心、服务器、桌面、移动端、边缘设备等的整个 AI 和物联网 (IoT, Internet of Things) 的生态。也许这种未来操作系统与之前的操作系统相比，其最大的不同是具有了人工智能的属性，跳出了单个设备节点，通过多种网络从不同维度来管理多个设备。这种操作系统也许就是尤瓦尔·赫拉利所著的《未来简史》 [#未来简史]_ 中描述的“无所不能”的神人操作系统。

目前支持 AIoT 的操作系统设计实现在本书中还没有对应的章节，不过我们的同学也设计了 `zCore操作系统 <https://github.com/rcore-os/zCore>`_ ，欢迎看完本书的同学能够尝试参与或独立设计面向未来的操作系统。

 
.. chyyuu https://www.sohu.com/a/323094413_783821 https://it.sohu.com/20050117/n223974639.shtml  https://www.museum.uestc.edu.cn/info/1184/2337.htm




.. [#Cambrian] 5亿年前的寒武纪期间生物种类突然丰富起来，呈爆炸式的增加，期间的典型生物是三叶虫。
.. [#Devonian] 4亿年前的泥盆纪期间鱼类空前繁荣，并在晚期出现了两栖动物。
.. [#Permian] 3亿年前的二叠纪期间是一个承上启下的阶段，两栖类动物最繁盛，爬行动物逐渐繁荣。 
.. [#Jurassic] 2亿年前的侏罗纪期间温暖潮湿，爬行类动物的代表--恐龙成为当时的统治者，哺乳动物开始发展。
.. [#Cretaceous] 1亿年前的白垩纪期间温暖干旱，恐龙经历了从鼎盛到灭绝的巨大变化，哺乳动物兴起。
.. [#Paleogene] 0.6亿年前的古近纪时期，哺乳动物迅速发展，且形态多样化，逐渐统治了地面。
.. [#人类简史] 尤瓦尔·赫拉利所著的“人类简史” 书中提到的智人遍布地球，可类比现在的Linux 。
.. [#未来简史] 尤瓦尔·赫拉利所著的“未来简史” 书中描述的神人可类比于未来支持AI的分布式操作系统 。
.. [#UNIX25Y] Peter H.Salus, A Quarter Century of UNIX, Addison-Wesley Publishing, 1997
.. [#UNIX] Brain W. Kernighan, UNIX: A History and a Memoir, Independently published, 2020 
.. [#OSHISTORY] Maarten Bullynck, What is an Operating System? A historical investigation (1954–1964), https://halshs.archives-ouvertes.fr/halshs-01541602/document
.. [#MAN] 布鲁克斯(Brooks, F. P.), 人月神话(40周年中文纪念版),2015
