####nginx location优先级

    =  高于  ^~  高于  ~* 等于 ~  高于  /
    
#####对比/和~

    示例1：
    server{
        listen 80;
        server_name www.aminglinux.com;
        root /tmp/123.com;

        location /abc/
        {
            echo "/";
        }
        location ~ 'abc'
        {
            echo "~";
        }
    }

    测试命令：curl -x127.0.0.1:80 'www.aminglinux.com/abc/1.png'
    结果是：~

#####对比~和~*

    示例2：
    server
    {
        listen 80;
        server_name www.aminglinux.com;
        root /tmp/123.com;

        location ~ 'abc'
        {
            echo '~';
        }
        location ~* 'abc'
        {
	        echo '~*';
        }
    }
    测试命令：curl -x127.0.0.1:80 'www.aminglinux.com/abc/123.html' 
    结果是：~

    示例3：
    server
    {
        listen 80;
        server_name www.aminglinux.com;
        root /tmp/123.com;

        location ~* 'abc'
        {
            echo '~*';
        }
        location ~ 'abc'
        {
	        echo '~';
        }
    }
    测试命令：curl -x127.0.0.1:80 'www.aminglinux.com/abc/123.html' 
    结果是：~*
    
    结论：~和~*优先级其实是一样的，如果两个同时出现，配置文件中哪个location靠前，哪个生效。
    
#####对比^~和~

    示例4：
    server
    {
        listen 80;
        server_name www.aminglinux.com;
        root /tmp/123.com;

        location ~ '/abc'
        {
            echo '~';
        }
        location ^~ '/abc'
        {
            echo '^~';
        }
    }

    测试命令：curl -x127.0.0.1:80 'www.aminglinux.com/abc/123.html
    结果是：^~
    
#####对比=和^~

    示例5：
    server
    {
        listen 80;
        server_name www.aminglinux.com;
        root /tmp/123.com;

        location ^~ '/abc.html'
        {
            echo '^~';
        }
        location = '/abc.html'
        {
            echo '=';
        }
    }
    
    测试命令：curl -x127.0.0.1:80 'www.aminglinux.com/abc.html
    结果是：=