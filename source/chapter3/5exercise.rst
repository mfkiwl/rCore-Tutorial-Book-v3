chapter3练习
=======================================

- 本节难度： **并不那么简单了！早点动手** 

编程作业
--------------------------------------

stride 调度算法
+++++++++++++++++++++++++++++++++++++++++

lab3中我们引入了任务调度的概念，可以在不同任务之间切换，目前我们实现的调度算法十分简单，存在一些问题且不存在优先级。现在我们要为我们的 os 实现一种带优先级的调度算法：stide 调度算法。

算法描述如下:

(1) 为每个进程设置一个当前 stride，表示该进程当前已经运行的“长度”。另外设置其对应的 pass 值（只与进程的优先权有关系），表示对应进程在调度后，stride 需要进行的累加值。

(2) 每次需要调度时，从当前 runnable 态的进程中选择 stride 最小的进程调度。对于获得调度的进程 P，将对应的 stride 加上其对应的步长 pass。

(3) 一个时间片后，回到上一步骤，重新调度当前 stride 最小的进程。

可以证明，如果令 P.pass = BigStride / P.priority 其中 P.priority 表示进程的优先权（大于 1），而 BigStride 表示一个预先定义的大常数，则该调度方案为每个进程分配的时间将与其优先级成正比。证明过程我们在这里略去，有兴趣的同学可以在网上查找相关资料。

其他实验细节：

- stride 调度要求进程优先级 :math:`\geq 2`，所以设定进程优先级 :math:`\leq 1` 会导致错误。
- 进程初始 stride 设置为 0 即可。
- 进程初始优先级设置为 16。

tips: 可以使用优先级队列比较方便的实现 stride 算法。如使用：`BinaryHeap <https://doc.rust-lang.org/alloc/collections/binary_heap/struct.BinaryHeap.html>`_ 。

实验要求
+++++++++++++++++++++++++++++++++++++++++

- 完成分支: ch3。

- 完成实验指导书中的内容，实现 sys_yield，实现协作式和抢占式的调度。

- 实现 stride 调度算法，实现 sys_gettime, sys_set_priority 两个系统调用并通过 `Rust测例 <https://github.com/DeathWish5/rCore_tutorial_tests>`_ 中 chapter3 对应的所有测例，测例详情见对应仓库，系统调用具体要求参考 `guide.md <https://github.com/DeathWish5/rCore_tutorial_tests/blob/master/guide.md>`_ 。

需要说明的是 lab3 有3类测例，``ch3_0_*`` 用来检查基本 syscall 的实现，``ch3_1_*`` 基于 yield 来检测基本的调度，``ch3_2_*`` 基于时钟中断来测试 stride 调度算法实现的正确性。测试时可以分别测试 3 组测例，使得输出更加可控、更加清晰。

特别的，我们有一个死循环测例 ``ch3t_deadloop`` 用来保证大家真的实现了时钟中断。这一章中我们人为限制一个程序执行的最大时间（必须很大），超过就杀死，这样，我们的程序更不容易被恶意程序伤害。这一规定可以在实验4开始删除，仅仅为通过 lab3 测例设置。

challenge: 实现多核，可以并行调度。

实验检查
++++++++++++++++++++++++++++++++++++++++

- 实验目录要求

    目录要求不变(参考lab1目录或者示例代码目录结构)。同样在 os 目录下 `make run` 之后可以正确加载用户程序并执行。

    目标用户目录 ``../user/build/bin``。

- 检查

    可以正确 `make run` 执行，可以正确执行目标用户测例，并得到预期输出（详见测例注释）。

简答作业
--------------------------------------------

(1) 简要描述这一章的进程调度策略。何时进行进程切换？如何选择下一个运行的进程？如何处理新加入的进程？

