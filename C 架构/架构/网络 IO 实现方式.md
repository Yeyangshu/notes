# 网络 IO 实现方式

当我们使用 Socket 套接字进行网络通信开发时，有哪些实现方式呢？

- BIO
- NIO
- AIO

## 1 BIO

BIO  即 Blocking IO，采用阻塞的方式实现。也就是一个 Socket 套接字需要使用一个线程来进行处理。发生建立连接、读数据、写数据的操作时，都有可能会阻塞。

## 2 NIO

NIO 即 Nonblocking IO，基于事件驱动思想，采用的是 Reactor 模式。在 Java 实现的服务器端系统中也是采用比较多的一种方式。相对于 BIO，NIO 的一个明显的好处就是不需要为每个 Socket 套接字分配一个线程，而可以在一个线程中处理多个 Socket 套接字相关的工作。

在 NIO 的方式下不是用单个线程去应对单个 Socket 套接字，而是统一通过 Reactor 对所有客户端的 Socket 套接字的事件作处理，然后派发到不同的线程中。这样就解决了 BIO 中为支撑更多的 Socket 套接字而需要打开更多的线程的问题。

## 3 AIO

AIO 即 Asynchronous IO，就是异步 IO。AIO 采用 Proactor 模式。AIO 与 NIO 的差别是，AIO 在进行读写时，只需要调用相应的读/写操作，只需要调用响应的 read/write 方法，并且需要传入 CompletionHandler（动作完成的处理器）；在动作完成后，会调用 CompletionHandler。



AIO 和 NIO 的最大的一个区别是，NIO 在有通知时可以进行相关操作，例如读和写，而 AIO 在有通知时表示相关操作已经完成。

