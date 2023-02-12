Introduction
=====================

Chapter Guide
--------------------------

This chapter primary explains why we write this book, given the presence of a series of excellent Operating System textbooks. To this end, this chapter starts with analyzing the challenges and problems that students currently face; and introduce how we come up with each chapter of this book by referring to the Operating System history and overall experiments. Afterwards, we describe what is an operating system, its access interfaces, abstractions, and characteristics from a high-level perspective, i.e., the evolution of computers and operating system. At last, we introduce the setup procedures for the experimental environments (including online, local environment and etc) affiliated to this book, to lay out a solid background for experiments hereafter. 

.. 本章主要解释了在已经有一系列优秀的操作系统教材的情况下，为何要写本书。所以本章一开始就是分析学生目前学习操作系统碰到的困难和问题，并介绍如何参考操作系统历史，结合操作系统的完整实验来设计本书的各个章节来编写本书。接下来将从非常高层次的角度和计算机以及操作系统的发展史来进一步描述了什么是操作系统、操作系统的访问接口、操作系统的抽象、操作系统特征，让同学能够对操作系统有一个大致的整体把握。最后介绍了本书关联的操作系统实验环境（包括在线实验和本地实验等）的搭建过程，为后续开展各个操作系统实验打好基础。


.. 为何要写这本操作系统书

Why we write this book
-------------------------------------------------------

There are already a series of excellent operating system textbooks for teaching, e.g, "Operating Systems Internals and Design Principles" from William Stallings, "Operating System Concepts" from Avi Silberschatz, Peter Baer Galvin and Greg Gagne,
"Operating Systems: Three Easy Pieces" from Remzi H. Arpaci-Dusseau and Andrea C. Arpaci-Dusseau.

.. 在目前的操作系统教学中，已有一系列优秀的操作系统教材，例如 William Stallings 的《Operating Systems Internals and Design Principles》，Avi Silberschatz 、 Peter Baer Galvin 和 Greg Gagne 的《Operating System Concepts》，
.. Remzi H. Arpaci-Dusseau 和 Andrea C. Arpaci-Dusseau 的《Operating Systems: Three Easy Pieces》等。


.. 有待思考的问题

Questions Worth Musing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

However, according to our teaching experiences since 2000 onwards, though some classic textbooks place overdue emphasis on operating system concepts and principles, the following problems deserve a second thought:

.. 然而，从我们自 2000 年以来的教学实践来看，某些经典教材对操作系统的概念和原理很重视，但还有如下一些问题有待进一步思考：

- Disconnection between Theory and Practice: compared to its concrete implemention, the Operation System's principles and concepts are relatively vague. Existing textbooks do not do well to bridge "**principles and concepts of operating system**" with "**its design and implementation**", widening the chasm between them. As a result, even though students are instilled with abundant concepts, they have no ideas how to imnplement an operating system, as if "talking about stratagems on paper". On the other hand, in the operating system design and implementation experiments, students tend to miss the wood for the trees, i.e., get lost into unnecessary details of hardware conventions, assembly codes, data structures, program optimization, and etc, unaware of their relationship to the operating system concepts, lacking a panoramic view and systematic methodology, hard to correspond them to basic concepts taught by teachers in lectures. 

.. - 原理与实践脱节：与操作系统的具体实现而言，操作系统的原理与概念相对过于抽象。目前的一些教材缺乏在“**操作系统的原理与概念**”和“**操作系统的设计与实现**”之间建立关联关系的桥梁，使得二者之间存在较大的鸿沟。这导致学生即使知道了操作系统的概念，还只能停留在“纸上谈兵”的阶段，依然不知如何实现一个操作系统。另外，学生在完成设计与实现操作系统的实验过程中，容易“一叶障目，不见泰山”，陷入到硬件规范、汇编代码、数据结构、编程优化等细节中，不知这些细节与操作系统概念的关系，缺少全局观和系统思维，难以与课堂上老师讲解的操作系统基本概念对应起来。

- Scarcity of the Revelation of Historical Evolution: History teaches us how booms and busts go. The operating system principles and concepts come from the historical developments of their own design and implementation, which evolve incrementally from zero along with the developments of computer hardwares and application demands, a process abundant with historical trends and tendency. However, existing textbooks only scratch the surface on the principles and concepts of current mainstream operating systems, giving a false sense that "they appear out of thin air". Students, unaware of their back and forth, are limited to **"How"** instead of **"Why"**. And in the operaing system history, many design ideas and practices go up and down and evolve continuously. They are not outdated, but emerged as new paradigms. For example, the design philosophy of **LibOS**, a prehistoric operating system, renewes its youth in the current era of Cloud Computing, as a brand-new hot topic for acadamia and internet giants to explore. 

.. - 缺少历史发展的脉络：以史为鉴，可以知兴替。操作系统的概念和原理是从实际操作系统设计与实现的历史发展过程中，随着计算机硬件和应用需求的变化，从无到有逐步演进而产生的，有其发展的历史渊源和规律。但目前的大部分教材只提及当前主流操作系统的概念和原理，有“凭空出现”的感觉，学生并不知道这些内容出现的前因后果，只知道 **“How”** ，而不知道 **“Why”** 。而且操作系统发展史上的很多设计思路和实践方法起起伏伏，不断演进，它们并没有过时，而是以新的形态出现。如操作系统远古阶段的 **LibOS** 设计思路在当前云计算时代重新焕发青春，成为学术机构和各大互联网企业探索的新热点。

- Neglection on Hardware Details or The Use of Overly Complex Hardwares: may textbooks neglect or overly abstract hardware details, difficult to put concrete operating system concepts; students can not touch on how the software and hardware coordinate. Some textbooks choose complex x86 processor as operating system experiment platform, but overlook the support on fast-developing RSIC-V and other reduced instruction set computer architectures. As a results, students have taken unduely efforts to understand overly complex x86 hardware details, which easily generate bugs in their programming, affect their experimental effects, and hinder their understanding on operating system core concepts. 

.. - 忽视硬件细节或用复杂硬件：很多教材忽视或抽象硬件细节，使得操作系统概念难以落地，学生了解不到软硬件是如何具体协同运行的。部分教材把复杂的 x86 处理器作为操作系统实验的硬件参考平台，缺乏对当前快速发展的 RISC-V 等精简体系结构的实验支持，使得学生在操作系统实验中可能需要花较大代价了解相对繁杂的 x86 硬件细节，编程容易产生缺陷（bug），影响操作系统实验的效果，以及对操作系统核心概念的掌握。

