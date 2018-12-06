####Nginx访问日志切割
```
如果任由访问日志写下去，日志文件会变得越来越大，甚至是写满磁盘。
所以，我们需要想办法把日志做切割，比如每天生成一个新的日志，旧的日志按规定时间删除即可。

实现日志切割可以通过写shell脚本或者系统的日志切割机制实现。
```
#####shell脚本切割Nginx日志
```
切割脚本内容：
#!/bin/bash
logdir=/var/log/nginx  //定义日志路径
prefix=`date -d "-1 day" +%y%m%d`  //定义切割后的日志前缀
cd $logdir  
for f in `ls *.log`
do
   mv $f $f-$prefix  //把日志改名
done
/bin/kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid 2>/dev/null) 2>/dev/null  //生成新的日志
bzip2 *$prefix  //压缩日志
find . -type f -mtime +180 |xargs /bin/rm -f  //删除超过180天的老日志

```
#####系统日志切割机制
```
在/etc/logrotate.d/下创建nginx文件，内容为：
/data/logs/*log {
    daily
    rotate 30
    missingok
    notifempty
    compress
    sharedscripts
    postrotate
        /bin/kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid 2>/dev/null) 2>/dev/null || :
    endscript
}

说明：
1 nginx日志在/data/logs/目录下面，日志名字以log结尾
2 daily表示每天切割
3 rotate 30表示日志保留30天
4 missingok表示忽略错误
5 notifempty表示如果日志为空，不切割
6 compress表示压缩
7 sharedscripts和endscript中间可以引用系统的命令
8 postrotate表示当切割之后要执行的命令
```
