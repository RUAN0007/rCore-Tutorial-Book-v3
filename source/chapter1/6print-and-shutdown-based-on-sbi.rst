.. 基于 SBI 服务完成输出和关机

Output and shutdown based on SBI services
=================================================================

.. toctree::
   :hidden:
   :maxdepth: 5

.. 本节导读

Section Guide
------------------------------------

In this section, we will carry out the last step of building the "Trilobite" operating system, which is to print ``Hello world!`` on the screen and shut down based on the services provided by RustSBI. In fact, as a refinement of the :ref:`application execution environment <app-software-stack>` we mentioned earlier, RustSBI sits between the underlying hardware and the kernel, and is the underlying execution environment of our kernel. This section will mention the second responsibility of the execution environment in addition to initializing the upper-layer application: that is, to provide services when the upper-layer application is running. The code in this section involves many details of assembly and Rust, and it is not necessary to fully understand its meaning. The key point is to successfully run the kernel.

.. 本节我们将进行构建“三叶虫”操作系统的最后一个步骤，即基于 RustSBI 提供的服务完成在屏幕上打印 ``Hello world!`` 和关机操作。事实上，作为对我们之前提到的 :ref:`应用程序执行环境 <app-software-stack>` 的细化，RustSBI 介于底层硬件和内核之间，是我们内核的底层执行环境。本节将会提到执行环境除了为上层应用进行初始化的第二种职责：即在上层应用运行时提供服务。本节的代码涉及的汇编和 Rust 的细节较多，不必完全理解其含义，重点在于将内核成功运行起来。

使用 RustSBI 提供的服务

Use RustSBI-provided Services
------------------------------------------

Our previous understanding of RustSBI was limited to the fact that it initializes the environment when the computer starts up. And it hands over the control of the computer to the kernel. But in fact, as the execution environment of the kernel, it has another responsibility: to provide services for the kernel in response to the kernel's request when the kernel is running. When the kernel sends a request, the computer will be controlled by RustSBI to respond to the kernel's request. After the request is processed, the computer control will be returned to the kernel. Thinking from the perspective of memory layout, each layer of execution environment (or software stack) corresponds to a piece of code and data in memory. The transfer of control here refers to the process that the CPU shifts the execution of one layer of software to another layer.  This process is similar to a function call, but the kernel cannot request the services provided by RustSBI through a function call. It is because the kernel is not linked with RustSBI. We only use the executable file built by RustSBI. So the kernel knows nothing about RustSBI's symbols. In fact, the kernel needs to "call" RustSBI's services in another complicated way:

.. 之前我们对 RustSBI 的了解仅限于它会在计算机启动时进行它所负责的环境初始化工作，并将计算机控制权移交给内核。但实际上作为内核的执行环境，它还有另一项职责：即在内核运行时响应内核的请求为内核提供服务。当内核发出请求时，计算机会转由 RustSBI 控制来响应内核的请求，待请求处理完毕后，计算机控制权会被交还给内核。从内存布局的角度来思考，每一层执行环境（或称软件栈）都对应到内存中的一段代码和数据，这里的控制权转移指的是 CPU 从执行一层软件的代码到执行另一层软件的代码的过程。这个过程和函数调用比较像，但是内核无法通过函数调用来请求 RustSBI 提供的服务，这是因为内核并没有和 RustSBI 链接到一起，我们仅仅使用 RustSBI 构建后的可执行文件，因此内核对于 RustSBI 的符号一无所知。事实上，内核需要通过另一种复杂的方式来“调用” RustSBI 的服务：

.. _term-llvm-sbicall:

.. code-block:: rust
    :linenos:

    // os/src/main.rs
    mod sbi;

    // os/src/sbi.rs
    use core::arch::asm;
    #[inline(always)]
    fn sbi_call(which: usize, arg0: usize, arg1: usize, arg2: usize) -> usize {
        let mut ret;
        unsafe {
            asm!(
                "ecall",
                inlateout("x10") arg0 => ret,
                in("x11") arg1,
                in("x12") arg2,
                in("x17") which,
            );
        }
        ret
    }

