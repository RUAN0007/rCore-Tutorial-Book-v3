.. 实验环境配置

Experimental Environment Setup
===================================

.. toctree::
   :hidden:
   :maxdepth: 4
   
In this section we will complete the environment setup, in order to run rCore-Tutorial-v3 successfully. The whole process is divided into the following parts:

- System environment setup
- C/Rust development environment setup
- QEMU emulator installation
- Additional tool installation
- Run rCore-Tutorial-v3

.. 本节我们将完成环境配置并成功运行 rCore-Tutorial-v3 。整个流程分为下面几个部分：

.. - 系统环境配置
.. - C/Rust 开发环境配置
.. - QEMU 模拟器安装
.. - 其他工具安装
.. - 运行 rCore-Tutorial-v3

.. 在线开发环境配置

Online Development Environment Configuration
-------------------------------------------------

.. Github Classroom方式进行在线OS 环境配置

Setup Online OS Environment with Github Classroom
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note: This is currently mainly used for `2022 Open Source Operating System Training Camp <https://learningos.github.io/rust-based-os-comp2022/>`_

.. 注：这种方式目前主要用于 `2022年开源操作系统训练营 <https://learningos.github.io/rust-based-os-comp2022/>`_ 

.. note::

   **Online Development based on Github Classroom**
   
   Based on github classroom, it is convenient to establish a git repository for development, and can be developed and used online based on github's codespace (online version ubuntu +vscode). The entire development environment requires only a web browser.

   1. Log in to github.com with your github id in a web browser
   2. Receive the github classroom online invitation `the first experimental exercise setup-env-run-os1 <https://classroom.github.com/a/hnoWuKGF>`_, and select OK all the way according to the prompts.
   3. After completing the second step, the github repository of your first experiment exercise setup-env-run-os1 will be automatically created. Click the link of this github repository to see the first experiment you want to complete up.
   4. You can see a striking `code` green button in the upper middle of the webpage of your first lab exercise. After clicking, you can further see the `codespace` label and the eye-catching `create codesapce on main` green button. Please click this green button to enter the online ubuntu +vscode environment
   5. Follow the environment installation prompts below to install and configure the development environment in the `console` of vscode: rustc, qemu and other tools. Note: You can also execute ``make codespaces_setenv`` in the `console` of vscode to automatically install and configure the development environment (the execution of ``sudo`` requires root privileges and only needs to be executed once).
   6. **Important:** Execute `make setupclassroom_testX` in the `console` of vscode (this command is only executed once, and the range of X is 1-8) to configure the automatic scoring function of github classroom.
   7. Then the complete experimental process of development, operation and submission can be carried out based on online vscode.

   The above steps 3, 4, and 5 are not necessary, you can also just generate a git repository based on ``Github Classroom``, and develop locally.

   .. **基于 github classroom 的在线开发方式**
   
   .. 基于 github classroom，可方便建立开发用的 git repository，并可基于 github 的 codespace （在线版 ubuntu +vscode）在线开发使用。整个开发环境仅仅需要一个网络浏览器。

   .. 1. 在网络浏览器中用自己的 github  id 登录 github.com
   .. 2. 接收 `第一个实验练习 setup-env-run-os1 的github classroom在线邀请 <https://classroom.github.com/a/hnoWuKGF>`_  ，根据提示一路选择OK即可。
   .. 3. 完成第二步后，你的第一个实验练习 setup-env-run-os1 的 github repository 会被自动建立好，点击此 github repository 的链接，就可看到你要完成的第一个实验了。
   .. 4. 在你的第一个实验练习的网页的中上部可以看到一个醒目的 `code`  绿色按钮，点击后，可以进一步看到  `codespace` 标签和醒目的 `create codesapce on main` 绿色按钮。请点击这个绿色按钮，就可以进入到在线的ubuntu +vscode环境中
   .. 5. 再按照下面的环境安装提示在 vscode 的 `console` 中安装配置开发环境：rustc，qemu 等工具。注：也可在 vscode 的 `console` 中执行 ``make codespaces_setenv`` 来自动安装配置开发环境（执行 ``sudo`` 需要root权限，仅需要执行一次）。
   .. 6. **重要：** 在 vscode 的 `console` 中执行 `make setupclassroom_testX`  （该命令仅执行一次，X的范围为 1-8）配置 github classroom 自动评分功能。
   .. 7. 然后就可以基于在线 vscode 进行开发、运行、提交等完整的实验过程了。

   .. 上述的3，4，5步不是必须的，你也可以仅仅基于 ``Github Classroom`` 生成 git repository，并进行本地开发。


.. 本地操作系统开发环境配置

Setup Local OS Development Environment
-------------------------------------------------

At present, the experimental can be supported on `Ubuntu operating system <https://cdimage.ubuntu.com/releases/>`_, `openEuler operating system <https://repo.openeuler.org/openEuler-20.03-LTS-SP2/ ISO/>`_, `Dragon Lizard Operating System <https://openanolis.cn/anolisos>`_, etc. For users on Windows10/11 and macOS, you can use WSL2, VMware Workstation or VirtualBox and other related software to install Ubuntu18.04 / 20.04, openEuler operating system, Dragon Lizard operating system, etc. through virtual machines, and conduct experiments on them.


