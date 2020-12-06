# Java I/O与网络编程

参考《疯狂Java讲义》

@[toc]
## Java I/O基础

### File类

代表与平台无关的文件和目录，不能访问文件内容，如果需要访问文件内容，要使用输入流/输出流；

### 理解Java的IO流

#### 流的分类

- 输入流和输出流；
- 字节流（操作16位的字符）和字符流（操作8位的字节）；
- 节点流（可以从/向特定的IO设备读/写数据的流，低级流）和处理流（对一个已存在的流进行连接或封装，高级流）；

#### 流的概念模型

Java IO流的40多个类都是从以下4个基类派生：

- InputStream/Reader：所有字节/字符输入流的基类；
- OutputStream/Writer：所有字节/字符输出流的基类；

这些流都是阻塞式的输入/输出

**处理流的好处：**

- 性能的提高：增加缓冲提高输入/输出的效率；
- 操作的便捷：更多便捷的方法；
- 隐藏底层设备上节点流的差异，并对外提供更加方便的输入/输出方法；

### 输入/输出流体系

#### 处理流的用法

只要流的构造器参数不是一个物理节点，而是已存在的流，则为处理流；

#### 输入/输出流体系

- 通常来说，字节流的功能比字符流的功能强大，因为计算机里所有的数据都是二进制的，而字节流可以处理所有的二进制文件；
- 如果输入输出的内容是文本内容，则使用字符流，如果是二进制内容，则使用字节流；

#### 转换流

InputStreamReader将字节输入流转换成字符输入流，OutputStreamWriter将字节输出流转换成字符输出流；

#### 推回输入流

PushbackInputStream和PushbackReader，可以把已读的内容推回缓冲区；

### Java虚拟机读写其他进程的数据

使用Runtime对象的exec()方法可以运行平台上的其他程序，该方法产生一个Process对象，它提供了以下方法：

```java
InputStream getErrorStream(); //获取子进程的错误流
InputStream getInputStream(); //获取子进程的输入流（可通过此方法获得调用程序的输出）
InputStream getErrorStream(); //获取子进程的输出流
```

### RandomAccessFile

丰富的文件内容访问类（用法略）

### NIO

#### nio概述

- nio采用内存映射文件的方式处理输入输出，nio将文件或文件的一段区域映射到内存中，这样就可以像访问内存一样来访问文件了；
- Channel是对传统io的模拟，在nio中所有的数据都需要通过Channel传输，它与传统io最大的区别是提供了一个map()方法，可以将一块数据映射到内存中，传统io是面向流的处理，nio是面向块的处理；
- Buffer可以被理解成一个容器，本质上是一个数组，发送到Channel中的所有对象都必须先放到Buffer中，而Channel中读取数据也必须先放到Buffer中；
- nio也提供了用于支持非阻塞式输入/输出的Selector类；

#### 使用Buffer

Buffer中有三个重要概念：

- 容量（capacity）：表示该buffer的最大数据容量，即最多可以存储多少数据，其不可能为负值，创建后不能改变；
- 界限（limit）：第一个不应该被读出或写入的缓冲区位置索引（位于limit后的数据无法被读也无法被写）；
- 位置（position）：用于指明下一个可以被读出或写入的缓冲区位置索引，其刚好等于已读的数据数量；

Buffer的方法：

- Buffer的flip()方法可以将limit置为position，然后把position置为0，之后Buffer准备开始输出数据（保证使用get时不会读到null数据）；
- Buffer的clear()方法可以将position置为0，limit置为capacity，之后Buffer可以重新装入数据（clear不会清除数据，只是移动指针）；
- Buffer的所有子类还提供了put()和get()方法，用于放入和取出数据；

#### 使用Channel

Channel类似于传统的流对象，但是与流对象有两个主要区别：

- Channel可以直接将指定文件的部分或全部直接映射成Buffer；
- 程序不能直接访问Channel中的数据，Channel只能与Buffer交互；
- Channel按照功能进行划分，如支持线程之间通信的Pipe.SinkChannel、Pipe.SourceChannel，用于支持TCP网络通信的ServerSocketChannel、SocketChannel，用于UDP网络通信的DatagramChannel等；
- 所有的Channel都不应该通过构造器创建，而是通过传统的节点流的getChannel()方法获取；
- map()方法用于将Channel中的数据映射成ByteBuffer，而read()或write()方法用于从Buffer中读取或写入数据；

#### 文件锁

FileChannel中提供lock()和tryLock()方法

### Java7的NIO2

对原有的NIO进行了重大改进，主要包括：

- 提供了全面的文件IO和文件系统访问支持；
- 基于异步Chanel的IO；

#### Path、Paths和Files核心API

Path接口代表一个平台无关的平台路径，Files、Paths则是两个工具类

#### 使用FileVisitor遍历文件和目录

优雅地遍历文件和子目录

#### 使用WatchService监控文件变化

监听path代表地目录下的文件变化

## 网络编程

### 基于TCP协议的网络编程

