# Paper Overview of FastSocket

"Scalable Kernel TCP Design and Implementation for Short-Lived Connections"这篇文章发表于ASPLOS 2016，针对短连接short-lived TCP connections的特定设计了一种可扩展的、BSD socket API兼容的全功能内核协议栈Fastsocket。

随着CPU核数的增加，内核协议栈的可扩展性对于网络应用的性能提升起到了决定性的因素。在过去传统的网络应用中，TCP建立一次连接后，进行长时间的数据交互，因此，传统TCP协议栈中以TCB、VFS等metadata方式管理连接的设计并不会产生太多的扩展性问题。然而，对于微博等以短消息为特点的社交媒体来说，其TCP连接都属于Short-lived connection，如微博中一个tcp连接典型过程只涉及600字节请求和1200字节响应两个报文，传统协议栈的metadata管理方式就会产生严重的争用问题，导致随着cpu核数的增加性能反而下降。

对于传统协议栈来说，其TCB管理连接的方式存在接收数据包和处理发送数据包不一定在同一个cpu核上处理的问题，即connection locality；VFS对socket的管理存在inode、dentry等共享状态的同步问题。

Fastsocket采用的方法主要有：
- 对TCB中的全局数据结构进行分割，每一个CPU Core设置一个per-core listen table和per-core established table,
- 当接收到一个数据包时，利用Recive Flow Deliver将数据包发送到对应的CPU核，即最大化实现connection locality，减少CPU Cache boucing现象，其中采用了将cpu core id哈希到源端口上的办法；
- 针对VFS中的调用路径进行优化，避免lock-intersive的接口，如随着socket创建销毁而进行的inode/dentry的初始化和析构；同时保持兼容性,对dentry和inode等磁盘文件所用结构体进行裁剪，只保持/proc文件系统相功能。


经过测试，在24核机器中，针对短连接，Fastsocket类比于Linux标准协议栈，其提速20.4倍；对于Nginx和HAProxy，性能提升267%和621%。同时，Fastsocket的兼容性保证了标准协议栈应用无需移植即可在Fastsocket上运行，通过在新浪微博上部署，在日均5千万用户的数以亿计的请求连接负载下稳定运行。