.. 目前实验内容可支持在 `Ubuntu操作系统 <https://cdimage.ubuntu.com/releases/>`_ 、 `openEuler操作系统 <https://repo.openeuler.org/openEuler-20.03-LTS-SP2/ISO/>`_ 、 `龙蜥操作系统 <https://openanolis.cn/anolisos>`_ 等上面进行操作。对于 Windows10/11 和 macOS 上的用户，可以使用WSL2、VMware Workstation 或 VirtualBox 等相关软件，通过虚拟机方式安装 Ubuntu18.04 / 20.04、openEuler操作系统、龙蜥操作系统等，并在上面进行实验。

.. Windows的WSL2方式建立Linux环境

Setup Linux Environment using Windows WSL2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For Windows10/11 users, you can install Ubuntu 18.04 / 20.04 through the system's built-in WSL2 virtual machine (please do not use WSL1). Proceed as follows:

- Upgrade Windows 10/11 to the latest version (Windows 10 version 18917 or later builds). Note that if
  Not Windows 10/11 Professional Edition, may need manual update, download from Microsoft official website. After upgrading,
  You can enter the ``winver`` command in PowerShell to see the build number.
- Select "Dev developer mode" at "Windows Settings > Update and Security > Windows Insider Experience Program".
- Open a PowerShell terminal as administrator and enter the following command:

  .. code-block::

     # Enable Windows functionality: "Windows Subsystem for Linux"
     >> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

     # Enable Windows functionality: "Installed System Virtual Machine Platforms"
     >> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

     # <Distro> is changed to correspond to the Linux version installed from the Microsoft Store, for example: `wsl --set-version Ubuntu 2`
     # Skip this step if you didn't install any Linux version from Microsoft Store beforehand
     >> wsl --set-version <Distro> 2

     # Set the default to WSL 2, if the Windows version is not up-to-date, this command will report an error
     >> wsl --set-default-version 2

- `Download the Linux kernel installation package <https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package>`_
- Search and install Ubuntu18.04 / 20.04 in Microsoft Store. 


.. 对于Windows10/11 的用户可以通过系统内置的 WSL2 虚拟机（请不要使用 WSL1）来安装 Ubuntu 18.04 / 20.04 。步骤如下：

.. - 升级 Windows 10/11 到最新版（Windows 10 版本 18917 或以后的内部版本）。注意，如果
..   不是 Windows 10/11 专业版，可能需要手动更新，在微软官网上下载。升级之后，
..   可以在 PowerShell 中输入 ``winver`` 命令来查看内部版本号。
.. - 「Windows 设置 > 更新和安全 > Windows 预览体验计划」处选择加入 “Dev 开发者模式”。
.. - 以管理员身份打开 PowerShell 终端并输入以下命令：

  .. code-block::

..      # 启用 Windows 功能：“适用于 Linux 的 Windows 子系统”
..      >> dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

..      # 启用 Windows 功能：“已安装的系统虚拟机平台”
..      >> dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

..      # <Distro> 改为对应从微软应用商店安装的 Linux 版本名，比如：`wsl --set-version Ubuntu 2`
..      # 如果你没有提前从微软应用商店安装任何 Linux 版本，请跳过此步骤
..      >> wsl --set-version <Distro> 2

..      # 设置默认为 WSL 2，如果 Windows 版本不够，这条命令会出错
..      >> wsl --set-default-version 2

.. -  `下载 Linux 内核安装包 <https://docs.microsoft.com/zh-cn/windows/wsl/install-win10#step-4---download-the-linux-kernel-update-package>`_
.. -  在微软商店（Microsoft Store）中搜索并安装 Ubuntu18.04 / 20.04。


.. VMware虚拟机方式进行本地OS开发环境配置

Setup Local Development Environment using VMware Virtual Machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you plan to use VMware to install a virtual machine, we have already configured an Ubuntu22.04 image that can directly run rCore-Tutorial-v3. It is a virtual disk file in ``vmdk`` format. You only need to create a new one in VMware A virtual machine, just select it when setting up the virtual disk. `Baidu network disk link <https://pan.baidu.com/s/1yQHtQIXQUbHCbyqSPtuqqQ?pwd=pcxf>`_ or `Tsinghua cloud disk link <https://cloud.tsinghua.edu.cn/d/a9b7b0a1b4724c3f9c66/> `_ (Currently a mirror of the old version of Ubuntu18.04+QEMU5.0). The user oslab has been created, and the password is a space. It has installed the Chinese input method and Visual Studio Code as the Rust integrated development environment, which makes it easier to complete the experiment and write the experiment report. If you want to use VMWare to install the openEuler virtual machine, you can download the ISO from the `openEuler official website <https://repo.openeuler.org/openEuler-20.03-LTS-SP2/ISO/>`_ and install it yourself, then you need to refer to the Internet Some tutorials for configuring the network and installing the GUI.

.. 如果你打算使用 VMware 安装虚拟机的话，我们已经配置好了一个能直接运行 rCore-Tutorial-v3 的 Ubuntu22.04 镜像，它是一个 ``vmdk`` 格式的虚拟磁盘文件，只需要在 VMware 中新建一台虚拟机，在设置虚拟磁盘的时候选择它即可。`百度网盘链接 <https://pan.baidu.com/s/1yQHtQIXQUbHCbyqSPtuqqQ?pwd=pcxf>`_ 或者 `清华云盘链接 <https://cloud.tsinghua.edu.cn/d/a9b7b0a1b4724c3f9c66/>`_ （目前是旧版的 Ubuntu18.04+QEMU5.0的镜像）。已经创建好用户 oslab ，密码为一个空格。它已经安装了中文输入法和作为 Rust 集成开发环境的 Visual Studio Code，能够更容易完成实验并撰写实验报告。如果想要使用 VMWare 安装 openEuler 虚拟机的话，可以在 `openEuler官网 <https://repo.openeuler.org/openEuler-20.03-LTS-SP2/ISO/>`_ 下载 ISO 自行安装，接着需要参考网络上的一些教程配置网络和安装图形界面。