.. 解决问题的思路

Ideas to Address Problems
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These exsiting problems add to the students' difficulties to learn and understand an operating system. We attempt to address the above three problems using the following approaches, in order to relieve the student's pressure, arouse their interest, achieve the objective to understand well an operating system within a semester. 

.. 这些现存问题增加了学生学习和掌握操作系统的难度。我们尝试通过如下方法来解决上面三个问题，达到缓解学生的学习压力，提升学习兴趣，能在一个学期内比较好地掌握操作系统的目标。

In particular, to address "Disconnection between Theory and Practice", we emphasize the pedagogical philosophy of **Practice first, Practice over Principles"**. MIT Professor Frans Kasshoek and related, who implement UNIX-v6-based operating system xv6 used for undergraduate course experiments and fuse principles and experiments in the course, has gained wide recognitions across the globe and also gives us a great inspiration. From our more-than-ten-year teaching experience in operating systems, we believe: for a Computer Science undergraduate, to design and implement an operating system (including a CPU) is challeging but viable, on the premise that the operating system is neat and smart, which is capable to embody the basic core ideas of an operating system, bridge principles and concepts of the majority parts of an operating system to form a cohesive whole. And affiliated resources are required, such as the analysis documents, affiliated diagrams and video explanations on the operating system framework, core algorithms, and critical component relationships, the testing cases and environments to auto-test an operating system's functionalities, the online version control management environment to demonstrate an operating system's development phases, and an incremental comprehensive online experimental environment; so thats students are easy to deepen their understanding on the operating system concepts and principles through practices, and make these concepts and principles land. 

.. 具体而言，为应对“原理与实践脱节”的问题，我们强调 **实践先行，实践引领原理** 的教学理念。MIT 教授 Frans Kaashoek 等师生设计实现了基于 UNIX v6 的 xv6 教学操作系统用于每年的本科操作系统课的实验中，并在课程讲解中把原理和实验结合起来，在国际上得到了广泛的认可，也给了我们很好的启发。经过十多年的操作系统教学工作，我们认为：对一位计算机专业的本科生而言，设计实现一个操作系统（包括CPU）有挑战但可行，前提是这样的操作系统要简洁小巧，能体现操作系统中最基本的核心思想，并能把操作系统各主要部分的原理与概念关联起来，形成一个整体。而且还需要丰富的配套资源，比如对操作系统的整体框架、核心算法、关键组件之间的联系等的分析文档、配套的图示和视频讲解、能够自动测试操作系统功能的测试用例和测试环境、能展现操作系统逐步编写过程的在线源代码版本管理环境，以及逐步递进的综合性在线实验环境等，这样就能够让学生很方便地通过实践来加深对操作系统原理和概念的理解，并能让操作系统原理和概念落地。

To address "Scarcity of the Revelation of Historical Evolution", we overhaul the operating system experiment and course content, the content of each chapter devised according to a phase of the operating system historical developments. Each chapter is oriented towards one of core demands from applications, which is supported by the operating system, and then induce basic software and hardware knowledge points and concrete practices. In the meantime, each chapter comes with a series of incrementally stepped and independent minor experiments, each of which makes up an independent operating system and epitomizes a compressed development history of an operating system. From these, the operating system concepts and principles can be summarized and generalized. In this way, students can be guided in teaching to understand how these concepts and principles of the operating system evolve step by step. On the surface, this will require students to understand multiple different operating systems, which will increase their learning burden. But in fact, the operating system in each experiment is a gradual expansion of the operating system in the previous experiment, and students only need to understand the differences. Moreover, by analyzing the differences in application support capabilities and corresponding implementations of different operating systems, students can gain a deeper understanding of the causes and consequences of the emergence of related operating system concepts and principles. Some students may think that explaining the operating system in history is too outdated. But we believe that technology can be outdated, but ideas are worth passing on.


.. 为应对“缺少历史发展的脉络”的问题，我们重新设计操作系统实验和教学内容，按照操作系统的历史发展过程来设立每一章的内容，每一章会围绕操作系统支持应用的某个核心目标来展开，形成相应的软硬件基本知识点和具体实践内容。同时建立与每章配套的多个逐步递进且相对独立的小实验，每个实验会形成一个独立的操作系统，体现了操作系统的一个微缩的发展历史，并可从中归纳总结出操作系统相关的概念与原理。这样可以在教学中引导学生理解操作系统的这些概念和原理是如何一步一步演进的。表面上看，这样会要求同学了解多个不同的操作系统，增加了同学的学习负担。但其实每个实验中的操作系统都是在前一个实验的操作系统上的渐进式扩展，同学只需理解差异的部分即可。而且学生通过分析不同操作系统对应用支持能力和对应实现上的差异，可以更加深入地理解相关操作系统概念与原理出现的前因后果。也许有同学认为讲解历史上的操作系统太过时了。但我们认为：技术可以过时，思想值得传承。

- To address "Neglection on Hardware Details or The Use of Overly Complex Hardwares", we have tried for many years to choose between hardwares (x86, ARM, MIPS, RISC-V, etc.) and programming languages (C, C++, Go, Rust, etc.) choices. In 2017, the complex x86 architecture was replaced with a simple RISC-V architecture as the hardware environment for operating system experiments, reducing the burden on students to learn hardware details. We ushered in the Rust programming language as one of the optional programming languages ​​for operating system experiments in 2018. By relying on Rust language-specific compile-time memory/concurrency security checks, strong type static analysis, advanced data structure support, abstract encapsulation support, inline assembly and abundant third-party libraries that improve programming efficiency and reduce debugging costs, the occurrence of more runtime defects in C language programming and hardware operations can be reduced. It enables students to conduct operating system experiments with a relatively small cost of development and debugging. At the same time, we directly correspond the concepts and principles of the operating system to the actual implementation of the program code, hardware specification and operating system, so as to strengthen students' actual experience and feeling of the connotation of the operating system.


