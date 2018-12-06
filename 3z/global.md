###nginx.conf全局配置

####user  nobody;

    定义运行nginx服务的用户,还可以加上组,如 user nobody nobody;

####worker_processes  1;

    定义nginx子进程数量，即提供服务的进程数量，该数值建议和服务cpu核数保持一致。
    除了可以定义数字外，还可以定义为auto，表示让系统自动调整。

####error_log  logs/error.log;

    定义错误日志的路径，可以是相对路径（相对prefix路径的），也可以是绝对路径。
    该配置可以在此处定义，也可以定义到http、server、location里

####error_log  logs/error.log  notice;

    定义错误日志路径以及日志级别.
    错误日志级别：常见的错误日志级别有[debug|info|notice|warn|error|crit|alert|emerg]，级别越高记录的信息越少。
    如果不定义默认是error


####pid        logs/nginx.pid;

    定义nginx进程pid文件所在路径，可以是相对路径，也可以是绝对路径。

####worker_rlimit_nofile 100000;
  
    定义nginx最多打开文件数限制。如果没设置的话，这个值为操作系统（ulimit -n）的限制保持一致。
    把这个值设高，nginx就不会有“too many open files”问题了。