.. _link-docker-env:

.. Docker方式进行本地OS开发环境配置

Setup Local Development Environment using Docker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

   **Docker development environment**

   Thanks to qobilidop, dinghao188 and Mr. Zhang Handong for helping to configure the Docker development environment. After entering the Docker development environment, there is no need to install and configure any software tool chain. You can run the tutorial directly. Currently, it should only support running the tutorial on the QEMU emulator.
   
   The usage method is as follows (take Ubuntu18.04 as an example):

   1. Switch to the administrator account ``root`` through ``su`` (Note: If you have not set the password of the ``root`` account before, you need to set it through ``sudo passwd``), in ` Under the rCore-Tutorial-v3`` root directory, execute ``make build_docker`` to build a docker-based development environment;
   2. In the ``rCore-Tutorial-v3`` root directory, execute ``make docker`` to enter the Docker environment;
   3. After entering Docker, you will find that you are currently in the root directory ``/``, we switch the current working path to the ``/mnt`` directory through ``cd mnt``;
   4. Through ``ls``, you can find that the contents of the ``/mnt`` directory are exactly the same as the contents of the ``rCore-Tutorial-v3`` directory, and then you can run the tutorial in this environment. For example ``cd os && make run``.

   .. **Docker 开发环境**

   .. 感谢 qobilidop，dinghao188 和张汉东老师帮忙配置好的 Docker 开发环境，进入 Docker 开发环境之后不需要任何软件工具链的安装和配置，可以直接将 tutorial 运行起来，目前应该仅支持将 tutorial 运行在 QEMU 模拟器上。
   
   .. 使用方法如下（以 Ubuntu18.04 为例）：

   .. 1. 通过 ``su`` 切换到管理员账户 ``root`` （注：如果此前并未设置 ``root`` 账户的密码需要先通过 ``sudo passwd`` 进行设置）, 在 ``rCore-Tutorial-v3`` 根目录下，执行 ``make build_docker`` ，来建立基于docker的开发环境 ；
   .. 2. 在 ``rCore-Tutorial-v3`` 根目录下，执行 ``make docker`` 进入到 Docker 环境；
   .. 3. 进入 Docker 之后，会发现当前处于根目录 ``/`` ，我们通过 ``cd mnt`` 将当前工作路径切换到 ``/mnt`` 目录；
   .. 4. 通过 ``ls`` 可以发现 ``/mnt`` 目录下的内容和 ``rCore-Tutorial-v3`` 目录下的内容完全相同，接下来就可以在这个环境下运行 tutorial 了。例如 ``cd os && make run`` 。    


.. chyyuu 下面的说法是面向github网页的，在书中应该有改变???

You can also experiment on Windows10/11 or macOS native systems or other Linux distributions, basically there will be no major problems. However, due to time issues, we mainly tested on Ubuntu18.04 on x86-64, and the subsequent configurations are also based on it. If you encounter any problems, please leave a message in the discussion area of ​​this section, and we will try our best to help you address them. 

.. 你也可以在 Windows10/11 或 macOS 原生系统或者其他 Linux 发行版上进行实验，基本上不会出现太大的问题。不过由于时间问题我们主要在 Ubuntu18.04 on x86-64上进行了测试，后面的配置也都是基于它的。如果遇到了问题的话，请在本节的讨论区中留言，我们会尽量帮助解决。

.. 手动进行本地OS开发环境配置

Setup Local Development Environment Manually
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. 基于RISC-V硬件环境的配置

Configure RISC-V-based Hardware Environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

At present, there have been Linux system environments that can be used in RISC-V 64 (RV64 for short) hardware simulation environment (such as QEMU with RV64) and real physical environment (such as Quanzhi Nezha D1 development board, SiFive U740 development board). However, compared with Linux x86-64, although Linux RV64 is quite novel, it is not mature enough. There are relatively few application software packages that have been adapted and precompiled, so it is suitable for hackers to try. If students are interested, we also provide a variety of corresponding hardware simulation environments and real physical environment Linux for RV64 distributions, so that such students can conduct experiments:

- `QEMU for Ubuntu for RV64 and SiFive U740 development board system image <https://cdimage.ubuntu.com/releases/20.04.3/release/ubuntu-20.04.3-preinstalled-server-riscv64+unmatched.img.xz>`_
- `QEMU system image of OpenEuler for RV64 <https://repo.openeuler.org/openEuler-preview/RISC-V/Image/openEuler-preview.riscv64.qcow2>`_
- `Debian for RV64 D1 Nezha development board system image <http://www.perfxlab.cn:8080/rvboards/RVBoards_D1_Debian_img_v0.6.1/RVBoards_D1_Debian_img_v0.6.1.zip>`_

.. 目前已经出现了可以在RISC-V 64（简称RV64）的硬件模拟环境（比如QEMU with RV64）和真实物理环境（如全志哪吒D1开发板、SiFive U740开发板）的Linux系统环境。但Linux RV64相对于Linux x86-64而言，虽然挺新颖的，但还不够成熟，已经适配和预编译好的应用软件包相对少一些，适合hacker进行尝试。如果同学有兴趣，我们也给出多种相应的硬件模拟环境和真实物理环境的Linux for RV64发行版，以便于这类同学进行实验：

