####Nginx的错误日志
```
Nginx错误日志平时不用太关注，但是一旦出了问题，就需要借助错误日志来判断问题所在。

配置参数格式：error_log /path/to/log level;
```

#####Nginx错误日志级别
```
常见的错误日志级别有debug | info | notice | warn | error | crit | alert | emerg
级别越高记录的信息越少，如果不定义，默认级别为error.

它可以配置在main、http、server、location段里。

如果在配置文件中定义了两个error_log，在同一个配置段里的话会产生冲突，所以同一个段里只允许配置一个error_log。
但是，在不同的配置段中出现是没问题的。
```

#####Nginx错误日志示例
```
error_log  /var/log/nginx/error.log crit;

如果要想彻底关闭error_log，需要这样配置
error_log /dev/null;
```
