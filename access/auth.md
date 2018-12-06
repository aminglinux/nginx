####Nginx的用户认证
```
当访问一些私密资源时，最好配置用户认证，增加安全性。
```
#####步骤和示例
* 安装httpd
```
yum install -y httpd
```

* 使用htpasswd生产密码文件
```
htpasswd -c /usr/local/nginx/conf/htpasswd aming
```

* 配置nginx用户认证
```
    location  /admin/
    {
        auth_basic              "Auth";
        auth_basic_user_file   /usr/local/nginx/conf/htpasswd;
    }
```

* 测试
```
curl -uaming:passwd www.aminglinux.com/admin/1.html
```