.. - `Ubuntu for RV64的QEMU和SiFive U740开发板系统镜像 <https://cdimage.ubuntu.com/releases/20.04.3/release/ubuntu-20.04.3-preinstalled-server-riscv64+unmatched.img.xz>`_
.. - `OpenEuler for RV64的QEMU系统镜像 <https://repo.openeuler.org/openEuler-preview/RISC-V/Image/openEuler-preview.riscv64.qcow2>`_
.. - `Debian for RV64的D1哪吒开发板系统镜像 <http://www.perfxlab.cn:8080/rvboards/RVBoards_D1_Debian_img_v0.6.1/RVBoards_D1_Debian_img_v0.6.1.zip>`_

Note: The subsequent configuration is mainly based on the Linux for x86-64 system environment. If students use the Linux for RV64 environment, they need to configure it by themselves. However, if students are familiar with it, the configuration method is similar and simpler. The main problem that may exist is that the relevant software packages for Linux for RV64 may not be complete, so students need to compile the missing software packages directly from the source code.

.. 注：后续的配置主要基于Linux for x86-64系统环境，如果同学采用Linux for RV64环境，需要自己配置。不过在同学比较熟悉的情况下，配置方法类似且更加简单。可能存在的主要问题是，面向Linux for RV64的相关软件包可能不全，这样需要同学从源码直接编译出缺失的软件包。

.. C 开发环境配置
 
Setup C Development Environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

During the experiment or practice, part of the development based on C language will also be involved, and the basic local development environment and cross-development environment can be installed. The following takes Ubuntu 20.04 as an example. Below are the softwares involved in the C development environment that needs to be installed:


.. 在实验或练习过程中，也会涉及部分基于C语言的开发，可以安装基本的本机开发环境和交叉开发环境。下面是以Ubuntu 20.04为例，需要安装的C 开发环境涉及的软件：

.. code-block:: bash

   $ sudo apt-get update && sudo apt-get upgrade
   $ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu

Note: The above software is not required for the Rust development environment. And the QEMU software version of ubuntu 20.04 is low. The experiments in this book need to install QEMU version 7.0 or higher.

.. 注：上述软件不是Rust开发环境所必须的。且ubuntu 20.04的QEMU软件版本低，而本书实验需要安装7.0以上版本的QEMU。

.. Rust 开发环境配置

Setup Rust Development Environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. 首先安装 Rust 版本管理器 rustup 和 Rust 包管理器 cargo，这里我们用官方的安装脚本来安装：

First install the Rust version manager rustup and the Rust package manager cargo, here we use the official installation script to install:

.. code-block:: bash

   curl https://sh.rustup.rs -sSf | sh

If the official script download fails, you can enter `<https://sh.rustup.rs>`_ in the address bar of the browser to download the script and run it locally.

If the official script has a problem of slow network speed during operation, it can optionally be accelerated by modifying the mirror address of rustup (modified to the mirror server of the University of Science and Technology of China):

.. 如果通过官方的脚本下载失败了，可以在浏览器的地址栏中输入 `<https://sh.rustup.rs>`_ 来下载脚本，在本地运行即可。

.. 如果官方的脚本在运行时出现了网络速度较慢的问题，可选地可以通过修改 rustup 的镜像地址（修改为中国科学技术大学的镜像服务器）来加速：

.. code-block:: bash
   
   export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
   export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
   curl https://sh.rustup.rs -sSf | sh


Or use tuna sources to speed up `see rustup help <https://mirrors.tuna.tsinghua.edu.cn/help/rustup/>`_:

.. 或者使用tuna源来加速 `参见 rustup 帮助 <https://mirrors.tuna.tsinghua.edu.cn/help/rustup/>`_：

.. code-block:: bash
   
   export RUSTUP_DIST_SERVER=https://mirrors.tuna.edu.cn/rustup
   export RUSTUP_UPDATE_ROOT=https://mirrors.tuna.edu.cn/rustup/rustup
   curl https://sh.rustup.rs -sSf | sh

Or it can also be achieved by setting the "scientific Internet proxy" in the command line before running:

.. 或者也可以通过在运行前设置命令行中的科学上网代理来实现：

.. code-block:: bash 

   # e.g. Shadowsocks 代理，请根据自身配置灵活调整下面的链接
   export https_proxy=http://127.0.0.1:1080
   export http_proxy=http://127.0.0.1:1080
   export ftp_proxy=http://127.0.0.1:1080

After the installation is complete, we can reopen a terminal for the previously set environment variables to take effect. We can also manually apply the environment variable settings to the current terminal, just enter the following command:

.. 安装完成后，我们可以重新打开一个终端来让之前设置的环境变量生效。我们也可以手动将环境变量设置应用到当前终端，只需要输入以下命令：

.. code-block:: bash

   source $HOME/.cargo/env


Next, we can confirm that we installed the Rust toolchain correctly:

.. 接下来，我们可以确认一下我们正确安装了 Rust 工具链：

.. code-block:: bash
   
   rustc --version

You can see the version of the currently installed toolchain.

.. 可以看到当前安装的工具链的版本。

.. code-block:: bash

   rustc 1.62.0-nightly (1f7fb6413 2022-04-10)

.. warning::
   The current version of the rustc compiler used for operating system experimental development is not limited to numbers like 1.46.0, you can choose a newer version of the rustc compiler. But note that only the nightly version of rustc can be used.

   .. 目前用于操作系统实验开发的 rustc 编译器的版本不局限在 1.46.0 这样的数字上，你可以选择更新版本的 rustc 编译器。但注意只能用 rustc 的 nightly 类型的版本。


