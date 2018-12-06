####nginx的return指令

    该指令一般用于对请求的客户端直接返回响应状态码。在该作用域内return后面的所有nginx配置都是无效的。
    可以使用在server、location以及if配置中。
    
    除了支持跟状态码，还可以跟字符串或者url链接。
    
#####直接返回状态码

    示例1：
    server{
        listen 80;
        server_name www.aming.com;
        return 403;
        rewrite /(.*) /abc/$1;  //该行配置不会被执行。
    }
    
    示例2：
    server {
    .....
    
    if ($request_uri ~ "\.htpasswd|\.bak")
    {
        return 404;
        rewrite /(.*) /aaa.txt;  //该行配置不会被执行。
    }
    //如果下面还有其他配置，会被执行。
    .....
    }
    
#####返回字符串

    示例3：
    server{
        listen 80;
        server_name www.aming.com;
        return 200 "hello";
    }
    说明：如果要想返回字符串，必须要加上状态码，否则会报错。
    还可以支持json数据
    
    示例4：
	location ^~ /aming {
        default_type application/json ;
        return 200  '{"name":"aming","id":"100"}';
    }
    
    也支持写一个变量
    
    示例5：
    location /test {
	    return 200 "$host $request_uri";
	}

#####返回url

    示例6：
    server{
        listen 80;
        server_name www.aming.com;
        return http://www.aminglinux.com/123.html;
        rewrite /(.*) /abc/$1;  //该行配置不会被执行。
    }
    注意：return后面的url必须是以http://或者https://开头的。
    
    
#####生成场景实战

    背景：网站被黑了，凡是在百度点击到本网站的请求，全部都跳转到了一个赌博网站。
    通过nginx解决：
    if ($http_referer ~ 'baidu.com') 
    {
        return 200 "<html><script>window.location.href='//$host$request_uri';</script></html>";
    }
    
    如果写成：
    return http://$host$request_uri; 在浏览器中会提示“重定向的次数过多”。
    
    