.. 为应对“忽视硬件细节或用复杂硬件”的问题，我们在硬件（x86, ARM, MIPS, RISC-V 等）和编程语言（C, C++, Go, Rust 等）选择方面进行了多年尝试。在 2017 年把 复杂的 x86 架构换为 简洁的 RISC-V 架构，作为操作系统实验的硬件环境，降低了学生学习硬件细节的负担。在 2018 年引入 Rust 编程语言作为操作系统实验的可选编程语言之一，通过Rust语言特有的编译时内存/并发安全检查、强类型静态分析、高级数据结构支持、抽象封装支持、内嵌汇编和丰富的第三方库，来提高编程效率、降低调试成本，从而减少了用C语言编程和对硬件操作出现较多运行时缺陷的情况。使得学生以相对较小的开发和调试代价进行操作系统实验。同时，我们把操作系统的概念和原理直接对应到程序代码、硬件规范和操作系统的实际执行中，加强学生对操作系统内涵的实际体验和感受。


.. 如何基于本书学习操作系统

How to Learn Operating System based on this Book
-------------------------------------------------

.. 前期准备

Preliminaries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Learning the operating system requires some preliminary preparations, mainly including basic knowledge of computer science, such as computer architectures, data structures and algorithms, programming languages, software development environments, etc. Specifically, it is necessary to understand the basic principles of computers, especially the instruction set and some privileged operations of RISC-V processors; there is also the need to master basic data structures and algorithms. After all, the operating system is also a kind of software, which requires a variety of Data structures and algorithms are used to solve problems; in the process of understanding the design of the operating system and conducting operating system experiments, it is necessary to master the system-level high-level programming language and assembly language, such as C or Rust programming language, RISC-V assembly language, for the in-depth understanding of the implementation details and design ideas of the operating system; finally, you need to master the development and experimental environment of the operating system. The main development and experimental environment involved in this book is Linux, so students need to be able to use various development tools and auxiliary tools, and mastering the IDE integrated development environment based on graphical interface or character interface, such as VSCode, Vim, Emacs, etc., which can improve the analysis of operating system source code and simplify the development and debugging process of the operating system.


.. 学习操作系统需要有一些前期准备，主要包括计算机科学基础知识，比如计算机组成原理、数据结构与算法、编程语言、软件开发环境等。具体而言，需要了解计算机的基本原理，特别是RISC-V处理器的指令集和部分特权操作；还有就是需要掌握基本的数据结构和算法，毕竟操作系统也是一种软件，需要通过多种数据结构和算法了解决问题；在了解操作系统的设计并进行操作系统实验的过程中，需要掌握系统级的高级编程语言和汇编语言，比如C或Rust编程语言，RISC-V汇编语言，这样才能深入理解操作系统的实现细节和设计思想；最后还需掌握操作系统的开发与实验环境，本书的主要涉及的开发与实验环境是Linux，所以同学们需要能够通过Linux的命令行界面使用各种开发工具和辅助工具，而掌握基于图形界面或字符界面的IDE集成开发环境，如VSCode、Vim、Emacs等，可以提高分析操作系统源码，简化操作系统的开发与调试过程。


.. 目标与步骤

Objectives and Procedures
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Therefore, the goal of this book is to take the simple RISC-V basic architecture as the underlying hardware foundation, according to the needs of upper-layer applications from small to large, and according to the historical context of OS development, gradually explain how to design and implement multiple "little" operating systems to meet the application requirements "from simple to complex". And in the process of designing and implementing the operating system, gradually analyze the knowledge points of various concepts and principles of the operating system, so that there are "reasons" to follow and "codes" to check, and finally let students go deeper through the design and implementation of the operating system to master the concepts and principles of the operating system.

.. 所以本书的目标是以简洁的 RISC-V 基本架构为底层硬件基础，根据上层应用从小到大的需求，按 OS 发展的历史脉络，逐步讲解如何设计实现能满足“从简单到复杂”应用需求的多个“小”操作系统。并且在设计实现操作系统的过程中，逐步解析操作系统各种概念与原理的知识点，做到有“理”可循和有“码”可查，最终让同学通过操作系统设计与实现来深入地掌握操作系统的概念与原理。

In this book, Chapter 0 is an overview of the operating system, allowing students to have a general understanding of the history, definitions, characteristics and other concepts of the operating system. Each of the following chapters reflects a miniature historical development process of the operating system, that is, from the perspective of supporting applications from simple to complex, each chapter will explain how to design an operating system that can run applications to meet the phased requirements of applications . In this way, students can learn how to add or enhance operating system functions from a trivial "small" operating system according to application requirements through supporting operating system design experiments, and gradually form a relatively complete "small" operating system similar to UNIX. Each step is small enough to feel manageable. And at the end of each step, you can run a "small" operating system that supports the execution of different applications.

.. 在本书中，第零章是对操作系统的一个概述，让同学对操作系统的历史、定义、特征等概念上有一个大致的了解。后面的每个章节体现了操作系统的一个微缩的历史发展过程，即从对应用由简到繁的支持角度出发，每章会讲解如何设计一个可运行应用的操作系统，满足应用的阶段性需求。从而同学可以通过配套的操作系统设计实验，了解如何从一个微不足道的“小”操作系统，根据应用需求，添加或增强操作系统功能，逐步形成一个类似 UNIX 的相对完善的“小”操作系统。每一步都小到足以让人感觉到易于掌控。而在每一步结束时，你都能运行一个支持不同应用执行的“小”操作系统。

..
  chyyuu：有一个比较大的ascii图，画出我们做出的各种OSes。

.. admonition:: **Which miniature operating systems does this book offer?**
   :class: note

   According to the development history of the operating system, we have designed the following gradually evolving "small" operating systems
  
   - LibOS: Isolate applications from hardwares, simplifying the difficulty and complexity of applications accessing hardware
   - BatchOS: isolate applications from operating systems, strengthen system security, and improve execution efficiency
   - Multiprog & Timesharing OS: Allow applications to share CPU resources
   - Address Space OS: Isolate the memory address space accessed by applications, limit the mutual interference between applicationss, and improve security
   - Process OS: Support applications to dynamically create new processes, enhance process management and resource management capabilities
   - Filesystem OS: Support applications for persistent storage of data
   - IPC OS: Support data interaction and event notification between multiple applications processes
   - Thread & Coroutine OS: Support thread and coroutine applications, simplify switching and data sharing
   - SyncMutex OS: Supports synchronized mutual exclusion access to shared resources in multi-threaded apps
   - Device OS: Improve applications's I/O efficiency and human-computer interaction capabilities, support serial ports/block devices/keyboard/mouse/display devices based on peripheral interrupts

  ..  我们按照操作系统的发展历史，设计了如下一些逐步进化的“小”操作系统
  
  ..  - LibOS: 让APP与HW隔离，简化应用访问硬件的难度和复杂性
  ..  - BatchOS： 让APP与OS隔离，加强系统安全，提高执行效率
  ..  - Multiprog & Timesharing OS: 让APP共享CPU资源
  ..  - Address Space OS: 隔离APP访问的内存地址空间，限制APP之间的互相干涉，提高安全性
  ..  - Process OS: 支持APP动态创建新进程，增强进程管理和资源管理能力
  ..  - Filesystem OS：支持APP对数据的持久保存
  ..  - IPC OS：支持多个APP进程间数据交互与事件通知 
  ..  - Thread & Coroutine OS：支持线程和协程APP，简化切换与数据共享  
  ..  - SyncMutex OS：在多线程APP中支持对共享资源的同步互斥访问
  ..  - Device OS：提高APP的I/O效率和人机交互能力，支持基于外设中断的串口/块设备/键盘/鼠标/显示设备

