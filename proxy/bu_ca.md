####nginx的proxy_buffering和proxy_cache
```
两个都是nginx代理中内存设置相关的参数。
```

#####proxy_buffering设置
```
proxy_buffering主要是实现被代理服务器的数据和客户端的请求异步。
为了方便理解，我们定义三个角色，A为客户端，B为代理服务器，C为被代理服务器。

当proxy_buffering开启，A发起请求到B，B再到C，C反馈的数据先到B的buffer上，
然后B会根据proxy_busy_buffer_size来决定什么时候开始把数据传输给A。在此过程中，如果所有的buffer被写满，
数据将会写入到temp_file中。

相反，如果proxy_buffering关闭，C反馈的数据实时地通过B传输给A。
```

######以下配置，都是针对每一个http请求的。
```
1. proxy_buffering  on;
该参数设置是否开启proxy的buffer功能，参数的值为on或者off。
如果这个设置为off，那么proxy_buffers和proxy_busy_buffers_size这两个指令将会失效。 
但是无论proxy_buffering是否开启，proxy_buffer_size都是生效的

2. proxy_buffer_size  4k;
该参数用来设置一个特殊的buffer大小的。
从被代理服务器（C）上获取到的第一部分响应数据内容到代理服务器（B）上，通常是header，就存到了这个buffer中。 
如果该参数设置太小，会出现502错误码，这是因为这部分buffer不够存储header信息。建议设置为4k。

3. proxy_buffers  8  4k;
这个参数设置存储被代理服务器上的数据所占用的buffer的个数和每个buffer的大小。
所有buffer的大小为这两个数字的乘积。

4. proxy_busy_buffer_size 16k;
在所有的buffer里，我们需要规定一部分buffer把自己存的数据传给A，这部分buffer就叫做busy_buffer。
proxy_busy_buffer_size参数用来设置处于busy状态的buffer有多大。

对于B上buffer里的数据何时传输给A，我个人的理解是这样的：
1）如果完整数据大小小于busy_buffer大小，当数据传输完成后，马上传给A；
2）如果完整数据大小不少于busy_buffer大小，则装满busy_buffer后，马上传给A；

5. proxy_temp_path
语法：proxy_temp_path  path [level1 level2 level3]
定义proxy的临时文件存在目录以及目录的层级。

例：proxy_temp_path /usr/local/nginx/proxy_temp 1 2;
其中/usr/local/nginx/proxy_temp为临时文件所在目录，1表示层级1的目录名为一个数字(0-9),2表示层级2目录名为2个数字(00-99)

6. proxy_max_temp_file_size
设置临时文件的总大小，例如 proxy_max_temp_file_size 100M;

7. proxy_temp_file_wirte_size
设置同时写入临时文件的数据量的总大小。通常设置为8k或者16k。

```
#####proxy_buffer示例
```
server
{
    listen 80;
    server_name www.aminglinux.com;
    proxy_buffering on;
	proxy_buffer_size 4k;
    proxy_buffers 2 4k;
    proxy_busy_buffers_size 4k;
    proxy_temp_path /tmp/nginx_proxy_tmp 1 2;
	proxy_max_temp_file_size 20M;
	proxy_temp_file_write_size 8k;
	
	location /
	{
	    proxy_pass      http://192.168.10.110:8080/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

	}

}
```

#####proxy_cache设置
```
proxy_cache将从C上获取到的数据根据预设规则存放到B上（内存+磁盘）留着备用，
A请求B时，B会把缓存的这些数据直接给A，而不需要再去向C去获取。

proxy_cache相关功能生效的前提是，需要设置proxy_buffering on;

```
######proxy_cache主要参数
```
1. proxy_cache
语法：proxy_cache zone|off

默认为off，即关闭proxy_cache功能，zone为用于存放缓存的内存区域名称。
例：proxy_cache my_zone;

从nginx 0.7.66版本开始，proxy_cache机制开启后会检测被代理端的HTTP响应头中的"Cache-Control"、"Expire"头域。
如，Cache-Control为no-cache时，是不会缓存数据的。

2. proxy_cache_bypass 
语法：proxy_cache_bypass string;

该参数设定，什么情况下的请求不读取cache而是直接从后端的服务器上获取资源。
这里的string通常为nginx的一些变量。

例：proxy_cahce_bypass $cookie_nocache $arg_nocache$arg_comment;
意思是，如果$cookie_nocache $arg_nocache$arg_comment这些变量的值只要任何一个不为0或者不为空时，
则响应数据不从cache中获取，而是直接从后端的服务器上获取。

3. proxy_no_cache
语法：proxy_no_cache string;

该参数和proxy_cache_bypass类似，用来设定什么情况下不缓存。

例：proxy_no_cache $cookie_nocache $arg_nocache $arg_comment;
表示，如果$cookie_nocache $arg_nocache $arg_comment的值只要有一项不为0或者不为空时，不缓存数据。

4. proxy_cache_key
语法：proxy_cache_key string;

定义cache key，如： proxy_cache_key $scheme$proxy_host$uri$is_args$args; （该值为默认值，一般不用设置）

5. proxy_cache_path
语法：proxy_cache_path path [levels=levels] keys_zone=name:size  [inactive=time] [max_size=size] 

path设置缓存数据存放的路径；

levels设置目录层级，如levels=1:2，表示有两级子目录,第一个目录名取md5值的倒数第一个值，第二个目录名取md5值的第2和3个值。如下图：
```
![image](https://coding.net/u/aminglinux/p/nginx/git/raw/master/proxy/nginx_cache_levels.png)
```
keys_zone设置内存zone的名字和大小，如keys_zone=my_zone:10m

inactive设置缓存多长时间就失效，当硬盘上的缓存数据在该时间段内没有被访问过，就会失效了，该数据就会被删除，默认为10s。

max_size设置硬盘中最多可以缓存多少数据，当到达该数值时，nginx会删除最少访问的数据。

例：proxy_cache_path /data/nginx_cache/ levels=1:2 keys_zone=my_zone:10m inactive=300s max_size=5g

```
#####proxy_cache示例
```
http 
{
    ...;
    
    proxy_cache_path /data/nginx_cache/ levels=1:2 keys_zone=my_zone:10m inactive=300s max_size=5g;
    
    ...;
    
    server
    {
        listen 80;
        server_name www.aminglinux.com;
        proxy_buffering on;
    	proxy_buffer_size 4k;
        proxy_buffers 2 4k;
        proxy_busy_buffers_size 4k;
        proxy_temp_path /tmp/nginx_proxy_tmp 1 2;
    	proxy_max_temp_file_size 20M;
    	proxy_temp_file_write_size 8k;
	
	
	
	    location /
	    {
	        proxy_cache my_zone;
	        proxy_pass      http://192.168.10.110:8080/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

	    }

    }
}

说明：核心配置为proxy_cache_path那一行。
```
