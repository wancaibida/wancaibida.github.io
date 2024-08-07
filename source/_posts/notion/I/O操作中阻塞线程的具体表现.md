---
created: 2024-07-24 02:05:00
categories:
  - jvm
tags:
  - linux
updated: 2024-07-24 08:09:00
date: 2024-07-24 00:00:00
title: I/O操作中阻塞线程的具体表现
id: fec3c214-bce5-429c-bb23-bb4b6f22729f
---

在 I/O 操作中阻塞线程的具体表现：

1. 系统调用层面：
   - 当程序执行 I/O 操作（如读取文件）时，它会发起一个系统调用。
   - 在传统的阻塞 I/O 模型中，这个系统调用会导致线程进入等待状态。
2. 线程状态变化：
   - 线程从"运行"状态转变为"阻塞"状态。
   - 操作系统会将这个线程从 CPU 的执行队列中移除。
3. CPU 资源释放：
   - 阻塞的线程不再占用 CPU 时间片。
   - CPU 可以切换到其他就绪的线程执行。
4. 等待 I/O 完成：
   - 线程保持阻塞状态，直到 I/O 操作完成。
   - 完成可能需要相对较长的时间（相对于 CPU 速度而言）。
5. 唤醒过程：
   - I/O 操作完成后（通常通过中断机制通知 CPU），操作系统将线程标记为就绪状态。
   - 线程重新进入就绪队列，等待被调度执行。
6. 继续执行：
   - 当线程再次获得 CPU 时间片时，它从 I/O 操作的下一条指令继续执行。

阻塞的影响：

- 资源利用率低：阻塞的线程无法执行其他任务，即使 I/O 操作耗时很长。
- 并发处理能力受限：每个阻塞的 I/O 操作都可能需要一个专门的线程，限制了系统能同时处理的 I/O 操作数量。

java NIO

- 非阻塞 I/O：
  - NIO 允许线程发起 I/O 操作后立即返回，而不是等待操作完成。
  - 线程可以继续执行其他任务，而不是被阻塞。
- 多路复用：
  - NIO 使用选择器（Selector）来管理多个通道（Channel）。
  - 一个线程可以监控多个 I/O 操作的状态。
- 工作原理：
  - 线程不会"阻塞然后去做别的事"，而是根本不会阻塞。
  - 它会持续检查多个 I/O 操作的状态，只在数据就绪时才进行实际的 I/O 操作。
- 实际操作流程：
  a. 设置通道为非阻塞模式。
  b. 将多个通道注册到一个选择器上。
  c. 线程调用选择器的 select()方法。
  d. 选择器返回已经就绪的通道。
  e. 线程对就绪的通道执行 I/O 操作。

> A context switch can occur while the kernel is executing a system call on behalf of the user. If the system call blocks because it is waiting for some event to occur, then the kernel can put the current process to sleep and switch to another process. For example, if a system call requires a disk access, the kernel can opt to perform a context switch and run another process instead of waiting for the data to arrive from the disk. Another example is the system call, which is an explicit request to put the calling process to sleep. In general, even if a system call does not block, the kernel can decide to perform a context switch rather than return control to the calling process.
