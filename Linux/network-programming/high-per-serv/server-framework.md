# 高性能服务器程序框架

> 全书的核心!

> [高性能网络编程(一)：单台服务器并发TCP连接数到底可以有多少](http://www.52im.net/thread-561-1-1.html) 暂时不知道放哪先放这，真的是很棒的专栏! 下一节就讲C10K问题，值得好好刷一遍

将服务器结构为三个主要模块
- I/O处理单元; 四种I/O模型和两种高效事件处理模式
- 逻辑单元; 两种高效并发模式，以及有限状态机
- 存储单元(暂不讨论)

client连接请求是随机到达的异步事件，因此服务器需要使用某种I/O模型来监听这一事件。

I/O处理单元等待并接受新的client连接，接收client数据，响应数据给client；数据的收发并不一定在I/O处理单元进行，也可以在逻辑单元，这取决于**事件处理模式**

## IO模型

显然，我们只有在事件已经发生时操作非阻塞I/O，才能提高程序的运行效率。**因此非阻塞IO通常需要和其他IO通知机制一起使用**，比如I/O复用和SIGIO信号。

**I/O复用**是指：应用程序通过I/O复用函数向内核注册**一组事件**(而非单个)，内核再通过I/O复用函数把其中就绪的时间通知给应用程序。

**I/O复用函数本身是阻塞的**，能提高程序效率的原因在于他们能同时监听多个I/O事件(select, poll, epoll)。但对I/O本身的读写操作是非阻塞的(?????)

> 同步异步，阻塞非阻塞，要看看UNP了吧

同步I/O，都是在I/O事件发生以后，由应用程序来完成的。异步I/O的读写操作总是立刻返回，真正读写由内核接管。也可以认为，同步I/O向应用程序通知的是I/O就绪事件，异步I/O向应用程序通知I/O完成事件。


## 两种高效的事件处理模式

**Reactor**
使用同步的epoll_wait
主线程(I/O处理单元)只负责监听文件描述上是否有事件发生，有的话立即将该事件通知工作线程(逻辑单元)。
读写数据，接受新的连接，处理客户请求均在工作线程完成。


**Proactor**
使用异步I/O调用
所有I/O操作都交给主线程和内核来处理，工作线程仅负责业务逻辑

**同步IO模拟Proactor**

## 两种并发模式

多进程和多线程是两种实现方式，但不是模式; 并发模式是指I/O处理单元和多个逻辑单元之间协调完成任务的方法，主要两种：半同步/半异步模式，领导者/追随者模式

**半同步/半异步模式**
**这里的同步异步和IO的同步异步概念完全不同**。IO的同步异步区分的是内核向程序通知的是什么IO事件(就绪/完成)，以及谁来完成IO读写(应用程序/内核)；这里就是普遍的同步异步概念。

简单来说：异步去做IO，同步去做逻辑
> 突然想起当时在Tencent写的SDK不就是这样的吗，而且好像还是半同步/半反应堆


**领导者/追随者模式**
领导者同一时间只有一个，但不会是一个线程固定作为领导者

## 有限状态机