(2) 在 C 版代码中，同样实现了类似 RR 的调度算法，但是由于没有 VecDeque 这样直接可用的数据结构（Rust很棒对不对），C 版代码的实现严格来讲存在一定问题。大致情况如下：C版代码使用一个进程池（也就是一个 struct proc 的数组）管理进程调度，当一个时间片用尽后，选择下一个进程逻辑在 `chapter３相关代码 <https://github.com/DeathWish5/ucore-Tutorial/blob/ch3/kernel/proc.c#L60-L74>`_ ，也就是当第 i 号进程结束后，会以 i -> max_num -> 0 -> i 的顺序遍历进程池，直到找到下一个就绪进程。C 版代码新进程在调度池中的位置选择见 `chapter5相关代码 <https://github.com/DeathWish5/ucore-Tutorial/blob/ch5/kernel/proc.c#L90-L98>`_ ，也就是从头到尾遍历进程池，找到第一个空位。

    (2-1) 在目前这一章（chapter3）两种调度策略有实质不同吗？考虑在一个完整的 os 中，随时可能有新进程产生，这两种策略是否实质相同？

    (2-2) 其实 C 版调度策略在公平性上存在比较大的问题，请找到一个进程产生和结束的时间序列，使得在该调度算法下发生：先创建的进程后执行的现象。你需要给出类似下面例子的信息（有更详细的分析描述更好，但尽量精简）。同时指出该序列在你实现的 stride 调度算法下顺序是怎样的？

        .. list-table:: 调度顺序举例
            :header-rows: 1
            :align: center

            *   - 时间点
                - 0
                - 1
                - 2
                - 3
                - 4
                - 5
                - 6
                - 7
            *   - 运行进程
                - 
                - p1
                - p2
                - p3
                - p1
                - p4
                - p3
                - 
            *   - 事件
                - p1、p2、p3产生
                - 
                - p2 结束
                - p4 产生
                - p1 结束
                - p4 结束
                - p3 结束
                - 

        产生顺序：p1、p2、p3、p4。第一次执行顺序: p1、p2、p3、p4。

        其他细节：不考虑进程优先级，允许进程在其他进程执行时结束。

(3) stride 算法深入

    stride算法原理非常简单，但是有一个比较大的问题。例如两个 pass = 10 的进程，使用 8bit 无符号整形储存 stride， p1.stride = 255, p2.stride = 250，在 p2 执行一个时间片后，理论上下一次应该 p1 执行。

    - 实际情况是轮到 p1 执行吗？为什么？

    我们之前要求进程优先级 >= 2 其实就是为了解决这个问题。可以证明，在进程优先级全部 >= 2 的情况下，如果严格按照算法执行，那么 STRIDE_MAX – STRIDE_MIN <= BigStride / 2。

    - 为什么？尝试简单说明（传达思想即可，不要求严格证明）。

    在已知以上结论的前提下，我们可以通过设计 Stride 的比较接口，结合 BinaryHeap 的 pop 接口可以很容易的找到真正最小的 Stride。
    
    - 请补全如下 ``partial_cmp`` 函数（假设永远不会相等）。

    .. code-block:: rust

        use core::cmp::Ordering;

        struct Stride(u64);

        impl PartialOrd for Stride {
            fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
                // ...
            }
        }

        impl PartialEq for Person {
            fn eq(&self, other: &Self) -> bool {
                false
            }
        }
    

报告要求
-------------------------------

- 简单总结与上次实验相比本次实验你增加的东西（控制在5行以内，不要贴代码）。
- 完成问答问题。
- (optional) 你对本次实验设计及难度/工作量的看法，以及有哪些需要改进的地方，欢迎畅所欲言。

参考信息
-------------------------------
如果有兴趣进一步了解stride调度相关内容，可以尝试看看：

- `作者 Carl A. Waldspurger 写这个调度算法的原论文 <https://people.cs.umass.edu/~mcorner/courses/691J/papers/PS/waldspurger_stride/waldspurger95stride.pdf>`_
- `作者 Carl A. Waldspurger 的博士生答辩slide <http://www.waldspurger.org/carl/papers/phd-mit-slides.pdf>`_ 
- `南开大学实验指导中对Stride算法的部分介绍 <https://nankai.gitbook.io/ucore-os-on-risc-v64/lab6/tiao-du-suan-fa-kuang-jia#stride-suan-fa>`_
- `NYU OS课关于Stride Scheduling的Slide <https://cs.nyu.edu/rgrimm/teaching/sp08-os/stride.pdf>`_