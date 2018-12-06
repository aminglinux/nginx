####Nginx配置SSL
#####Nginx配置示例（单向）
```
cp /etc/pki/ca_test/server/server.* /usr/local/nginx/conf/
{
    listen 443 ssl;
    server_name www.aminglinux.com;
    index index.html index.php;
    root /data/wwwroot/aminglinux.com;
    ssl on;
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ALL:!DH:!EXPORT:!RC4:+HIGH:+MEDIUM:!eNULL;
    ssl_prefer_server_ciphers on;
    ...
}
```
#####配置说明
```
1. 443端口为ssl监听端口。
2. ssl on表示打开ssl支持。
3. ssl_certificate指定crt文件所在路径，如果写相对路径，必须把该文件和nginx.conf文件放到一个目录下。
4. ssl_certificate_key指定key文件所在路径。
5. ssl_protocols指定SSL协议。
6. ssl_ciphers配置ssl加密算法，多个算法用:分隔，ALL表示全部算法，!表示不启用该算法，+表示将该算法排到最后面去。
7. ssl_prefer_server_ciphers 如果不指定默认为off，当为on时，在使用SSLv3和TLS协议时，服务器加密算法将优于客户端加密算法。
```
#####Nginx配置双向认证
```
cp /etc/pki/ca_test/root/ca.crt /usr/local/nginx/conf/
配置示例：
{
    listen 443 ssl;
    server_name www.aminglinux.com;
    index index.html index.php;
    root /data/wwwroot/aminglinux.com;
    ssl on;
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ALL:!DH:!EXPORT:!RC4:+HIGH:+MEDIUM:!eNULL;
    ssl_prefer_server_ciphers on;
    ssl_client_certificate ca.crt; //这里的ca.crt是根证书公钥文件
    ssl_verify_client on;
    ...
}

```

#####客户端（浏览器）操作
```
如果不进行以下操作，浏览器会出现400错误。400 Bad Request（No required SSL certificate was sent）
首先需要将client.key转换为pfx(p12)格式

# cd /etc/pki/ca_test/client
# openssl pkcs12 -export -inkey client.key -in client.crt -out client.pfx  //这一步需要输入一个自定义密码，一会在windows上安装的时候要用到，需要记一下。

然后将client.pfx拷贝到windows下，双击即可安装。

也可以直接curl测试：
curl -k --cert /etc/pki/ca_test/client/client.crt  --key /etc/pki/ca_test/client/client.key https://www.aminglinux.com/index.html
```