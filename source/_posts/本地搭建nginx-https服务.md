---
title: 本地搭建nginx https服务
date: 2018-10-10 16:43:19
categories: 
- nginx
tags:
- nginx
- https
---
## 前言
  前几天，pm反馈页面被运营商劫持，出现了小广告，我们检查了项目内并没有http链接，理论上都是https应该不回被劫持，后来发现反馈的页面是外部的投放资源是http，另外我们的nginx配置的兜底策略关闭掉直接访问页面的http链接。所以需要在本地验证一波https的nginx配置。

## 一、相关原理

1. 对称加密
2. 非对称加密
3. https采用的是对称加密和非对称加密混合的方式
4. https tls握手过程
5. https session Key
6. openssl
   
## 二、生成本地证书

1. 具体命令
```
// a. 准备工作，这里新建目录在～，遇到一个建desktop，后面的nginx404的坑原因不明
cd ~ && mkdir sshkey && cd sshkey && mkdir demoCA && cd demoCA && touch index.txt && touch serial && echo "01">./serial && mkdir newcerts
// b. 生成ca.key CA私钥
openssl genrsa -des3 -out ca.key 2048
// c. 生成ca.crt CA根证书（公钥）
openssl req -new -x509 -days 7305 -key ca.key -out ca.crt
// d. 生成www.example.com证书私钥：
openssl genrsa -des3 -out www.example.com.pem 1024
// e. 制作解密后的www.example.com证书私钥，一定要填写common.name不然生成东西为空
openssl rsa -in www.example.com.pem -out www.example.com.key
// f. 生成签名请求
openssl req -new -key www.example.com.pem -out www.example.com.csr
// g. 用ca进行签名
openssl ca -policy policy_anything -days 1460 -cert ca.crt -keyfile ca.key -in www.example.com.csr -out www.example.com.crt
```
2. 遇到问题
```
// problem default_ca
Using configuration from /private/etc/ssl/openssl.cnf
variable lookup failed for ca::default_ca
// 将openssl安装目录下的openssl.cnf 拷贝到配置目录 
cp /usr/local/etc/openssl/openssl.cnf /private/etc/ssl/openssl.cnf
//修改路径 置文件的dir的文件路径为之前创建的demoCA文件路径
dir >> = /Users/weiqian/sshkey/demoCA > >
```
3. 具体操作演示
```
➜  ~ cd ~ && mkdir sshkey &&  cd sshkey && mkdir demoCA && cd demoCA && touch index.txt && touch serial && echo "01">./serial && mkdir newcerts
➜  demoCA openssl genrsa -des3 -out ca.key 2048

Generating RSA private key, 2048 bit long modulus
................................................+++
..............+++
e is 65537 (0x10001)
Enter pass phrase for ca.key:
Verifying - Enter pass phrase for ca.key:
➜  demoCA openssl req -new -x509 -days 7305 -key ca.key -out ca.crt
Enter pass phrase for ca.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:cn
State or Province Name (full name) []:shenzhen
Locality Name (eg, city) []:shenzhen
Organization Name (eg, company) []:xxxxx
Organizational Unit Name (eg, section) []:dp
Common Name (eg, fully qualified host name) []:www.example.com
Email Address []:
➜  demoCA openssl genrsa -des3 -out www.example.com.pem 1024
Generating RSA private key, 1024 bit long modulus
..........++++++
.......................++++++
e is 65537 (0x10001)
Enter pass phrase for www.example.com.pem:
Verifying - Enter pass phrase for www.example.com.pem:
➜  demoCA openssl rsa -in www.example.com.pem -out www.example.com.key
Enter pass phrase for www.example.com.pem:
writing RSA key
➜  demoCA openssl req -new -key www.example.com.pem -out www.example.com.csr
Enter pass phrase for www.example.com.pem:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) []:cn
State or Province Name (full name) []:shenzhen
Locality Name (eg, city) []:shenzhen
# testoid2=${testoid1}.5.6
Organization Name (eg, company) []:xxxxx
Organizational Unit Name (eg, section) []:dp
Common Name (eg, fully qualified host name) []:www.example.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
➜  demoCA openssl ca -policy policy_anything -days 1460 -cert ca.crt -keyfile ca.key -in www.example.com.csr -out www.example.com.crt
Using configuration from /private/etc/ssl/openssl.cnf
variable lookup failed for ca::default_ca
140735671473096:error:0E06D06C:configuration file routines:NCONF_get_string:no value:/BuildRoot/Library/Caches/com.apple.xbs/Sources/libressl/libressl-22.50.2/libressl/crypto/conf/conf_lib.c:323:group=ca name=default_ca
➜  demoCA cp /usr/local/etc/openssl/openssl.cnf /private/etc/ssl/openssl.cnf
cp: /private/etc/ssl/openssl.cnf: Permission denied
➜  demoCA sudo cp /usr/local/etc/openssl/openssl.cnf /private/etc/ssl/openssl.cnf
Password:
➜  demoCA pwd
/Users/weiqian/sshkey/demoCA
➜  demoCA vim /private/etc/ssl/openssl.cnf
➜  demoCA sudo vim /private/etc/ssl/openssl.cnf
➜  demoCA openssl ca -policy policy_anything -days 1460 -cert ca.crt -keyfile ca.key -in www.example.com.csr -out www.example.com.crt
Using configuration from /private/etc/ssl/openssl.cnf
Enter pass phrase for ca.key:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Oct 10 15:32:09 2018 GMT
            Not After : Oct  9 15:32:09 2022 GMT
        Subject:
            countryName               = cn
            stateOrProvinceName       = shenzhen
            localityName              = shenzhen
            organizationName          = xxxxx
            organizationalUnitName    = dp
            commonName                = www.example.com
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                F1:2E:2B:F6:C9:8E:3F:34:B9:A7:B8:96:45:92:21:47:35:54:F2:84
            X509v3 Authority Key Identifier:
                DirName:/C=cn/ST=shenzhen/L=shenzhen/O=xxxxx/OU=dp/CN=www.example.com
                serial:F8:D7:C0:D0:73:1E:B4:91

Certificate is to be certified until Oct  9 15:32:09 2022 GMT (1460 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
➜  demoCA ls
ca.crt              index.txt.attr      serial              www.example.com.csr
ca.key              index.txt.old       serial.old          www.example.com.pem
index.txt           newcerts            www.example.com.crt
➜  demoCA sudo vim /etc/hosts
Password:
➜  demoCA cd ..
➜  sshkey touch nginx-https.conf
➜  sshkey vim nginx-https.conf
➜  sshkey vim index.html
➜  sshkey sudo nginx -s stop
➜  sshkey sudo  nginx -c  /Users/weiqian/sshkey/nginx-https.conf
```