You can install the nightly version of rustc with the following command, and set this version as the default version of rustc.

.. 可通过如下命令安装 rustc 的 nightly 版本，并把该版本设置为 rustc 的缺省版本。

.. code-block:: bash
   
   rustup install nightly
   rustup default nightly


We'd better replace the package mirror address crates.io used by the package manager cargo with the mirror server of the University of Science and Technology of China to speed up the download of the three-party library. We open (or create if not) the ``~/.cargo/config`` file and modify the content to:


.. 我们最好把软件包管理器 cargo 所用的软件包镜像地址 crates.io 也换成中国科学技术大学的镜像服务器来加速三方库的下载。我们打开（如果没有就新建） ``~/.cargo/config`` 文件，并把内容修改为：

.. code-block:: toml

   [source.crates-io]
   registry = "https://github.com/rust-lang/crates.io-index"
   replace-with = 'ustc'
   [source.ustc]
   registry = "git://mirrors.ustc.edu.cn/crates.io-index"

Similarly, you can also use the tuna source `see crates.io help <https://mirrors.tuna.tsinghua.edu.cn/help/crates.io-index.git/>`_:

.. 同样，也可以使用tuna源 `参见 crates.io 帮助 <https://mirrors.tuna.tsinghua.edu.cn/help/crates.io-index.git/>`_：

.. code-block:: toml

   [source.crates-io]
   replace-with = 'tuna'

   [source.tuna]
   registry = "https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git"

Next install some Rust related packages

.. 接下来安装一些Rust相关的软件包

.. code-block:: bash

   rustup target add riscv64gc-unknown-none-elf
   cargo install cargo-binutils
   rustup component add llvm-tools-preview
   rustup component add rust-src

.. warning::
   If you change to another rustc compiler (must be the nightly version), you need to reinstall the above required packages for rustc. The ``Makefile`` in the rCore-Tutorial repository contains the installation of these tools, if you use ``make run`` you can also install them manually.

   .. 如果你换了另外一个rustc编译器（必须是nightly版的），需要重新安装上述rustc所需软件包。rCore-Tutorial 仓库中的 ``Makefile`` 包含了这些工具的安装，如果你使用 ``make run`` 也可以不手动安装。

As for the Rust development environment, we recommend JetBrains Clion + Rust plugin or Visual Studio Code with rust-analyzer and RISC-V Support plugin.

.. 至于 Rust 开发环境，推荐 JetBrains Clion + Rust插件 或者 Visual Studio Code 搭配 rust-analyzer 和 RISC-V Support 插件。

.. note::

   * JetBrains Clion is a paid commercial software, but for students and teachers, as long as they register an account on the JetBrains website, they can enjoy the benefits of free use for a certain period (about half a year).
   * Visual Studio Code is open source software and is free to use.
   * Of course, it is no problem to use traditional editors such as VIM and Emacs.

   .. * JetBrains Clion是付费商业软件，但对于学生和教师，只要在 JetBrains 网站注册账号，可以享受一定期限（半年左右）的免费使用的福利。
   .. * Visual Studio Code 是开源软件，不用付费就可使用。
   .. * 当然，采用 VIM，Emacs 等传统的编辑器也是没有问题的。

.. QEMU 模拟器安装

Install QEMU Emulator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We need to use the QEMU 7.0 version for experiments, and the lower version of QEMU may cause the framework code not to run properly. And the QEMU version in the default software source of the package manager of many Linux distributions is too low, so we need to manually compile and install the QEMU emulator software from the source code. The following is an example of the installation process on Ubuntu 18.04/20.04:


.. 我们需要使用 QEMU 7.0 版本进行实验，低版本的 QEMU 可能导致框架代码不能正常运行。而很多 Linux 发行版的软件包管理器默认软件源中的 QEMU 版本过低，因此我们需要从源码手动编译安装 QEMU 模拟器软件。下面以 Ubuntu 18.04/20.04 上的安装流程为例进行说明：

.. chyyuu warning::

   Note that if you have installed QEMU 6.0+, you need to update the bootloader (that is, RustSBI) in the project directory to the latest 0.2.0-alpha.6 version. They are currently available in the ``chX-dev`` branch.

.. 注意，如果安装了 QEMU 6.0+ 版本，则目前需要将项目目录下的 bootloader（也即 RustSBI）更新为最新的 0.2.0-alpha.6 版本。它们目前可以在 ``chX-dev`` 分支中找到。


First we install the dependencies, get the QEMU source code and compile it manually:

.. 首先我们安装依赖包，获取 QEMU 源代码并手动编译：

.. code-block:: bash

   # Install and Compiled Required Dependency Packages
   sudo apt install autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev \
                 gawk build-essential bison flex texinfo gperf libtool patchutils bc \
                 zlib1g-dev libexpat-dev pkg-config  libglib2.0-dev libpixman-1-dev libsdl2-dev \
                 git tmux python3 python3-pip ninja-build 

   # Download Source Code Packages
   # If the download speed is too slow, you can use the Baidu network disk link we provide:https://pan.baidu.com/s/1dykndFzY73nqkPL2QXs32Q 
   # Extraction code：jimc 
   wget https://download.qemu.org/qemu-7.0.0.tar.xz
   # uncompress
   tar xvJf qemu-7.0.0.tar.xz
   # Compile, Install and Configure for RISV-V Support
   cd qemu-7.0.0
   ./configure --target-list=riscv64-softmmu,riscv64-linux-user  # If you want to support GUI, you can add " --enable-sdl" parameter
   make -j$(nproc)

