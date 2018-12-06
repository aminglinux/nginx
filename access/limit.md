####Nginx的限速
```
可以通过ngx_http_limit_conn_module和ngx_http_limit_req_module模块来实现限速的功能。
```

#####ngx_http_limit_conn_module
```
该模块主要限制下载速度。
```
######1. 并发限制
```
配置示例
http
{
    ...
    limit_conn_zone $binary_remote_addr zone=aming:10m;
    ...
    server
    {
        ...
        limit_conn aming 10;
        ...   
    }
}
说明：首先用limit_conn_zone定义了一个内存区块索引aming，大小为10m，它以$binary_remote_addr作为key。
该配置只能在http里面配置，不支持在server里配置。

limit_conn 定义针对aming这个zone，并发连接为10个。在这需要注意一下，这个10指的是单个IP的并发最多为10个。
```
######2. 速度限制
```
location ~ /download/ {
    ...
    limit_rate_after 512k;
    limit_rate 150k;
    ...
}
说明：limit_rate_after定义当一个文件下载到指定大小（本例中为512k）之后开始限速；
limit_rate 定义下载速度为150k/s。

注意：这两个参数针对每个请求限速。
```

#####ngx_http_limit_req_module

```
该模块主要用来限制请求数。
```
######1. limit_req_zone
```
语法: limit_req_zone $variable zone=name:size rate=rate;
默认值: none
配置段: http

设置一块共享内存限制域用来保存键值的状态参数。 特别是保存了当前超出请求的数量。 
键的值就是指定的变量（空值不会被计算）。
如limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

说明：区域名称为one，大小为10m，平均处理的请求频率不能超过每秒一次,键值是客户端IP。
使用$binary_remote_addr变量， 可以将每条状态记录的大小减少到64个字节，这样1M的内存可以保存大约1万6千个64字节的记录。
如果限制域的存储空间耗尽了，对于后续所有请求，服务器都会返回 503 (Service Temporarily Unavailable)错误。
速度可以设置为每秒处理请求数和每分钟处理请求数，其值必须是整数，
所以如果你需要指定每秒处理少于1个的请求，2秒处理一个请求，可以使用 “30r/m”。

```
######2. limit_req
```
语法: limit_req zone=name [burst=number] [nodelay];
默认值: —
配置段: http, server, location

设置对应的共享内存限制域和允许被处理的最大请求数阈值。 
如果请求的频率超过了限制域配置的值，请求处理会被延迟，所以所有的请求都是以定义的频率被处理的。 
超过频率限制的请求会被延迟，直到被延迟的请求数超过了定义的阈值，
这时，这个请求会被终止，并返回503 (Service Temporarily Unavailable) 错误。

这个阈值的默认值为0。如：
limit_req_zone $binary_remote_addr zone=aming:10m rate=1r/s;
server {
    location /upload/ {
        limit_req zone=aming burst=5;
    }
}

限制平均每秒不超过一个请求，同时允许超过频率限制的请求数不多于5个。

如果不希望超过的请求被延迟，可以用nodelay参数,如：

limit_req zone=aming burst=5 nodelay;
```

#####示例

```
http {
    limit_req_zone $binary_remote_addr zone=aming:10m rate=1r/s;

    server {
        location  ^~ /download/ {  
            limit_req zone=aming burst=5;
        }
    }
}
```

#####设定白名单IP
```
如果是针对公司内部IP或者lo（127.0.0.1）不进行限速，如何做呢？这就要用到geo模块了。

假如，预把127.0.0.1和192.168.100.0/24网段设置为白名单，需要这样做。
在http { }里面增加：
geo $limited {
    default 1;
    127.0.0.1/32 0;
    192.168.100.0/24 0;
}

map $limited $limit {
	1 $binary_remote_addr;
    0 "";
}

原来的 “limit_req_zone $binary_remote_addr ” 改为“limit_req_zone $limit”

完整示例：

http {
	geo $limited {
		default 1;
		127.0.0.1/32 0;
		192.168.100.0/24 0;
	}

	map $limited $limit {
		1 $binary_remote_addr;
		0 "";
	}
    
    limit_req_zone $limit zone=aming:10m rate=1r/s;

    server {
        location  ^~ /download/ {  
            limit_req zone=aming burst=5;
        }
    }
}
```