TCP/IP通信协议是一种可靠的网络协议，它在通信的两端各建立一个Socket，从而在通信的两端之间形成网络虚拟链路。一旦建立了虚拟的网络链路，两端的程序就可以通过虚拟链路进行通信。Java使用Socket对象来代表两端的通信端口，并通过Socket产生IO流来进行网络通信；

#### 使用ServerSocket创建TCP服务器端

- ServerSocket对象用于监听来自客户端的Socket连接，如果没有连接，它将一直处于等待状态；
- 使用accept()方法来接收来自客户端Socket的连接请求，如果接收到请求则返回一个客户端Socket对应的Socket，否则方法一直处于阻塞状态；

#### 使用Socket进行通信

- 客户端可以使用Socket来连接到指定的服务器；
- Socket提供了getInputStream()和getOutputStream()方法来让程序可以向Socket取数据和写数据；

#### 使用NIO实现非阻塞Socket通信

使用ServerSocket与Socket实现的通信程序是基于阻塞式API的，服务器必须为每个客户端都提供一个独立线程进程处理，当有大量客户端时会导致服务器性能下降。使用NIO API则可以让服务器端使用一个或有限几个线程来同时处理连接到服务器端的所有客户端；

Java NIO为非阻塞式Socket提供了几个特殊类：

**Selector**

- 它是SelectableChanel对象的多路复用器，所有希望采用非阻塞方式进行通信的Channel都应该注册到Selector对象，可以使用它类的open()方法创建一个默认的Selector实例。Selector可以同时监控多个SelectorChannel的IO状况，是非阻塞IO的核心。
- select()方法监控所有注册的Channel，当它们中间有需要处理的IO操作时，该方法将对应的SelectionKey加入到被选择的SelectionKey集合中，并返回这些Channel的数量；

**SelectableChannel**

- 代表可以支持非阻塞IO操作的Channel对象，它可以被注册到Selector上（通过调用register()方法），这种注册关系由SelectionKey实例表示；
- SelectableChannel对象支持阻塞和非阻塞两种模式，必须使用非阻塞模式才可以利用非阻塞IO操作；

**SelectionKey**：代表SelectableChannel和Selector之间的注册关系；

**ServerSocketChannel**：对应ServerSocket，支持非阻塞操作，也提供了accept()方法；

**SocketChannel**：对应Socket，支持非阻塞操作，还实现了ByteChannel接口；

服务器上所有的Channel都需要向Selector注册，而Selector负责监视这些Socket的IO状态；

#### 使用Java7的AIO实现非阻塞通信

- Java7的NIO2提供了异步Channel支持，这种异步Channel可以提供更高效的IO，这种基于异步Channel的IO机制也被称为异步IO（Asynchronous IO）；
- AsynchronousServerSocketChannel是一个负责监听的Channel，与ServerSocketChannel相似；
- AsynchronousChannelGroup是异步Channel的分组管理器，它可以实现资源共享，创建AsynchronousChannelGroup时需传入一个ExecutorService，该线程池负责处理IO事件和触发CompletionHandler；

### 基于UDP协议的网络编程

UDP是一种不可靠的协议，它在通信实例的两端各建立一个Socket，但这两个Socket之间并没有虚拟链路，只是发送、接收数据报的对象。Java提供了DatagramSocket对象作为基于UDP协议的Socket，使用DatagramPacket代表DatagramSocket发送、接收的数据报；

#### UDP协议基础

UDP协议是一种面向非连接的协议，面向非连接是只在通信前不必与对方先建立连接，不管对方状态就直接发送，适用于一次只传送少量数据、对可靠性要求不高的应用环境；

UDP协议与TCP协议的简单对比：

- TCP协议：可靠，传输大小无限制，但是需要连接建立时间，差错控制开销大；
- UDP协议：不可靠，差错控制开销较小，传输大小限制在64KB以下，不需要建立连接；

#### 使用DatagramSocket发送、接收数据

- DatagramSocket本身只是码头，不维护状态，不能产生IO流，它的唯一作用就是接收和发送数据报；
- DatagramSocket使用send()和receive()方法来发送和接收数据；
- DatagramPacket可以指定将要发往的地址和端口；

#### 使用MulticastSocket实现多点广播

MulticastSocket可以将数据报以广播方式发送到多个客户端；

### 使用代理服务器

从Java5开始，Java提供了Proxy和ProxySelector两个类，前者代表一个代理服务器，可以在打开URLConnection连接和创建Socket连接时指定Proxy。后者代表一个代理选择器，提供了对代理服务器更加灵活的控制，它可以对HTTP、HTTPS、FTP、SOCKS等进行分别设置，还可以设置不太需要通过代理服务器的主机和地址等；

**代理服务器**

介于浏览器和服务器之间的服务器，主要提供两个功能：

- 突破自身IP地址限制，对外隐藏自身IP地址；
- 提高访问速度，代理服务器提供的缓冲功能可以避免每个用户都直接访问远程主机，从而提高客户端访问速度；