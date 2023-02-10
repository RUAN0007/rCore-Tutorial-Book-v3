.. rCore-Tutorial-Book-v3 documentation master file, created by
   sphinx-quickstart on Thu Oct 29 22:25:54 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

rCore-Tutorial-Book Third Edition

.. rCore-Tutorial-Book-v3 documentation master file, created by

==================================================

.. toctree::
   :maxdepth: 2
   :caption: Part1 - Just do it!
   :hidden:
   
   chapter0/index
   chapter1/index
   chapter2/index
   chapter3/index
   chapter4/index
   chapter5/index
   chapter6/index
   chapter7/index
   chapter8/index
   chapter9/index

.. toctree::
   :maxdepth: 2
   :caption: Part2 - Do it better!
   :hidden:

.. toctree::
   :maxdepth: 2
   :caption: Appendix
   :hidden:

   final-lab
   appendix-a/index
   appendix-b/index
   appendix-c/index
   appendix-d/index
   appendix-e/index
   appendix-f/index
   terminology

.. toctree::
   :maxdepth: 2
   :caption: Notes on Development
   :hidden:

   setup-sphinx
   rest-example
   log

Welcome to rCore-Tutorial-Book Third Edition!

..  欢迎来到 rCore-Tutorial-Book 第三版！

Welcome to `2022 Open Source Operating System Bootcamp! <https://learningos.github.io/rust-based-os-comp2022/>`_

..  欢迎参加 `2022年开源操作系统训练营! <https://learningos.github.io/rust-based-os-comp2022/>`_

.. note::

   :doc:`/log` 



Project Overview

..  项目简介

---------------------

This tutorial aims to demonstrate step-by-step how **from zero** use **Rust** to develop a **Unix-like kernel** in **RISV-V** architecture. Worthy of attention, this project is not only capable to run in the emulator environment (like Qemu/terminus or so), but also in real hardware platform like Kendryte K210. 

.. 这本教程旨在一步一步展示如何 **从零开始** 用 **Rust** 语言写一个基于 **RISC-V** 架构的 **类 Unix 内核** 。值得注意的是，本项目不仅支持模拟器环境（如 Qemu/terminus 等），还支持在真实硬件平台 Kendryte K210 上运行。


.. 导读

Guide
---------------------

Please go through :ref:`Chapter 0 <link-chapter0>`, to have a general grasp over the project development background and operating system concepts. 

.. 请大家先阅读 :ref:`第零章 <link-chapter0>` ，对于项目的开发背景和操作系统的概念有一个整体把控。

Before formally embarking on experiments, please follow :doc:`/chapter0/5setup-devel-env` at the end of Chapter 0 to set up the environment, and then start to read from the body of Chapter 1. 
 
.. 在正式进行实验之前，请先按照第零章章末的 :doc:`/chapter0/5setup-devel-env` 中的说明完成环境配置，再从第一章开始阅读正文。

.. chyyuu 如果已经对 RISC-V 架构、Rust 语言和内核的知识有较多了解，第零章章末的 :doc:`/chapter0/6hardware` 提供了我们采用的真实硬件平台 Kendryte K210 的一些信息。


Collaboration on Project

.. 项目协作

----------------------

- :doc:`/setup-sphinx` introduces how to configure the document development environment according to Sphinx framework configurations, in order to locally build and generate documents in html and other formats. 
- :doc:`/rest-example` lays out the basic grammar and examples of ReStructuredText, used for ths document. 
- `The Project Source Code Repository <https://github.com/rcore-os/rCore-Tutorial-v3>`_ && `The Document Repository <https://github.com/rcore-os/rCore-Tutorial-Book-v3>`_
- This project still has many imperfections as time is tight. Welcome every one's feedback in the comment section of each chapter, or submit Issues and Pull Requests, let's work together to better this book.
- Welcome to join the QQ chat group, ID:735045051

.. - :doc:`/setup-sphinx` 介绍了如何基于 Sphinx 框架配置文档开发环境，之后可以本地构建并渲染 html 或其他格式的文档；
.. - :doc:`/rest-example` 给出了目前编写文档才用的 ReStructuredText 标记语言的一些基础语法及用例；
.. - `项目的源代码仓库 <https://github.com/rcore-os/rCore-Tutorial-v3>`_ && `文档仓库 <https://github.com/rcore-os/rCore-Tutorial-Book-v3>`_
.. - 时间仓促，本项目还有很多不完善之处，欢迎大家积极在每一个章节的评论区留言，或者提交 Issues 或 Pull Requests，让我们一起努力让这本书变得更好！
.. - 欢迎大家加入项目交流 QQ 群，群号：735045051

.. 项目进度

Project Progress
-----------------------

- 2020-11-03: Finish setting up environment; start on documents. 
- 2020-11-13: Finish Chapter 1.
- 2020-11-27: Finish Chapter 2.
- 2020-12-20: Finish Codes for first seven chapters.
- 2021-01-10: Finish Chapter 3. 
- 2021-01-18: Add Chapter 0. 
- 2021-01-30: Finish Chapter 4. 
- 2021-02-16: Finish Chapter 5. 
- 2021-02-20: Finish Chapter 6. 
- 2021-03-06: Finish Chapter 7. Up until then, finish the first version draft. 
- 2021-10-20: Finish Codes for Chapter 8 previously. Start to update earlier chapters and finish Chapter 8 document. 
- 2021-11-20: Update Chapter 1~9, add new Chapter 8 (Synchronization and Mutex), and shift the previous Chapter 8 (External Device) to Chapter 9. 
- 2022-01-02: Finish editing Chapter 1.
- 2022-01-05: Finish editing Chapter 2.
- 2022-01-06: Finish editing Chapter 3. 
- 2022-01-07: Finish editing Chapter 4.
- 2022-01-09: Finish editing Chapter 5. 