## 三、nginx配置
1. 修改host
```
sudo vim /etc/hosts
127.0.0.1 www.example.com
```
2. nginx-https.conf
```
user root owner;
worker_processes 1;
events {
 worker_connections 1024;
}
http{
  server{
    listen 80;
    server_name www.example.com;
    root /Users/weiqian/sshkey;
    location / {
      return   301  https://$host$request_uri;
    }
  }

 server {
    
    listen 443 ssl;
    server_name www.example.com;
    root /Users/weiqian/sshkey;
    ssl_certificate       /Users/weiqian/sshkey/demoCA/www.example.com.crt; 
    ssl_certificate_key   /Users/weiqian/sshkey/demoCA/www.example.com.key; 
    location / {
      index index.html;
    }  
    location /produk-digital/m {
        try_files     $uri      /digital-product/m/index.html;
    }
    location = /digital-product/m/index.html {
        alias       /Users/weiqian/sshkey/index.html;
    }
  } 
}
```
3. 运行nginx
```
sudo nginx -s stop
sudo  nginx -c  /Users/weiqian/sshkey/nginx-https.conf
```
访问 www.example.com
## 四、chrome将证书设置为始终信任
1. 打开浏览器，访问 www.example.com，点击=>tab不安全=>证书=>图标拖到桌面
2. 打开钥匙串，把桌面的证书拖进去[系统|证书]，选择始终信任
3. 重新访问www.example.com，页面内容显示test https example