.. note::
   
   Note that the above dependencies may not be complete, such as on Ubuntu 18.04:

   - When ``ERROR: pkg-config binary 'pkg-config' not found`` appears, you can install the ``pkg-config`` package;
   - When ``ERROR: glib-2.48 gthread-2.0 is required to compile QEMU`` appears, you can install
     ``libglib2.0-dev`` package;
   - When ``ERROR: pixman >= 0.21.8 not present`` appears, you can install the ``libpixman-1-dev`` package.

   .. 注意，上面的依赖包可能并不完全，比如在 Ubuntu 18.04 上：

   .. - 出现 ``ERROR: pkg-config binary 'pkg-config' not found`` 时，可以安装 ``pkg-config`` 包；
   .. - 出现 ``ERROR: glib-2.48 gthread-2.0 is required to compile QEMU`` 时，可以安装 
   ..   ``libglib2.0-dev`` 包；
   .. - 出现 ``ERROR: pixman >= 0.21.8 not present`` 时，可以安装 ``libpixman-1-dev`` 包。

   Some other Linux distributions dependencies to compile QEMU can be found from `here <https://risc-v-getting-started-guide.readthedocs.io/en/latest/linux-qemu.html#prerequisites>`_.

   .. 另外一些 Linux 发行版编译 QEMU 的依赖包可以从 `这里 <https://risc-v-getting-started-guide.readthedocs.io/en/latest/linux-qemu.html#prerequisites>`_ 找到。

After that, we can install QEMU to the ``/usr/local/bin`` directory with ``sudo make install`` in the same directory, but this often causes conflicts. Personally, it is more customary to edit the ``~/.bashrc`` file (if you are using the default ``bash`` terminal), and add a few lines at the end of the file:

.. 之后我们可以在同目录下 ``sudo make install`` 将 QEMU 安装到 ``/usr/local/bin`` 目录下，但这样经常会引起冲突。个人来说更习惯的做法是，编辑 ``~/.bashrc`` 文件（如果使用的是默认的 ``bash`` 终端），在文件的末尾加入几行：

.. code-block:: bash

   # Please note that the parent directory of qemu-7.0.0 can be flexibly adjusted according to your actual installation location
   export PATH=$PATH:/path/to/qemu-7.0.0/build

Then you can update the system path in the current terminal ``source ~/.bashrc``, or restart a new terminal directly.

At this point we can confirm the version of QEMU:

.. 随后即可在当前终端 ``source ~/.bashrc`` 更新系统路径，或者直接重启一个新的终端。

.. 此时我们可以确认 QEMU 的版本：

.. code-block:: bash

   qemu-system-riscv64 --version
   qemu-riscv64 --version

On other Linux x86-64 environments that lack precompiled QEMU with RV64 packages (such as the openEuler operating system), you first need the `riscv branch <https://gitee.com/src-openeuler/qemu/tree/riscv/>`_ of QEMU maintained by the openEuler community to download QEMU source code and build it directly through rpmbuild.

.. 在其他缺少预编译 QEMU with RV64 软件包的Linux x86-64 环境（如openEuler操作系统）上，首先需要从 openEuler 社区维护的 QEMU 的 `riscv分支 <https://gitee.com/src-openeuler/qemu/tree/riscv/>`_ 下载 QEMU 源码，并直接通过 rpmbuild 进行构建。

.. warning::


   Please try not to install ``qemu-kvm``, it may cause our framework not to run properly. If you already have it installed, consider switching to Docker.

   Also, we only tested on Qemu version 7.0.0, please try not to switch to other versions.

   .. 请尽量不要安装 ``qemu-kvm`` ，这可能会导致我们的框架无法正常运行。如果已经安装，可以考虑换用 Docker 。

   .. 另外，我们仅在 Qemu 7.0.0 版本上进行了测试，请尽量不要切换到其他版本。

.. K210 真机串口通信

K210 Real Machine Serial Port Communication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to run Tutorial on the K210 real machine, we also need to install a Python-based serial port communication library and a simple serial port terminal.

.. 为了能在 K210 真机上运行 Tutorial，我们还需要安装基于 Python 的串口通信库和简易的串口终端。

.. code-block:: bash

   pip3 install pyserial
   sudo apt install python3-serial

.. GDB 调试支持

Debug in GDB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently we only support debugging based on the QEMU emulator. In the ``os`` directory ``make debug`` can debug our kernel, which requires the installation of the terminal multiplexing tool ``tmux``, and the gdb debugger ``riscv64-unknown-elf that supports the riscv64 instruction set -gdb``. The debugger is included in the riscv64 gcc toolchain, and the precompiled version of the toolchain can be downloaded at the following link:

.. 目前我们仅支持基于 QEMU 模拟器进行调试。在 ``os`` 目录下 ``make debug`` 可以调试我们的内核，这需要安装终端复用工具 ``tmux`` ，还需要支持 riscv64 指令集的 gdb 调试器 ``riscv64-unknown-elf-gdb`` 。该调试器包含在 riscv64 gcc 工具链中，工具链的预编译版本可以在如下链接处下载：

- `Ubuntu Environment <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-linux-ubuntu14.tar.gz>`_
- `macOS Environment <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-apple-darwin.tar.gz>`_
- `Windows Environment <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-w64-mingw32.zip>`_
- `CentOS Environment <https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2020.04.1-x86_64-linux-centos6.tar.gz>`_