In addition, through a sufficiently detailed test program and an automatic test framework, it is possible to verify at any time whether the operating system implemented by the students works normally after each update. Since the code size and implementation complexity of the experiment are within a gradually increasing controllable range, students can combine the principle/concept analysis of the corresponding operating system experiment to establish the corresponding relationship between the operating system concept principle and the actual implementation, so that they can strength their understanding on concepts and theories through operating system experimental practices, and, on the other hand, use these theories and concepts to further direct their implementation and improvements over experiments. 

.. 另外，通过足够详尽的测试程序和自动测试框架，可以随时验证同学实现的操作系统在每次更新后是否正常工作。由于实验的代码规模和实现复杂度在一个逐步递增的可控范围内，同学可以结合对应操作系统实验的原理/概念分析，来建立操作系统概念原理和实际实现的对应关系，从而能够通过操作系统实验的实践过程来加强对理论概念的理解，并通过理论概念来进一步指导操作系统实验的实现与改进。

.. admonition:: **How to Learn Operating System？**
   :class: note

   It depends on your goal of learning the operating system, here are mainly divided into two categories:

   - Master the basic principles first, and understand the specific implementation as a supplement (general learning)

     - Comprehensible learning method: read and practice chapter by chapter, read and analyze applications, and master OS principles by analyzing the dynamic execution process of applications and OS.

   - Master the implementation and principle of the operating system (in-depth study)

     - Constructive learning: based on the comprehensible learning method, further analyze the source code, gradually understand the internal incremental implementation of each OS, and refer to and based on these small OSs, expand some OS functions, and pass the test cases, so as to master the operation at the same time system implementation and principle.

  ..  这取决于你想学习操作系统的目标，这里主要分为两类：

  ..  - 掌握基本原理为主，了解具体实现为辅（一般学习）

  ..    - 理解式学习方式：逐章阅读与实践，阅读分析应用，并通过分析应用与OS的动态执行过程，掌握OS原理。

  ..  - 掌握操作系统实现和原理为主（深入学习）

  ..    - 构造式学习：在理解式学习方式基础上，进一步分析源码，逐步深入了解每个OS的内部增量实现，并且参考并基于这些小OS，扩展部分OS功能，通过测试用例，从而同时掌握操作系统实现和原理。

.. 编程语言与硬件环境

Programming Language and Hardware Environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before you start reading and practicing the content of this book, you need to decide what programming language to use to complete the operating system experiment. You can choose your favorite programming language and implement the operating system on your favorite CPU. Our recommended programming languages ​and architectures are Rust and RISC-V, respectively.


.. 在你开始阅读与实践本书讲解的内容之前，你需要决定用什么编程语言来完成操作系统实验。你可以选择你喜欢的编程语言和在你喜欢的CPU上来实现操作系统。我们推荐的编程语言和架构分别是 Rust 和 RISC-V。



.. admonition:: **Choices Programming Languages and Instruction Sets**
   :class: note

   **Currently common operating system kernels are based on the C language, why recommend the Rust language?**
   - In fact, the C language was born to write UNIX. Dennis Ritchie and Ken Thompson did not expect to design a new language to help efficiently develop complex and concurrent operating system logic (for the future), but hoped to use a concise way to replace the difficult-to-use assembly language to abstract computer behavior. It is convenient to write an operating system that controls computer hardware (according to the actual situation at the time).
   - Pointers in the C language are both angels and devils. It's flexible and easy to use, but the language itself offers few guarantees of safety and lacks effective concurrency support. This leads to memory and concurrency bugs that are currently a nightmare for mainstream C-based operating systems.
   - The Rust language has the same hardware control capabilities as C, and greatly strengthens the capabilities of safe programming and abstract programming. From a certain point of view, the core goal of the emerging Rust language is to solve the shortcomings of C and replace C. So writing OS with Rust has a good development and running experience.
   - The cost of writing an OS in Rust is just learning to program in Rust.

   
   **The current common instruction set architectures are x86 and ARM, why recommend RISC-V?**
   
   - By far the most common instruction set architectures are x86 and ARM, which are widely used in servers, desktops, mobile terminals and many embedded systems. Due to their generality and backward compatibility requirements, the need to support a very large number (including implementations decades ago) of software systems and application requirements has led to the increasing complexity of these instruction set architectures.
   - The x86 backward compatibility strategy ensures its status in the desktop and server fields, but it cannot lose many outdated hardware designs, allowing the operating system to adapt to various new and old hardware features through redundant code.
   - Both x86 and ARM are commercially successful, and their widespread use makes their CPU hardware logic more and more complex, and it is not open enough to change, not open source, and it is difficult for students who are interested in exploring hardware to understand hardware details. To a certain extent, the CPU becomes a black box, and makes the interaction between the operating system and the hardware less transparent, increasing the burden of learning the operating system.
   - From a certain point of view, the core goal of the emerging RISC-V is to flexibly adapt to future AIoT (Artificial Intelligence Internet of Things, AI + IoT) scenarios, guarantee basic functions, and provide configurable expansion functions. Its open source feature allows students to go deep into the details of CPU operation, and even design a RISC-V CPU conveniently. This can help students gain a deep understanding of the cooperative execution process of the operating system and hardware.
   - The hardware learning cost of writing a RISC-V-oriented OS is just that you understand the Supervisor privileged mode of RISC-V and know the control ability of the OS in the Supervisor privileged mode.

  ..  **目前常见的操作系统内核都是基于 C 语言的，为何要推荐 Rust 语言？**
   
  ..  - 事实上， C 语言就是为写 UNIX 而诞生的。Dennis Ritchie 和 KenThompson 没有期望设计一种新语言能帮助高效地开发复杂与并发的操作系统逻辑(面向未来)，而是希望用一种简洁的方式来代替难以使用的汇编语言抽象出计算机的行为，便于编写控制计算机硬件的操作系统（符合当时实际情况）。
  ..  - C 语言的指针既是天使又是魔鬼。它灵活且易于使用，但语言本身几乎不保证安全性，且缺少有效的并发支持。这导致内存和并发漏洞成为当前基于 C 语言的主流操作系统的噩梦。
  ..  - Rust 语言具有与 C 一样的硬件控制能力，且大大强化了安全编程和抽象编程能力。从某种角度上看，新出现的 Rust 语言的核心目标是解决 C 的短板，取代 C 。所以用 Rust 写 OS 具有很好的开发和运行体验。
  ..  - 用 Rust 写 OS 的代价仅仅是学会用 Rust 编程。

  ..  **目前常见的指令集架构是 x86 和 ARM ，为何要推荐 RISC-V ？**
   
  ..  - 目前为止最常见的指令集架构是 x86 和 ARM ，它们已广泛应用在服务器、台式机、移动终端和很多嵌入式系统中。由于它们的通用性和向后兼容性需求，需要支持非常多（包括几十年前实现）的软件系统和应用需求，导致这些指令集架构越来越复杂。
  ..  - x86 后向兼容的策略确保了它在桌面和服务器领域的江湖地位，但导致其丢不掉很多已经比较过时的硬件设计，让操作系统通过冗余的代码来适配各种新老硬件特征。
  ..  - x86 和 ARM 在商业上都很成功，其广泛使用使得其 CPU 硬件逻辑越来越复杂，且不够开放，不能改变，不是开源的，难以让感兴趣探索硬件的学生了解硬件细节，在某种程度上让CPU成为了一个黑盒子，并使得操作系统与硬件的交互变得不那么透明，增加了学习操作系统的负担。
  ..  - 从某种角度上看，新出现的 RISC-V 的核心目标是灵活适应未来的 AIoT （人工智能物联网, AI + IoT）场景，保证基本功能，提供可配置的扩展功能。其开源特征使得学生都可以深入CPU的运行细节，甚至可以方便地设计一个 RISC-V CPU。从而可帮助学生深入了解操作系统与硬件的协同执行过程。
  ..  - 编写面向 RISC-V 的 OS 的硬件学习代价仅仅是你了解 RISC-V 的 Supervisor 特权模式，知道 OS 在 Supervisor 特权模式下的控制能力。

