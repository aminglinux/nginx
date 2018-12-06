### events配置部分

####worker_connections  1024;

    定义每个work_process同时开启的最大连接数，即允许最多只能有这么多连接。
    
####accept_mutex on;

    当某一个时刻只有一个网络连接请求服务器时，服务器上有多个睡眠的进程会被同时叫醒，这样会损耗一定的服务器性能。
    Nginx中的accept_mutex设置为on，将会对多个Nginx进程（worker processer）接收连接时进行序列化，防止多个进程争抢资源。
    默认就是on。
    
####multi_accept on;

    nginx worker processer可以做到同时接收多个新到达的网络连接，前提是把该参数设置为on。
    默认为off，即每个worker process一次只能接收一个新到达的网络连接。
    
####use epoll;

    Nginx服务器提供了多个事件驱动器模型来处理网络消息。
    其支持的类型有：select、poll、kqueue、epoll、rtsing、/dev/poll以及eventport。
    
    * select：只能在Windows下使用，这个事件模型不建议在高负载的系统使用
    
    * poll:Nginx默认首选，但不是在所有系统下都可用
    
    * kqueue:这种方式在FreeBSD 4.1+, OpenBSD2.9+, NetBSD 2.0, 和 MacOS X系统中是最高效的
    
    * epoll: 这种方式是在Linux 2.6+内核中最高效的方式
    
    * rtsig:实时信号，可用在Linux 2.2.19的内核中，但不适用在高流量的系统中
    
    * /dev/poll: Solaris 7 11/99+,HP/UX 11.22+, IRIX 6.5.15+, and Tru64 UNIX 5.1A+操作系统最高效的方式
    
    * eventport: Solaris 10最高效的方式
    