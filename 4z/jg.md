####Nginx架构

    Nginx服务器使用 master/worker 多进程模式。
    主进程(Master process)启动后，会接收和处理外部信号；
    主进程启动后通过fork() 函数产生一个或多个子进程(work process)，每个子进程会进行进程初始化、
    模块调用以及对事件的接收和处理等工作。

![image](https://coding.net/u/aminglinux/p/nginx/git/raw/master/4z/nginx_jg.png)

#####主进程

    主要功能是和外界通信和对内部其他进程进行管理，具体来说有以下几点：
    
    * 读取Nginx配置文件并验证其有效性和正确性
    
    * 建立、绑定和关闭socket
    
    * 按照配置生成、管理工作进程
    
    * 接收外界指令，比如重启、关闭、重载服务等指令
    
    * 日志文件管理
    
#####子进程（worker process)

    是由主进程生成，生成数量可以在配置文件中定义。该进程主要工作有：
    
    * 接收客户端请求
    
    * 将请求依次送入各个功能模块进行过滤处理
    
    * IO调用，获取响应数据
    
    * 与后端服务器通信，接收后端服务器处理结果
    
    * 数据缓存，访问缓存索引，查询和调用缓存数据
    
    * 发送请求结果，响应客户端请求
    
    * 接收主进程指令，如重启、重载、退出等
    
![img](https://coding.net/u/aminglinux/p/nginx/git/raw/master/4z/nginx_m.jpg)