.. 本书章节导引

Book Chapter Guide
-----------------------------------------------

This book consists of 10 chapters from 0 to 9, of which Chapter 0 is the review of the book, explaining why we write this book and summarizing a concise operating system development history, the operating system definition, system call interfaces, the operating system abstraction and characteristics. Chapter 0 also explains how to learn the operation system from this book. 

.. 本书由0~9共10章组成，其中第0章是本书的总览，介绍了为何写本书，概述了操作系统的简要发展历史，操作系统的定义，系统调用接口，操作系统的抽象表示和特征等，以及如何基于本书来学习操作系统。

Chapter 1 mainly explains how to use the operating system to isolate applications and hardware to simplify application progarmming. It also illustrates how to design and implement a runtime environment on a bare-metal machine, and how to code a application that displays "Hello World" in the bare-metal machine runtime. At last, we come up with with a Cambrian "Trilobita" operaing system -- LibOS. So that students can have a thorough and in-depth understanding on the abstract concepts and concrete implementations of applications and their dependent runtime environments. 


.. 第1章主要讲解了如何通过操作系统来解决应用和硬件隔离达到简化应用编程的问题。并详细讲述了如何设计和实现建立在裸机上的执行环境，如何编写可在裸机执行环境上运行的显示“Hello Worold”的应用程序。最终形成可运行在裸机上的寒武纪“三叶虫”操作系统 -- LibOS。这样学生能对应用程序和它所依赖的执行环境的抽象概念与具体实现有一个全面和深入的理解。

Chapter 2 mainly explains how to use the operating system to address the two core issues -- system security and multi-application support. It also describes in detail how to design application programs, how to support the automatic loading and running of multiple programs through batch processing, and how to realize the isolation of application programs and operating systems in terms of execution privileges. The result is BatchOS, a Devonian "Dunkleosteus" operating system that can run multiple applications. In this way, students can see the specific implementation of concepts such as system calls, privilege levels, and batch processing on the operating system, and understand how to improve the overall performance of the system through batch processing, how to protect the operating system through privilege isolation, and how to achieve cross-privilege-level system calls and other operating system core techniques. 


.. 第2章主要讲解了如何通过操作系统来保障系统安全和多应用支持这两个核心问题。并详细讲述了应该如何设计应用程序，如何通过批处理方式支持多个程序的自动加载和运行，如何实现应用程序与操作系统在执行特权上的隔离。最终形成可运行多个应用程序的泥盆纪“邓式鱼”操作系统 -- BatchOS。这样学生可以看到系统调用、特权级、批处理等概念在操作系统上的具体实现，并了解如何通过批处理方式提高系统的整体性能，如何通过特许权隔离来保护操作系统，如何实现跨特权级的系统调用等操作系统核心技术。

Chapter 3 mainly explains how to improve the overall performance and fairness of multi-program execution. And it describes in detail how to reduce the overhead of application switching by loading applications into the memory in advance, how to support the program to actively give up the processor and improve the overall performance of the system through the cooperation mechanism between applications, and how to support it through the preemption mechanism based on hardware interrupts. The program passively gives up the processor to ensure the fairness of the use of processor resources by different programs, and further improves the response efficiency of the application to I/O events. Finally, our operating system evolves into the Permian "Prionosuchus" operating system that supports multi-programming -- MultiprogOS, the Triassic "Eoraptor" operating system that supports the cooperation mechanism -- CoopOS, the Triassic "Coelophysis" operating system that supports time-sharing and multitasking -- TimesharingOS. In this way, by analyzing the design and implementation of these operating systems, students can extract the core concepts of operating systems such as tasks and task switching, and have a deeper understanding of the interrupt processing mechanism of computer hardware and the time-sharing mechanism of operating systems.


