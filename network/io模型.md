
##阻塞非阻塞 异步同步
http://www.sohu.com/a/231730171_231667

###Unix网络编程中的五种IO模型

Blocking IO - 阻塞IO
NoneBlocking IO - 非阻塞IO
IO multiplexing - IO多路复用
signal driven IO - 信号驱动IO
asynchronous IO - 异步IO

由于signal driven IO在实际使用中并不常用，所以这里只讨论剩下的四种IO模型。

在讨论之前先说明一下IO发生时涉及到的对象和步骤，对于一个network IO，它会涉及到两个系统对象：

application 调用这个IO的进程
kernel 系统内核
那他们经历的两个交互过程是：

阶段1 wait for data 等待数据准备
阶段2 copy data from kernel to user 将数据从内核拷贝到用户进程中
之所以会有同步、异步、阻塞和非阻塞这几种说法就是根据程序在这两个阶段的处理方式不同而产生的。了解了这些背景之后，我们就分别针对四种IO模型进行讲解

###Blocking IO - 阻塞IO

在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概如下图：

当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据。对于network IO来说，很多时候数据在一开始还没有到达（比如，还没有收到一个完整的UDP包），这个时候kernel就要等待足够的数据到来。而在用户进程这边，整个进程会被阻塞。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。

所以，blocking IO的特点就是在IO执行的两个阶段都被block了。

###NoneBlockingIO - 非阻塞IO

linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：


从图中可以看出，当用户进程发出recvfrom这个系统调用后，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个结果（no datagram ready）。从用户进程角度讲 ，它发起一个操作后，并没有等待，而是马上就得到了一个结果。用户进程得知数据还没有准备好后，它可以每隔一段时间再次发送recvfrom操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。

所以，用户进程其实是需要不断的主动询问kernel数据好了没有。

###IO multiplexing - IO多路复用
I/O多路复用(multiplexing)是网络编程中最常用的模型，像我们最常用的select、epoll都属于这种模型。以select为例：

看起来它与blocking I/O很相似，两个阶段都阻塞。但它与blocking I/O的一个重要区别就是它可以等待多个数据报就绪（datagram ready），即可以处理多个连接。这里的select相当于一个“代理”，调用select以后进程会被select阻塞，这时候在内核空间内select会监听指定的多个datagram (如socket连接)，如果其中任意一个数据就绪了就返回。此时程序再进行数据读取操作，将数据拷贝至当前进程内。由于select可以监听多个socket，我们可以用它来处理多个连接。

在select模型中每个socket一般都设置成non-blocking，虽然等待数据阶段仍然是阻塞状态，但是它是被select调用阻塞的，而不是直接被I/O阻塞的。select底层通过轮询机制来判断每个socket读写是否就绪。

当然select也有一些缺点，比如底层轮询机制会增加开销、支持的文件描述符数量过少等。为此，Linux引入了epoll作为select的改进版本。

###asynchronous IO - 异步IO

异步I/O在网络编程中几乎用不到，在File I/O中可能会用到：

这里面的读取操作的语义与上面的几种模型都不同。这里的读取操作(aio_read)会通知内核进行读取操作并将数据拷贝至进程中，完事后通知进程整个操作全部完成（绑定一个回调函数处理数据）。读取操作会立刻返回，程序可以进行其它的操作，所有的读取、拷贝工作都由内核去做，做完以后通知进程，进程调用绑定的回调函数来处理数据。

https://blog.csdn.net/zh13544539220/article/details/44856363

