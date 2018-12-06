####if指令

    格式：if (条件判断) { 具体的rewrite规则 }
    
#####条件举例

    条件判断语句由Nginx内置变量、逻辑判断符号和目标字符串三部分组成。
    其中，内置变量是Nginx固定的非自定义的变量，如，$request_method, $request_uri等。
    逻辑判断符号，有=, !=, ~, ~*, !~, !~*
    !表示相反的意思，~为匹配符号，它右侧为正则表达式，区分大小写，而~*为不区分大小写匹配。
    目标字符串可以是正则表达式，通常不用加引号，但表达式中有特殊符号时，比如空格、花括号、分号等，需要用单引号引起来。

#####示例1

    if ($request_method = POST)  //当请求的方法为POST时，直接返回405状态码
    {
	    return 405; //在该示例中并未用到rewrite规则，if中支持用return指令。
    }

#####示例2

    if ($http_user_agent ~ MSIE) //user_agent带有MSIE字符的请求，直接返回403状态码
    {
	    return 403;
    }

    如果想同时限制多个user_agent，还可以写成这样

    if ($http_user_agent ~ "MSIE|firefox|spider")
    {
	    return 403;
    }

#####示例3

    if(!-f $request_filename)  //当请求的文件不存在，将会执行下面的rewrite规则
    {
        rewrite 语句;
    }
    
#####示例4

    if($request_uri ~* 'gid=\d{9,12}/')  //\d表示数字，{9,12}表示数字出现的次数是9到12次，如gid=123456789/就是符合条件的。
    {
        rewrite 语句;
    }
    
    