The latest version of the toolchain can be found in `sifive official repo <https://github.com/sifive/freedom-tools/releases>`_.

After decompression, you can find ``riscv64-unknown-elf-gdb`` and some other common tools ``objcopy/objdump/readelf`` in the ``bin`` directory.

On other Linux x86-64 environments that lack the precompiled riscv64 gcc toolchain (such as openEuler operating system, dragon lizard operating system, etc.), you need to clone `riscv toolchain repository <https://github.com/riscv-collab/riscv-gnu-toolchain>`_ and refer to its instructions to build manually.

For some reason, we build in ``release`` mode throughout. In order to debug normally, please make sure that the ``Cargo.toml`` of each project (such as ``os``, ``user`` and ``easy-fs``) contains the following configuration:

.. 最新版的工具链可以在 `sifive 官方的 repo 中 <https://github.com/sifive/freedom-tools/releases>`_ 找到。

.. 解压后在 ``bin`` 目录下即可找到 ``riscv64-unknown-elf-gdb`` 以及另外一些常用工具 ``objcopy/objdump/readelf`` 等。

.. 在其他缺少预编译 riscv64 gcc 工具链的 Linux x86-64 环境（如 openEuler 操作系统、龙蜥操作系统等）上，则需要 clone `riscv 工具链仓库 <https://github.com/riscv-collab/riscv-gnu-toolchain>`_ 并参考其说明手动构建。

.. 出于某些原因，我们全程使用 ``release`` 模式进行构建。为了正常进行调试，请确认各项目（如 ``os`` , ``user`` 和 ``easy-fs`` ）的 ``Cargo.toml`` 中包含如下配置：

.. code-block:: toml

   [profile.release]
   debug = true

In addition, refer to ``os/Makefile``, you can first open a terminal page ``make gdbserver`` to start QEMU, and then open another terminal page in the same directory ``make gdbclient`` to connect the GDB client to QEMU for debugging. We recommend using the `gdb-dashboard <https://github.com/cyrus-and/gdb-dashboard>`_ plugin, which can greatly improve the debugging experience. In the comment area of ​​this section, students have provided debugging methods based on various IDEs, which can also be referred to.

.. 此外，参考 ``os/Makefile`` ，还可以先打开一个终端页面 ``make gdbserver`` 启动 QEMU ，此后另开一个终端页面在同目录下 ``make gdbclient`` 将 GDB 客户端连接到 QEMU 进行调试。我们推荐使用 `gdb-dashboard <https://github.com/cyrus-and/gdb-dashboard>`_ 插件，可以大大提升调试体验。在本节的评论区已有同学提供了基于各种 IDE 的调试方法，也可参考。

.. 运行 rCore-Tutorial-v3

Run rCore-Tutorial-v3
------------------------------------------------------------

.. 在 QEMU 模拟器上运行

Run in QEMU Emulator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If it is running on the QEMU emulator, just ``make run`` in the ``os`` directory. After the kernel is loaded, you can see the currently available application. ``usertests`` packages a large part of it, so we can run it by typing its name into the terminal.

.. 如果是在 QEMU 模拟器上运行，只需在 ``os`` 目录下 ``make run`` 即可。在内核加载完毕之后，可以看到目前可以用的 应用程序。 ``usertests`` 打包了其中的很大一部分，所以我们可以运行它，只需输入在终端中输入它的名字即可。

.. image:: qemu-final.png

After that, you can first press ``Ctrl+a`` (that is: press and hold Ctrl first, then press and hold the lowercase letter a, and then release the two keys at the same time), and then press ``x`` to exit QEMU.

.. 之后，可以先按下 ``Ctrl+a`` （即：先按下 Ctrl 不松开，再按下小写字母 a 不放，随后同时将两个键松开） ，再按下 ``x`` 来退出 QEMU。

.. 在 K210 平台上运行

Run in K210 Platform
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


It is slightly more complicated if it is running on the K210 platform.

First, we need to insert the MicroSD into the PC to copy the file system image to it.

.. 如果是在 K210 平台上运行则略显复杂。

.. 首先，我们需要将 MicroSD 插入 PC 来将文件系统镜像拷贝上去。

.. image:: prepare-sd.png

.. warning::

   In ``os/Makefile`` we set by default that MicroSD can be accessed with device ``SDCARD=/dev/sdb`` in the current operating system. You can use the ``df -hT`` command to confirm which device MicroSD is in your environment, and make appropriate configurations for ``SDCARD`` in ``os/Makefile`` before ``make sdcard`` Modifications. Otherwise, this may cause data loss on **device /dev/sdb**!

Then, we insert the MicroSD into the K210 development board, connect the K210 development board to the PC, and then enter the ``os`` directory ``make run BOARD=k210`` to run rCore Tutorial on the K210 development board.

   .. 在 ``os/Makefile`` 中我们默认设置 MicroSD 在当前操作系统中可以用设备 ``SDCARD=/dev/sdb`` 访问。你可以使用 ``df -hT`` 命令来确认在你的环境中 MicroSD 是哪个设备，并在 ``make sdcard`` 之前对 ``os/Makefile`` 的 ``SDCARD`` 配置做出适当的修改。不然，这有可能导致 **设备 /dev/sdb 上数据丢失** ！
   
.. 随后，我们将 MicroSD 插入 K210 开发板，将 K210 开发板连接到 PC ，然后进入 ``os`` 目录 ``make run BOARD=k210`` 在 K210 开发板上跑 rCore Tutorial 。 

