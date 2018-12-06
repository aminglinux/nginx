####rewrite中的break和last

    两个指令用法相同，但含义不同，需要放到rewrite规则的末尾，用来控制重写后的链接是否继续被nginx配置执行(主要是rewrite、return指令)。
    
    示例1（连续两条rewrite规则）：
    server{
        listen 80; 
        server_name test.com;
	    root /tmp/123.com;

	    rewrite /1.html /2.html ;
	    rewrite /2.html /3.html ;
        
    }
    当我们请求1.html时，最终访问到的是3.html，两条rewrite规则先后执行。
    
    
#####break和last在location {}外部

    格式：rewrite xxxxx  break;
    
    示例2（增加break）：
    server{
        listen 80; 
        server_name test.com;
	    root /tmp/123.com;

	    rewrite /1.html /2.html break;
	    rewrite /2.html /3.html;
    }
    当我们请求1.html时，最终访问到的是2.html
    说明break在此示例中，作用是不再执行break以下的rewrite规则。
    
    但，当配置文件中有location时，它还会去执行location{}段的配置（请求要匹配该location）。
    
    示例3（break后面还有location段）：
    server{
        listen 80; 
        server_name test.com;
	    root /tmp/123.com;

	    rewrite /1.html /2.html break;
	    rewrite /2.html /3.html;
        location /2.html {
            return 403;
        }
    }
    当请求1.html时，最终会返回403状态码，说明它去匹配了break后面的location{}配置。
    
    以上2个示例中，可以把break替换为last，它们两者起到的效果一模一样。
    
#####当break和last在location{}里面
    
    示例4（什么都不加）：
    server{
        listen 80; 
        server_name test.com;
        root /tmp/123.com;
        
        location / {
            rewrite /1.html /2.html;
            rewrite /2.html /3.html;
        }
        location /2.html
        {
            rewrite /2.html /a.html;
        }
        location /3.html
        {
            rewrite /3.html /b.html;
        }
    }
    当请求/1.html，最终将会访问/b.html，连续执行location /下的两次rewrite，跳转到了/3.html，然后又匹配location /3.html
    
    示例5（增加break）：
    server{
        listen 80; 
        server_name test.com;
        root /tmp/123.com;
        
        location / {
            rewrite /1.html /2.html break;
            rewrite /2.html /3.html;
        }
        location /2.html
        {
            rewrite /2.html /a.html;
        }
        location /3.html
        {
            rewrite /3.html /b.html;
        }
    }
    当请求/1.html，最终会访问/2.html
    在location{}内部，遇到break，本location{}内以及后面的所有location{}内的所有指令都不再执行。

    
    示例6（增加last）:
    server{
        listen 80; 
        server_name test.com;
        root /tmp/123.com;
        
        location / {
            rewrite /1.html /2.html last;
            rewrite /2.html /3.html;
        }
        location /2.html
        {
            rewrite /2.html /a.html;
        }
        location /3.html
        {
            rewrite /3.html /b.html;
        }
    }
    当请求/1.html，最终会访问/a.html
    在location{}内部，遇到last，本location{}内后续指令不再执行，而重写后的url再次从头开始，从头到尾匹配一遍规则。
    

#####结论

* 当rewrite规则在location{}外，break和last作用一样，遇到break或last后，其后续的rewrite/return语句不再执行。但后续有location{}的话，还会近一步执行location{}里面的语句,当然前提是请求必须要匹配该location。
* 当rewrite规则在location{}里，遇到break后，本location{}与其他location{}的所有rewrite/return规则都不再执行。
* 当rewrite规则在location{}里，遇到last后，本location{}里后续rewrite/return规则不执行，但重写后的url再次从头开始执行所有规则，哪个匹配执行哪个。

    
    