.. 第3章主要讲解了如何在提高多程序运行的整体性能并保证多个程序运行的公平性这两个核心问题。并详细讲述了如何通过提前加载应用程序到内存来减少应用程序切换开销，如何通过应用程序之间的协作机制来支持程序主动放弃处理器并提高系统整体性能，如何通过基于硬件中断的抢占机制支持程序被动放弃处理器来保证不同程序对处理器资源使用的公平性，也进一步提高了应用对 I/O 事件的响应效率。最终形成了支持多道程序的二叠纪“锯齿螈” 操作系统 -- MultiprogOS，支持协作机制的三叠纪“始初龙” 操作系统 -- CoopOS，支持分时多任务的三叠纪“腔骨龙” 操作系统 -- TimesharingOS。这样学生可以通过分析这些操作系统的设计与实现，提炼出任务、任务切换等操作系统的核心概念，对计算机硬件的中断处理机制、操作系统的分时共享等机制有更深入的理解。


Chapter 4 mainly explains the security isolation and efficient use of memory. Limited physical memory is an important resource that the operating system needs to manage. How to allow multiple applications running on a computer to obtain unlimited memory space, how to isolate the memory space that running applications can access and ensure that different applications memory safety is the focus of this chapter. To this end, it is necessary to understand the page table and TLB mechanism in computer hardware, and use the operating system to build page tables for itself and different applications in the memory, establishing gmemory isolation between applications and between applications and the operating system, so as to solve the problem of memory safety isolation issues. Through technologies such as page fault exception and dynamic modification of page table, the data currently being accessed or about to be accessed by the currently running application is located in the memory, and the infrequently used data is cached in the storage device (such as a hard disk, etc.), forming a time-sharing memory operation. System capability, that is, "virtual storage" capability. Finally, the resulting operating system evolves into the Jurassic "Ankylosaurus" operating system that supports memory isolation -- Address Space OS. By analyzing the design and implementation of the operating system, students can connect the abstract concept of address space with the specific design of the page table, and master how to realize the address space through the page table mechanism. There will also be a deeper understanding of the address space switching mechanism added in task switching. Students can understand whether various page replacement strategies in the virtual memory mechanism can be effectively implemented, and how to implement them.

.. 第4章主要讲解了内存的安全隔离问题和高效使用问题。有限的物理内存是操作系统需要管理的一个重要资源，如何让运行在一台计算机上的多个应用程序得到无限大的内存空间，如何能够隔离运行应用能访问的内存空间并保证不同应用之间的内存安全是本章要重点解决的问题。为此需要了解计算机硬件中的页表和TLB机制，并通过操作系统在内存中构建面向自身和不同应用的页表，形成应用与应用之间、应用与操作系统之间的内存隔离，从而解决内存安全隔离问题。通过缺页异常和动态修改页表等技术，让当前运行的应用正在或即将访问的数据位于内存中，不常用的数据缓存放到存储设备（如硬盘等），形成分时复用内存的操作系统能力，即“虚存”能力。最终形成支持内存隔离的侏罗纪“头甲龙”操作系统 -- Address Space OS。学生通过分析操作系统的设计与实现，可以把地址空间这样的抽象概念和页表的具体设计建立起联系，掌握如何通过页表机制来实现地址空间。对任务切换中增加的地址空间切换机制也会有更深入的了解。能够理解虚存机制中的各种页面置换策略能否有效实现，以及如何具体实现。

Chapter 5 mainly explains how to improve the flexibility and interactivity of the dynamic execution of applications, that is, the management issues that allow developers to control the creation, operation and exit of programs in time. Before Chapter 5, during the entire execution process of the operating system, the application program is passively loaded and run by the operating system, there is no interaction between the developer and the operating system, there is no interaction between the developer and the application program, and the application program cannot control other application execution. This makes it impossible for the user to flexibly choose to execute a certain program. This requires providing users with a flexible application program (commonly known as the shell), forming a command line interface (Command Line Interface) for users to interact with the operating system. Users can enter commands in this `shell` program to start or kill applications, or monitor the health of the system, allowing developers to control the system more flexibly. This new user demand needs to reconstruct the function of the operating system, so that the operating system provides services that support the dynamic creation/destruction/waiting/suspension of applications. This is a further new abstraction based on the existing `task` abstraction: `process`, which is used to represent and manage the entire execution process of the application. In this way, the Cretaceous "Troodon" operating system with flexible and powerful process management functions--Process OS was finally born. By analyzing the design and implementation of the operating system, students can combine abstract concepts such as process, process scheduling, process switching, process state, and process life cycle with the process control block data structure, process-related system call functions, and process scheduling in the operating system implementation. Establishing its relationship with the specific design of the process switching function can help us to better grasp the core concept of the operating system, the process.


.. 第5章主要讲解了如何提高应用程序动态执行的灵活性和交互性的问题，即让开发者能够及时控制程序的创建、运行和退出的管理问题。在第5章之前，在操作系统整个执行过程中，应用程序是被动地被操作系统加载运行，开发者与操作系统之间没有交互，开发者与应用程序之间没有交互，应用程序不能控制其它应用的执行。这使得用户不能灵活地选择执行某个程序。这需要给用户提供一个灵活的应用程序（俗称 shell ），形成用户与操作系统进行交互的命令行界面（Command Line Interface）。用户可以在这个 `shell` 程序中输入命令即可启动或杀死应用，或者监控系统的运行状况，使得开发者可以更加灵活地控制系统。这种新的用户需求需要重构操作系统的功能，让操作系统提供支持应用程序动态创建/销毁/等待/暂停等服务。这就在已有的 `任务` 抽象的基础上进一步新抽象： `进程` ，用于表示和管理应用程序的整个执行过程。这样最终形成具备灵活强大的进程管理功能的白垩纪“伤齿龙”操作系统 -- Process OS。学生通过分析操作系统的设计与实现，可以把进程、进程调度、进程切换、进程状态、进程生命周期这样的抽象概念与操作系统实现中的进程控制块数据结构、进程相关系统调用功能、进程调度与进程切换函数的具体设计建立其联系，能够更加深入掌握进程这一操作系统的核心概念。