We implement the relevant functions of the kernel and RustSBI communication in the submodule ``sbi``. So we need to add ``mod sbi`` in ``main.rs`` to add this submodule to our project. In ``os/src/sbi.rs``, we first pay attention to the function signature of ``sbi_call``, ``which`` indicates the type of service that requests RustSBI (RustSBI can provide many different types of services), ``arg0`` ~ ``arg2`` represent the 3 parameters passed to RustSBI, and RustSBI will return a value to the kernel after processing the request. This return value will also be returned by the ``sbi_call`` function. Although we don't quite understand the specific implementation of the function ``sbi_call``, so far we already know how to use it: just call it when we need to use the RustSBI service. Know


.. 我们将内核与 RustSBI 通信的相关功能实现在子模块 ``sbi`` 中，因此我们需要在 ``main.rs`` 中加入 ``mod sbi`` 将该子模块加入我们的项目。在 ``os/src/sbi.rs`` 中，我们首先关注 ``sbi_call`` 的函数签名， ``which`` 表示请求 RustSBI 的服务的类型（RustSBI 可以提供多种不同类型的服务）， ``arg0`` ~ ``arg2`` 表示传递给 RustSBI 的 3 个参数，而 RustSBI 在将请求处理完毕后，会给内核一个返回值，这个返回值也会被 ``sbi_call`` 函数返回。尽管我们还不太理解函数 ``sbi_call`` 的具体实现，但目前我们已经知道如何使用它了：当需要使用 RustSBI 服务的时候调用它就行了。

.. 在 ``sbi.rs`` 中我们定义 RustSBI 支持的服务类型常量，它们并未被完全用到：

In ``sbi.rs`` we define service type constants supported by RustSBI, which are not fully used:

.. code-block:: rust
    :linenos:

    // os/src/sbi.rs
    #![allow(unused)] // Please put this line at the beginning of the file
    const SBI_SET_TIMER: usize = 0;
    const SBI_CONSOLE_PUTCHAR: usize = 1;
    const SBI_CONSOLE_GETCHAR: usize = 2;
    const SBI_CLEAR_IPI: usize = 3;
    const SBI_SEND_IPI: usize = 4;
    const SBI_REMOTE_FENCE_I: usize = 5;
    const SBI_REMOTE_SFENCE_VMA: usize = 6;
    const SBI_REMOTE_SFENCE_VMA_ASID: usize = 7;
    const SBI_SHUTDOWN: usize = 8;

As the name implies, the service ``SBI_CONSOLE_PUTCHAR`` can be used to output a character on the screen. We encapsulate this functionality into the ``console_putchar`` function:

.. 如字面意思，服务 ``SBI_CONSOLE_PUTCHAR`` 可以用来在屏幕上输出一个字符。我们将这个功能封装成 ``console_putchar`` 函数：

.. code-block:: rust
    :linenos:

    // os/src/sbi.rs
    pub fn console_putchar(c: usize) {
        sbi_call(SBI_CONSOLE_PUTCHAR, c, 0, 0);
    }

Note that we don't use the return value of ``sbi_call``, since it doesn't matter. If students are interested, you can try calling ``console_putchar`` in ``rust_main`` to output ``OK`` on the screen. Then run it on Qemu, we can see the first log output by ourselves.

.. 注意我们并未使用 ``sbi_call`` 的返回值，因为它并不重要。如果同学们有兴趣的话，可以试着在 ``rust_main`` 中调用 ``console_putchar`` 来在屏幕上输出 ``OK`` 。接着在 Qemu 上运行一下，我们便可看到由我们自己输出的第一条 log 了。

Similar to the above method, we can also encapsulate the shutdown service ``SBI_SHUTDOWN`` into a ``shutdown`` function:

.. 类似上述方式，我们还可以将关机服务 ``SBI_SHUTDOWN`` 封装成 ``shutdown`` 函数：

.. code-block:: rust
    :linenos:

    // os/src/sbi.rs
    pub fn shutdown() -> ! {
        sbi_call(SBI_SHUTDOWN, 0, 0, 0);
        panic!("It should shutdown!");
    }

.. 实现格式化输出

Implement formatted output
-----------------------------------------------

The functionality of ``console_putchar`` is too limited. If you want to print a line ``Hello world!``, you need to make multiple calls. Can we use the ``println!`` macro to complete the output in one line like in the first section of this chapter? So we try to write our own ``println!`` macro based on ``console_putchar``.