.. image:: k210-final.png

After that, you can press ``Ctrl+]`` to exit the serial terminal.

Since the ch1~ch5 branches of the tutorial do not have a file system yet, running these branches on the K210 does not require a MicroSD card and does not need to burn the file system image, just switch directly to the `os` directory and execute `make run BOARD=k210`.

At this point, congratulations, you have completed the configuration of the experimental environment, and you can start reading the main part of the tutorial!

.. 之后，可以按下 ``Ctrl+]`` 来退出串口终端。

.. 由于教程的 ch1~ch5 分支还没有文件系统，在 K210 上运行这些分支无需 MicroSD 卡也不需要进行文件系统镜像烧写工作，直接切换到 `os` 目录下 `make run BOARD=k210` 即可。

.. 到这里，恭喜你完成了实验环境的配置，可以开始阅读教程的正文部分了！

Q & A
----------------------------------------------------------------

When the code does not run, you can try:

- Whether the branch is synchronized with the corresponding branch of the original rCore-Tutorial-v3 repository (not the forked repository). Consider updating via ``git pull`` if out of sync. Note: This is because the version of Rust changes quickly, and if it is not updated in time, the code that used to work normally will not run.
- The ``rust-toolchain`` in the root directory of the project is very important, it represents the version of the Rust toolchain used by the entire project. Be sure to keep it consistent with the corresponding branch of the original repository.
- Is there a ``bootloader/`` directory where RustSBI is placed in the project root directory. If it does not exist, it can be obtained from each branch of the original repository.
- Use ``make clean`` or ``cargo clean`` to delete the build products in the ``os`` or ``user`` directory, and re-run ``make run``. Note: Such problems usually indicate that there is a bug in the framework's build script, and you can raise an issue.

If you suspect a network problem, you can check:

- Please follow the instructions in this section to install Rust and configure the crates.io image. It is often possible to resolve issues with Rust toolchain updates and downloads of libraries posted to crates.io.
- If you find that you are stuck when trying to download the following libraries from github, You can modify the ``Cargo.toml`` under the ``os`` and ``user`` directory to replace it with the image on gitee.  For example, replace:
   .. code-block:: toml

      riscv = { git = "https://github.com/rcore-os/riscv", features = ["inline-asm"] }
      virtio-drivers = { git = "https://github.com/rcore-os/virtio-drivers" }
      k210-pac = { git = "https://github.com/wyfcyx/k210-pac" }
      k210-hal = { git = "https://github.com/wyfcyx/k210-hal" }
      k210-soc = { git = "https://github.com/wyfcyx/k210-soc" }
   
  with：

   .. code-block:: toml

      riscv = { git = "https://gitee.com/rcore-os/riscv", features = ["inline-asm"] }
      virtio-drivers = { git = "https://gitee.com/rcore-os/virtio-drivers" }
      k210-pac = { git = "https://gitee.com/wyfcyx/k210-pac" }
      k210-hal = { git = "https://gitee.com/wyfcyx/k210-hal" }
      k210-soc = { git = "https://gitee.com/wyfcyx/k210-soc" }

.. 当代码跑不起来的时候，可以尝试：

.. - 分支是否与 rCore-Tutorial-v3 原版仓库（而非 fork 出来的仓库）的对应分支同步。如不同步的话考虑通过 ``git pull`` 进行更新。注：这是因为 Rust 的版本更迭较快，如不及时更新的话曾经能正常运行的代码也会无法运行。
.. - 项目根目录下的 ``rust-toolchain`` 非常重要，它代表整个项目采用的 Rust 工具链版本。请务必保持其与原版仓库对应分支一致。
.. - 项目根目录下是否存在放置 RustSBI 的 ``bootloader/`` 目录。如不存在的话可从原版仓库的各分支上获取。
.. - 通过 ``make clean`` 或者 ``cargo clean`` 删除 ``os`` 或 ``user`` 目录下的构建产物，并重新 ``make run`` 。注：出现这样的问题通常说明框架的构建脚本存在 bug，可以提 issue。

.. 如果怀疑遇到了网络问题，可以检查：

.. - 请按照本节说明进行 Rust 安装和 crates.io 镜像配置。通常情况下能够解决 Rust 工具链更新和下载已发布到 crates.io 上库的问题。
.. - 如果发现在试图从 github 上下载下述几个库的时候卡死，可以修改 ``os`` 和 ``user`` 目录下的 ``Cargo.toml`` 替换为 gitee 上的镜像。例如，将：
  
..    .. code-block:: toml

..       riscv = { git = "https://github.com/rcore-os/riscv", features = ["inline-asm"] }
..       virtio-drivers = { git = "https://github.com/rcore-os/virtio-drivers" }
..       k210-pac = { git = "https://github.com/wyfcyx/k210-pac" }
..       k210-hal = { git = "https://github.com/wyfcyx/k210-hal" }
..       k210-soc = { git = "https://github.com/wyfcyx/k210-soc" }
   
..   替换为：

..    .. code-block:: toml

..       riscv = { git = "https://gitee.com/rcore-os/riscv", features = ["inline-asm"] }
..       virtio-drivers = { git = "https://gitee.com/rcore-os/virtio-drivers" }
..       k210-pac = { git = "https://gitee.com/wyfcyx/k210-pac" }
..       k210-hal = { git = "https://gitee.com/wyfcyx/k210-hal" }
..       k210-soc = { git = "https://gitee.com/wyfcyx/k210-soc" }