Chapter 6 mainly explains how to let the program access the data on the storage device conveniently. Since the data stored in the internal memory will disappear after the computer is turned off or the power is turned off, the application program should store the data in the internal memory that needs to be stored for a long time on the storage device, and read it into the internal memory for processing when needed. The emergence of files and file systems has greatly simplified the operation of applications accessing data on storage devices. Chapter 6 will design and implement the operating system and core modules, that is, a simple file system -- easyfs, which provides applications with two abstractions of regular files and directory files, and provides `open`, `close`, `read `, `write` four system calls to read and write the data in the file, and manage the I/O peripheral physical resource of the storage device through the storage device driver. Then evolves the "Tyrannosaurus" operating system that supports file access -- Filesystem OS. By analyzing the design and implementation of the operating system, students can see how operating system abstractions such as files and file systems are embodied through a specific file system -- easyfs. And you can see and understand the close connection between the file system, process management, and memory management, so as to support the application program to conveniently access the data on the storage device.


.. 第6章主要讲解了如何让程序方便地访问存储设备上的数据的问题。由于放在内存中的数据在计算机关机或掉电后就会消失，所以应用程序要把内存中需要长久保存的数据放到存储设备上存起来，并在需要的时候能读到内存中进行处理。文件和文件系统的出现极大地简化了应用程序访问存储设备上数据的操作。第6章将设计并实现操作系统和核心模块，即一个简单的文件系统 -- easyfs，向上给应用程序提供了常规文件和目录文件两种抽象，并提供 `open` 、 `close` 、 `read` 、 `write` 四个系统调用来读写文件中的数据，向下通过存储设备驱动程序对存储设备这种 I/O 外设物理资源进行管理。这样就形成了支持文件访问的 “霸王龙” 操作系统 -- Filesystem OS。学生通过分析操作系统的设计与实现，可以看到文件、文件系统这样的操作系统抽象如何通过一个具体的文件系统 -- easyfs 来体现的。并可以看到并理解文件系统与进程管理、内存管理之间的紧密联系，从而支持应用程序便捷地对存储设备上的数据进行访问。

Chapter 7 mainly explains how to allow different applications to share and cooperate with each other. Before Chapter 7, the processes were completely isolated by the operating system, which made it impossible for the processes to share data conveniently and cooperate together. If different processes can realize data sharing and interaction, the functions of different programs can be combined to realize more powerful and flexible complex functions. The core goal of Chapter 7 is to allow different applications to run together through inter-process communication. To this end, a new operating system concept - pipe (pipe) will be introduced to support the I/O redirection function between processes, that is, the output of one process becomes the input of another process, so that the processes can cooperate effectively stand up. In this way, the pipeline can actually be regarded as a special memory file, and the memory data sharing between processes can be realized based on the operation of the file. In addition to the data sharing mechanism, a fast notification mechanism is also required between processes, which leads to the Signal (Signal) event notification mechanism, so that the process can obtain and process emergency notifications from other processes or operating systems in a timely manner. In this way, finally we finally evolve into the Cretaceous "Velociraptor" operating system -- IPC OS -- which supports data interaction and event notification functions between multiple application processes. By analyzing the design and implementation of the operating system, students can see that the isolation and sharing between processes can be achieved at the same time, and can further understand how to use the pipeline mechanism to break the address space isolation established between processes on the basis of processes, and realize data sharing, and how to interrupt the normal execution of the process through the signal mechanism to respond to relatively urgent events in a timely manner, so as to master the operating system mechanism of multi-application sharing and coordination.

.. 第7章主要讲解如何让不同的应用进行数据共享与合作的问题。在第7章之前，进程之间被操作系统彻底隔离了，导致进程之间无法方便地分享数据，不能一起协作。如果能让不同进程实现数据共享与交互，就能把不同程序的功能组合在一起，实现更加强大和灵活的复杂功能。第7章的核心目标就是让不同应用通过进程间通信的方式组合在一起运行。为此，将引入新的操作系统概念 -- 管道（pipe），以支持进程间的I/O重定向功能，即让一个进程的输出成为另外一个进程的输入，从而让进程间能够有效地合作起来。这样管道其实也可以看成是一种特殊的内存文件，并可基于文件的操作来实现进程间的内存数据共享。除了数据共享机制，进程间也需要快捷的通知机制，这就引出了信号（Signal） 事件通知机制，让进程能够及时的获得并处理来自其他进程或操作系统发的紧急通知。这样最终形成支持多个APP进程间数据交互与事件通知功能的白垩纪“迅猛龙”操作系统 -- IPC OS。学生通过分析操作系统的设计与实现，可以看到进程间的隔离和共享是可以同时做到的，并可进一步了解在进程的基础上如何通过管道机制来打破进程间建立的地址空间隔离，实现数据共享，以及如何通过信号机制打断进程的正常执行来及时响应相对紧急的事件，从而掌握多应用共享协同的操作系统机制。

Chapter 8 mainly explains how to improve the efficiency of concurrent execution of multiple applications and how to ensure that multiple applications can correctly access shared resources. The address space isolation of the process will bring management runtime overhead, such as TLB flushing, page table switching, and so on. If multiple tasks that can be executed in parallel in a process are scheduled by the operating system in a finer-grained manner, then concurrent execution can be realized in the process. And since these tasks are in the same address space of the process, it will not bring runtime overhead such as page table switching. The task here is a Thread. The shared address space between threads makes it more convenient for them to access shared resources, but if not handled properly, resource access conflicts and competition problems may occur. This requires a synchronization mechanism to coordinate the execution order of processes or threads, and a mutual exclusion mechanism to ensure that only one process or thread can access shared resources at the same time, thereby avoiding resource conflicts and competition. Chapter 8 refactors on the basis of process management, designs and implements the thread management mechanism. And then evolves the Dakotaraptor OS -- ThreadOS that supports multi-threading applications. Furthermore, we design the lock, semaphore and condition variable mechanism that supports thread synchronization and mutual exclusion to access shared resources. Finally, we come up with the Cretaceous "
maiasaura" operating system - SyncMutex OS that supports multi-threaded application with synchronous mutually exclusive access to shared resources. By analyzing the design and implementation of the operating system, students can understand the relationship and difference between threads and processes, and understand the different characteristics and operating mechanisms of the synchronization mutual exclusion mechanism, so that they can deeply understand the principle and implementation of the synchronization mutual exclusion mechanism that supports concurrent access to shared resources .