.. ``console_putchar`` 的功能过于受限，如果想打印一行 ``Hello world!`` 的话需要进行多次调用。能否像本章第一节那样使用 ``println!`` 宏一行就完成输出呢？因此我们尝试自己编写基于 ``console_putchar`` 的 ``println!`` 宏。

.. code-block:: rust
    :linenos:

    // os/src/main.rs
    #[macro_use]
    mod console;

    // os/src/console.rs
    use crate::sbi::console_putchar;
    use core::fmt::{self, Write};

    struct Stdout;

    impl Write for Stdout {
        fn write_str(&mut self, s: &str) -> fmt::Result {
            for c in s.chars() {
                console_putchar(c as usize);
            }
            Ok(())
        }
    }

    pub fn print(args: fmt::Arguments) {
        Stdout.write_fmt(args).unwrap();
    }

    #[macro_export]
    macro_rules! print {
        ($fmt: literal $(, $($arg: tt)+)?) => {
            $crate::console::print(format_args!($fmt $(, $($arg)+)?));
        }
    }

    #[macro_export]
    macro_rules! println {
        ($fmt: literal $(, $($arg: tt)+)?) => {
            $crate::console::print(format_args!(concat!($fmt, "\n") $(, $($arg)+)?));
        }
    }

We write ``println!`` macros in the ``console`` submodule. The struct ``Stdout`` does not contain any fields, so it is called a unit-like struct (see [#unit-like-structs]_ ). The ``core::fmt::Write`` trait contains a ``write_fmt`` method that is useful for implementing ``println!`` macros. Considering this, we are going to implement ``Write`` trait for the structure ``Stdout``. In the ``Write`` trait, the ``write_str`` method must be implemented, so we need to implement this method for ``Stdout``. It is not difficult to implement, just traversing the incoming ``&str`` for each character of the string and call ``console_putchar`` to print the entire string passed in to the screen.

After that, ``Stdout`` can call the ``write_fmt`` method provided by the ``Write`` trait to implement the ``print`` function. In the declaration macros (Declarative macros, refer to [#declarative-macros]_ ) ``print!`` and ``println!`` will call the ``print`` function to complete the output.

Now we can use the ``print!`` and ``println!`` macros in ``rust_main`` to format the output. If you are interested, you can output ``Hello, world!`` to try.


.. 我们在 ``console`` 子模块中编写 ``println!`` 宏。结构体 ``Stdout`` 不包含任何字段，因此它被称为类单元结构体（Unit-like structs，请参考 [#unit-like-structs]_ ）。 ``core::fmt::Write`` trait 包含一个用来实现 ``println!`` 宏很好用的 ``write_fmt`` 方法，为此我们准备为结构体 ``Stdout`` 实现 ``Write`` trait 。在 ``Write`` trait 中， ``write_str`` 方法必须实现，因此我们需要为 ``Stdout`` 实现这一方法，它并不难实现，只需遍历传入的 ``&str`` 中的每个字符并调用 ``console_putchar`` 就能将传入的整个字符串打印到屏幕上。

.. 在此之后 ``Stdout`` 便可调用 ``Write`` trait 提供的 ``write_fmt`` 方法并进而实现 ``print`` 函数。在声明宏（Declarative macros，参考 [#declarative-macros]_ ） ``print!`` 和 ``println!`` 中会调用 ``print`` 函数完成输出。

.. 现在我们可以在 ``rust_main`` 中使用 ``print!`` 和 ``println!`` 宏进行格式化输出了，如有兴趣的话可以输出 ``Hello, world!`` 试一下。

.. note::

    **Rust Tips：Rust Trait**

    In the Rust language, trait is a type used to describe a collection of methods. A trait can be used to define an interface, which can be implemented by other types.
    As an example, suppose we have a simple Rust program with a trait called Shape that describes shapes:

    .. 在 Rust 语言中，trait（中文翻译：特质、特征）是一种类型，用于描述一组方法的集合。trait 可以用来定义接口（interface），并可以被其他类型实现。
    .. 举个例子，假设我们有一个简单的Rust程序，其中有一个名为 Shape 的 trait，用于描述形状：

    .. code-block:: rust
        :linenos:

        trait Shape {
            fn area(&self) -> f64;
        }


    We can use this trait to define a circle type:

    .. 我们可以使用这个 trait 来定义一个圆形类型：

    .. code-block:: rust
        :linenos:

        struct Circle {
            radius: f64,
        }

        impl Shape for Circle {
            fn area(&self) -> f64 {
                3.14 * self.radius * self.radius
            }
        }

    In this way, we can call the area method with an instance of the Circle type.

    .. 这样，我们就可以使用 Circle 类型的实例调用 area 方法了。

    .. code-block:: rust
        :linenos:

        let c = Circle { radius: 1.0 };
        println!("Circle area: {}", c.area());  // 输出: Circle area: 3.14    



.. 处理致命错误

Handle fatal errors
-----------------------------------------------

Error handling is an important part of programming. It can ensure the reliability and availability of the program, so that the program can calmly deal with more emergencies without crashing prematurely. Unlike C's return error number ``errno`` model and C++/Java's ``try-catch`` exception capture model, Rust divides errors into two categories: recoverable and unrecoverable errors. Here we are mainly concerned with unrecoverable errors. Just like in C++/Java, an exception is never handled after it is thrown. In Rust, if an unrecoverable error is encountered, the program will report an error and exit directly. For example, using the ``panic!`` macro will directly trigger an unrecoverable error and cause the program to exit. However, in our kernel, the current unrecoverable error handling mechanism is not perfect:

.. 错误处理是编程的重要一环，它能够保证程序的可靠性和可用性，使得程序能够从容应对更多突发状况而不至于过早崩溃。不同于 C 的返回错误编号 ``errno`` 模型和 C++/Java 的 ``try-catch`` 异常捕获模型，Rust 将错误分为可恢复和不可恢复错误两大类。这里我们主要关心不可恢复错误。和 C++/Java 中一个异常被抛出后始终得不到处理一样，在 Rust 中遇到不可恢复错误，程序会直接报错退出。例如，使用 ``panic!`` 宏便会直接触发一个不可恢复错误并使程序退出。不过在我们的内核中，目前不可恢复错误的处理机制还不完善：

.. code-block:: rust
    :linenos:

    // os/src/lang_items.rs
    use core::panic::PanicInfo;

    #[panic_handler]
    fn panic(_info: &PanicInfo) -> ! {
        loop {}
    }

It can be seen that in the current implementation, when an unrecoverable error is encountered, the ``panic`` function marked as the semantic item ``#[panic_handler]`` will be called, but it is just an infinite loop , causing the computer to get stuck here. With the help of the ``println!`` macro and the ``shutdown`` function implemented earlier, we can print the error message and shut down in the ``panic`` function:

.. 可以看到，在目前的实现中，当遇到不可恢复错误的时候，被标记为语义项 ``#[panic_handler]`` 的 ``panic`` 函数将会被调用，然而其中只是一个死循环，会使得计算机卡在这里。借助前面实现的 ``println!`` 宏和 ``shutdown`` 函数，我们可以在 ``panic`` 函数中打印错误信息并关机：

.. code-block:: rust
    :linenos:

    // os/src/main.rs
    #![feature(panic_info_message)]

    // os/src/lang_item.rs
    use crate::sbi::shutdown;
    use core::panic::PanicInfo;

    #[panic_handler]
    fn panic(info: &PanicInfo) -> ! {
        if let Some(location) = info.location() {
            println!(
                "Panicked at {}:{} {}",
                location.file(),
                location.line(),
                info.message().unwrap()
            );
        } else {
            println!("Panicked: {}", info.message().unwrap());
        }
        shutdown()
    }

We try to print more detailed information, including the source file where the panic is located and the number of lines of code. We try to parse the information from the ``PanicInfo`` passed in, and if the parsing is successful, print it out along with the panic error message. We need to add ``#![feature(panic_info_message)]`` at the beginning of ``main.rs`` to get the error message through ``PanicInfo::message``. After printing, we directly call the ``shutdown`` function to shut down.

.. 我们尝试打印更加详细的信息，包括 panic 所在的源文件和代码行数。我们尝试从传入的 ``PanicInfo`` 中解析这些信息，如果解析成功的话，就和 panic 的报错信息一起打印出来。我们需要在 ``main.rs`` 开头加上 ``#![feature(panic_info_message)]`` 才能通过 ``PanicInfo::message`` 获取报错信息。当打印完毕之后，我们直接调用 ``shutdown`` 函数关机。

To test that our implementation is correct, we change ``rust_main`` to:

.. 为了测试我们的实现是否正确，我们将 ``rust_main`` 改为：

.. code-block:: rust
    :linenos:

    // os/src/main.rs
    #[no_mangle]
    pub fn rust_main() -> ! {
        clear_bss();
        println!("Hello, world!");
        panic!("Shutdown machine!");
    }

Using Qemu to run our kernel, the result is:

.. 使用 Qemu 运行我们的内核，运行结果为：

.. code-block::

    [RustSBI output]
    Hello, world!
    Panicked at src/main.rs:26 Shutdown machine!

It can be seen that the source file and the number of lines of code where the panic is located are correctly reported, which will bring great convenience to the development and debugging of our subsequent chapters. At this point, we have implemented a bare-metal application that can run on the computer simulated by Qemu. Specifically, it includes the above-mentioned `rust_main` function. And with other parts, such as `entry.asm`, `lang_items.rs`, `console.rs`, `sbi.rs` form LibOS, the Cambrian "Trilobite" operating system that supports bare-metal applications is born. 

.. 可以看到，panic 所在的源文件和代码行数被正确报告，这将为我们后续章节的开发和调试带来很大方便。到这里，我们就实现了一个可以在Qemu模拟的计算机上运行的裸机应用程序，其具体内容就是上述的 `rust_main`函数，而其他部分，如 `entry.asm` 、 `lang_items.rs` 、`console.rs` 、 `sbi.rs` 则形成了支持裸机应用程序的寒武纪“三叶虫”操作系统 -- LibOS 。

.. note::

    **Rust Tips: Rust Recoverable Errors**

    When there is a possibility of an error, the return value of a Rust function can belong to a special type that can cover two situations: either the function exits normally, and the function returns a normal return value; or an error occurred during the execution of the function, the function returns the type of error. Rust's type system guarantees that this return value will not be abused without the programmer's awareness, that is, the programmer must explicitly branch to check the error or force the exclusion of errors. Without any processing, there's no way to get meaningful results out of it for later use or to compile. In this way, a large part of errors caused by programmers' negligence (such as using a null pointer returned by a function without judgment) are eliminated.

    In Rust there are two such special types, both of which are enumeration structures:

    - ``Option<T>`` can either have value ``Option::Some<T>`` or ``Option::None``;
    - ``Result<T, E>`` can not only save the return value of an operation ``Result::Ok<T>``, but also indicate that an error occurred during the operation ``Result::Err<E>``.

    We can use ``Option/Result`` to hold a value that cannot be guaranteed to be there, or absent, or success/failure. Then you can match by ``if let``. And in scenarios when you can determine, take out the value inside directly through ``unwrap``. For details, please refer to the official Rust documentation [#recoverable-errors]_.

    .. **Rust Tips：Rust 可恢复错误**

    .. 在有可能出现错误时，Rust 函数的返回值可以属于一种特殊的类型，该类型可以涵盖两种情况：要么函数正常退出，则函数返回正常的返回值；要么函数执行过程中出错，则函数返回出错的类型。Rust 的类型系统保证这种返回值不会在程序员无意识的情况下被滥用，即程序员必须显式对其进行分支判断或者强制排除出错的情况。如果不进行任何处理，那么无法从中得到有意义的结果供后续使用或是无法通过编译。这样，就杜绝了很大一部分因程序员的疏忽产生的错误（如不加判断地使用某函数返回的空指针）。

    .. 在 Rust 中有两种这样的特殊类型，它们都属于枚举结构：

    .. - ``Option<T>`` 既可以有值 ``Option::Some<T>`` ，也有可能没有值 ``Option::None``；
    .. - ``Result<T, E>`` 既可以保存某个操作的返回值 ``Result::Ok<T>`` ，也可以表明操作过程中出现了错误 ``Result::Err<E>`` 。

    .. 我们可以使用 ``Option/Result`` 来保存一个不能确定存在/不存在或是成功/失败的值。之后可以通过匹配 ``if let`` 或是在能够确定
    .. 的场合直接通过 ``unwrap`` 将里面的值取出。详细的内容可以参考 Rust 官方文档 [#recoverable-errors]_ 。


.. [#unit-like-structs] https://doc.rust-lang.org/book/ch05-01-defining-structs.html#unit-like-structs-without-any-fields
.. [#declarative-macros] https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming
.. [#recoverable-errors] https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html