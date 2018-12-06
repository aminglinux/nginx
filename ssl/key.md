#####生成CA根证书
```
# mkdir /etc/pki/ca_test //创建CA更证书的目录

# cd /etc/pki/ca_test

# mkdir root server client newcerts  //创建几个相关的目录

# echo 01 > serial   //定义序列号为01

# echo 01 > crlnumber  //定义crl号为01

# touch index.txt  //创建index.txt

# cd ..

# vi tls/openssl.cnf  //改配置文件
default_ca     = CA_default 改为 default_ca     = CA_test
[ CA_default ] 改为 [ CA_test ]
dir             = /etc/pki/CA  改为  dir             = /etc/pki/ca_test
certificate	= $dir/cacert.pem  改为 certificate	= $dir/root/ca.crt
private_key	= $dir/private/cakey.pe 改为  private_key	= $dir/root/ca.key

# openssl genrsa -out /etc/pki/ca_test/root/ca.key  //生成私钥

# openssl req -new -key /etc/pki/ca_test/root/ca.key -out /etc/pki/ca_test/root/ca.csr   
//生成请求文件，会让我们填写一些指标,这里要注意：如果在这一步填写了相应的指标，
比如Country Name、State or Province Name、hostname。

# openssl x509 -req -days 3650 -in /etc/pki/ca_test/root/ca.csr -signkey /etc/pki/ca_test/root/ca.key -out /etc/pki/ca_test/root/ca.crt 
//生成crt文件
```

#####生成server端证书
```
# cd /etc/pki/ca_test/server

# openssl genrsa -out server.key   //生成私钥文件

# openssl req -new -key server.key -out server.csr//生成证书请求文件，填写信息需要和ca.csr中的Organization Name保持一致

# openssl ca -in server.csr -cert /etc/pki/ca_test/root/ca.crt -keyfile /etc/pki/ca_test/root/ca.key -out server.crt -days 3650  
//用根证书签名server.csr，最后生成公钥文件server.crt，此步骤会有两个地方需要输入y
Sign the certificate? [y/n]:y
1 out of 1 certificate requests certified, commit? [y/n]y

```

#####生成客户端证书
```
如果做ssl的双向认证，还需要给客户端生成一个证书，步骤和上面的基本一致
# cd /etc/pki/ca_test/client

# openssl genrsa -out  client.key  //生成私钥文件

# openssl req -new  -key client.key -out client.csr  //生成请求文件，填写信息需要和ca.csr中的Organization Name保持一致

# openssl ca -in client.csr -cert /etc/pki/ca_test/root/ca.crt -keyfile /etc/pki/ca_test/root/ca.key -out client.crt -days 3650 
//签名client.csr, 生成client.crt，此步如果出现
failed to update database
TXT_DB error number 2

需执行：
# sed -i 's/unique_subject = yes/unique_subject = no/' /etc/pki/ca_test/index.txt.attr

执行完，再次重复执行签名client.csr那个操作
```