.. 第8章主要讲解如何提高多个应用并发执行的效率和如何保证能多个应用正确访问共享资源的问题。进程的地址空间隔离会带来管理上的运行时开销，比如TLB刷新、页表切换等。如果把一个进程内的多个可并行执行的任务通过一种更细粒度的方式让操作系统进行调度，那么就可以在进程内实现并发执行，且由于这些任务在进程内的地址空间中，不会带来页表切换等运行时开销。这里的任务就是线程（Thread）。线程间共享地址空间，使得它们访问共享资源更加方便，但如果处理不当，就可能出现资源访问冲突和竞争的问题。这就需要通过同步机制来协调进程或线程的执行顺序，并通过互斥机制来保证在同一时刻只有一个进程或线程可以访问共享资源，从而避免了资源冲突和竞争的问题。第8章在进程管理的基础上进行重构，设计实现了线程管理机制，形成了支持多线程app的达科塔盗龙OS -- ThreadOS；并进一步设计了支持线程同步互斥访问共享资源的锁机制、信号量机制和条件变量机制，最终形成了支持多线程APP同步互斥访问共享资源的白垩纪“慈母龙”操作系统 -- SyncMutex OS。学生通过分析操作系统的设计与实现，可以理解线程和进程的关系与区别，理解同步互斥机制的不同特征和运行机理，从而能够深入理解支持并发访问共享资源的同步互斥机制的原理和实现。

Chapter 9 mainly explains how to make applications conveniently access I/O devices and make applications have more perception and interaction capabilities. The peripherals in the computer have different characteristics, such as graphics card, touch screen, keyboard, mouse, network card, sound card, etc. Before Chapter 9, students have been exposed to serial ports, clocks, and disk devices, so that applications can input and output characters, access time, read and write data on disks through the operating system, and let the operating system have the ability of preemptive time-sharing and multi-task scheduling, but this only covers a small part of the peripherals. And in practice, the details of the interaction between the operating system and the peripherals are not involved. The operating system needs to have more in-depth understanding of external devices in order to effectively manage and access peripherals and provide applications with rich perception and interaction capabilities. In terms of principles and concepts, Chapter 9 briefly analyzes the development process of peripherals and the data transmission methods of peripherals. It further elaborates how the operating system establishes different levels of abstraction and different I/O execution models for the peripherals, so as to facilitate the internal management of the peripherals by the operating system and the efficient and convenient access to the peripherals by applications. In practice, Chapter 9 analyzes how the operating system resolves the peripheral information in the computer through the Device Tree, and redesigns the interrupt-based serial port driver, involving serial device initialization and serial data input and output, and improve the scheduling mechanism of the process/thread, so that the process/thread waiting for the serial input or output to complete enters the blocking state, thereby improving the overall execution efficiency of the system. In Chapter 9, the virtio device architecture simulated by QEMU and the main functions of the virtio device driver are further introduced; and the virtio-blk device and its driver, virtio-gpu device and its driver are analyzed in depth. In this way, a Jurassic Jurassic Hunter operating system that supports graphics game applications and has efficient peripheral interrupt response--Device OS. By analyzing the design and implementation of the operating system, students can deeply understand the characteristics of different peripherals, the peripheral I /O transfer methods, different levels of peripheral abstraction and I/O execution models, so as to have a related and complete understanding of how the operating system can effectively manage different types of peripherals.


.. 第9章主要讲解如何让应用便捷访问I/O设备并让应用有更多感知与交互能力的问题。计算机中的外设特征各异，如显卡、触摸屏、键盘、鼠标、网卡、声卡等。在第9章之前，同学们已经接触到了串口、时钟、和磁盘设备，使得应用程序能通过操作系统输入输出字符、访问时间、读写在磁盘上的数据，并通过时钟中断让操作系统具有了抢占式分时多任务调度的能力，但这仅仅覆盖了很小的一部分外设，而且在实践上对操作系统与外设的交互细节也涉及不多。操作系统需要对外设有更多的深入理解，才能有效地管理和访问外设，给应用提供丰富的感知与交互能力。在原理与概念方面，第9章简要分析了外设的发展历程，外设的数据传输方式。并进一步阐述操作系统如何对外设建立不同层次的抽象和不同I/O执行模型，以便于操作系统对外设的内部管理，应用程序对外设的高效便捷访问。在实践上，第9章分析了操作系统如何通过设备树(Device Tree)来解析出计算机中的外设信息，并重新设计了基于中断方式的串口驱动程序，涉及串口设备初始化和串口数据输入输出，以及改进进程/线程的调度机制，让等待串口输入或输出完成的进程/线程进入阻塞状态，从而提高系统整体执行效率。在第9章还进一步介绍了QEMU模拟的virtio设备架构，以及virtio设备驱动程序的主要功能；并对virtio-blk设备及其驱动程序，virtio-gpu设备及其驱动程序进行了比较深入的分析。这样最终形成支持图形游戏APP并具备高效外设中断响应的侏罗纪侏罗猎龙操作系统 -- Device 学生通过分析操作系统的设计与实现，可以深入了解不同外设的特征，外设的I/O传输方式，不同层次的外设抽象概念和I/O执行模型，从而对操作系统如何有效管理不同类型的外设有一个相关完整的理解。

Hearing is believing once, if students can gradually clarify the application requirements and problems to be solved in each chapter through reading and reading the code, gradually understand the composition of the kernel module in the operating system in each chapter, and master the functions of the kernel module, and The relationship between different kernel modules can sum up the design ideas, strategies and mechanisms, principles and concepts of the operating system, and you can reach the level of understanding the operating system. Seeing is worse than doing, just reading is not enough. The important goal of this book is to promote students to master the operating system through programming. If students can complete the new functions of the operating system through after-school exercises and programming experiments, find and fix bugs in programming, and realize the operating system you wrote by yourself through test cases, then you will achieve a higher level of mastery of the operating system. I hope that students can go through the entire process of learning and practicing the operating system. After you complete the whole process, look back and find that the original operating system can still be so interesting and useful.

.. 百闻不如一见，如果同学们通过读书和阅读代码能逐步地明确每一章要解决的应用需求和问题，渐进地了解每章操作系统中内核模块的组成，并掌握内核模块的功能，以及不同内核模块之间的关系，能归纳总结出操作系统的设计思路、策略与机制、原理与概念，就能达到了解操作系统的层次。百见不如一干，仅仅看还是不够的，本书的重要目标是希望能推动同学们能够通过编程来掌握操作系统。如果同学们还能通过课后习题和编程实验来完成操作系统的新功能，发现编程中的bug并修复bug，通过测试用例，实现你自己编写的操作系统，那将达到掌握操作系统的更高层次。希望同学们能够完整走完整个操作系统的学习和练习的过程，当你完成整个过程后，再回首看，能够发现原来操作系统还可以这样有趣和有用。