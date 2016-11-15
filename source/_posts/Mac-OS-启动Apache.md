---
title: Mac OS 启动Apache
date: 2016-11-09 15:37:49
categories: web
tags: [web,Apache]
keywords: web
---

我们经常性的需要使用局域网搭建 Web 服务器测试环境，调试一些web界面等，Mac OS X 自带了 Apache 和 PHP 环境，我们只需要简单的启动它就行了。

## 启动 Apache

* 查看 Apache 版本       
  打开终端，输入  `/httpd -v`  可以查看 Apache 版本信息。
  
  ```
  $ httpd -v
  Server version: Apache/2.4.16 (Unix)
  Server built:   Aug 22 2015 16:51:57
  $
  ```
  
* 启动 Apache   
在终端输入 sudo apachectl start 即可启动 Apache。    
启动后，在浏览器中输入 http://127.0.0.1 或 http://localhost 如果看到 It Works! 页面：
那么 Apache 就启动成功了，站点的根目录为系统级根目录 `//Library/WebServer/Documents.`                                                                                
启动后，你可以通过编辑 `//etc/apache2/httpd.conf` 文件来修改 Apache 配置。
* 停止 Apache：sudo apachectl stop
* 重启 Apache：sudo apachectl restart

## 创建用户级根目录

1. 我们也可以创建用户级根目录，更方便管理和操作。
在用户目录下创建 `/Sites` 目录，`/cd; mkdir Sites; touch Sites/.localized`，旧的 Mac 系统中如果该目录已存在，则略过。
2. `/cd /etc/apache2/users` 检查目录下是否存在 `/username.conf` 文件，`/username` 为当前用户名，如果没有则创建一个 `/sudo touch username.conf`，并修改文件权限 `/sudo chmod 644 username.conf`。
3. 创建之后，打开  `/username.conf` 文件，`/sudo vi  username.conf` 将下面的配置信息写入文件，`/username` 依然为当前用户名：

  ```
针对Apache不同版本 配置不同
For OSX 10.9 Apache 2.2
<Directory "/Users/username/Sites/">
Options Indexes MultiViews FollowSymLinks
AllowOverride All
Order allow,deny
Allow from all
</Directory>
For OSX 10.10 Apache 2.4
<Directory "/Users/username/Sites/">
AllowOverride All
Options Indexes MultiViews FollowSymLinks
Require all granted
</Directory>

  ```
  
4. 编辑 `//etc/apache2/httpd.conf` 文件，找到下列代码，并将前面的注释符号`/ #` 删除：

   ```
Include /private/etc/apache2/extra/httpd-userdir.conf
LoadModule userdir_module libexec/apache2/mod_userdir.so

   ```
   
5. 编辑 `//etc/apache2/extra/httpd-userdir.conf`文件，找到下列代码，并将前面的注释符号`/#`删除：

   ```
 Include /private/etc/apache2/users/*.conf
    ```
 
6. 重启 Apache：sudo apachectl restart

在浏览器中输入 http://127.0.0.1/~username 或 http://localhost/~username，即可测试用户目录是否工作。

## 启动 PHP

Mac OS X 也默认集成了 PHP 环境，如果测试需要用到 PHP 环境，可以通过配置手动开启。

1. 编辑 `//etc/apache2/httpd.conf` 文件，找到 `/LoadModule php5_module libexec/apache2/libphp5.so` 并删除行前的注释符号 #。
2. 重启 Apache：`/sudo apachectl restart`。
3. 现在 PHP 应该已经可以工作了，在页面中嵌入 `/<?php phpinfo(); ?>` 可以查看 PHP 信息。
4. 命在终端输入 `/php --ini` 可查看 PHP 配置文件，我们可以将 `//private/etc/php.ini.default` 复制一份命名为 `//private/etc/php.ini` 并修改配置文件，如，设置 `/display_errors = On` 打开PHP错误显示。
## 安装 MySQL
Mac OS X 没有集成 MySQL，需要自己安装，这个后续补充。

## 开启 HTTPS
如果测试需要 HTTPS 环境，如， iOS 7.1 以上的设备部署无线安装环境就必须使用 HTTPS，我们可以配置 Apache 开启 HTTPS 服务。

## 创建自签名证书
首先创建一个 ssl 目录用来存放证书

```
$ cd /etc/apache2/
$ sudo mkdir ssl
$ cd ssl
```

1. 创建主机密钥

  ```
 $ sudo ssh-keygen -f local.server.com.key
 Generating public/private rsa key pair.
 Enter passphrase (empty for no passphrase): 
 Enter same passphrase again: 
 Your identification has been saved in local.server.com.key.
 Your public key has been saved in local.server.com.key.pub.
 The key fingerprint is:
 SHA256:bNX90ww2g2GCh38Q/h68JnazkZYtnbkMEb1G5E51QWw root@XuCreamandeiMac.local
 The key's randomart image is:
 +---[RSA 2048]----+
 |         oo.o +o+|
 |        o.o+ B E.|
 |         oo.+ %  |
 |       . ..o.* B.|
 |        S  .= +.+|
 |       .   . X o.|
 |          o & =  |
 |         . = B . |
 |            . o  |
 +----[SHA256]-----+
 $
  ```
 
 这里会被要求提供一个密码用于主机密钥，可以选择任何的密码或直接留空。

 也可以使用下面的命令创建密钥：
 
 ```
 $ sudo openssl genrsa -out local.server.com.key 2048
 Generating RSA private key, 2048 bit long modulus
 ....+++
 ....+++
 e is 65537 (0x10001)
 $
 ```
 
2.  创建签署申请
  
  ```
 $ sudo openssl req -new -key local.server.com.key -out local.server.com.csr
 You are about to be asked to enter information that will be incorporated
 into your certificate request.
 What you are about to enter is what is called a Distinguished Name or a DN.
 There are quite a few fields but you can leave some blank
 For some fields there will be a default value,
 If you enter '.', the field will be left blank.
 -----
 Country Name (2 letter code) [AU]:
 State or Province Name (full name) [Some-State]:
 Locality Name (eg, city) []:
 Organization Name (eg, company) [Internet Widgits Pty Ltd]:
 Organizational Unit Name (eg, section) []:
 Common Name (e.g. server FQDN or YOUR name) []:local.server.com
 Email Address []:

 Please enter the following 'extra' attributes
 to be sent with your certificate request
 A challenge password []:
 An optional company name []:
 $
 ```
 
  系统会提示输入各项信息，由于这是自签名的证书，除了 `/Common Name (e.g. server FQDN or  YOUR name) []:` FQDN（ fully qualified domain name）必须是服务器域名或 IP 外，其他都不重要，可以随意填写或一路回车，这里作为测试使用 `/local.server.com`。
  
3. 创建SSL证书       
在生产环境中，我们需要提交证书申请（CSR）文件给证书颁发机构，由证书颁发机构提供SSL证书。
 
  ```
   $ sudo openssl x509 -req -days 365 -in local.server.com.csr -signkey local.server.com.key -out local.server.com.crt
 Signature ok
 subject=/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd/CN=local.server.com
 Getting Private key
 $
  ```
  
  我们也可以直接通过以下的命令创建证书：
  
   ```
   $ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout local.server.com.key -out local.server.com.crt
   ```
   
   OK，我们看下SSL目录下面的文件，这些文件将在后面被用到：
   
   ```
   $ ls -l
-rw-r--r--  1 root  wheel  1180 10 22 13:08 local.server.com.crt
-rw-r--r--  1 root  wheel   993 10 22 11:58 local.server.com.csr
-rw-------  1 root  wheel  1679 10 22 11:44 local.server.com.key
-rw-r--r--  1 root  wheel   408 10 22 11:44 local.server.com.key.pub
-rw-r--r--  1 root  wheel  1679 10 22 13:19 local.server.com.nopass.key
   ```
 
## 配置 SSL

1. 加载`/mod_ssl.so`，编辑 `//etc/apache2/httpd.conf` 文件，删除下列代码前的注释符号 #：

 ```
LoadModule ssl_module libexec/apache2/mod_ssl.so
 ```
 
2. 包含 `/httpd-ssl.conf` 文件，编辑 `//etc/apache2/httpd.conf` 文件，删除下列代码前的注释符号 #：

 ```
Include /private/etc/apache2/extra/httpd-ssl.conf
 ```
 
3. 添加 `/<VirtualHost>` 到 `/httpd-ssl.conf`，编辑 `//etc/apache2/extra/httpd-ssl.conf` 文件：
`/httpd-ssl.conf` 中已经有一条 `/<VirtualHost>`记录，我们将其注释掉，新建一条：

 ```
 <VirtualHost *:443>
 #General setup for the virtual host
 DocumentRoot "/Library/WebServer/Documents"
 ServerName local.server.com

 #SSL Engine Switch:
 SSLEngine on

 #Server Certificate:
 SSLCertificateFile "/etc/apache2/ssl/local.server.com.crt"

 #Server Private Key:
 SSLCertificateKeyFile "/etc/apache2/ssl/local.server.com.key"

 #SSL Engine Options:
 <FilesMatch "\.(cgi|shtml|phtml|php)$">
     SSLOptions +StdEnvVars
 </FilesMatch>
 <Directory "/Library/WebServer/CGI-Executables">
     SSLOptions +StdEnvVars
 </Directory>
 </VirtualHost>
  ```
  
4. 为了能够使用 URL 访问服务器，我们需要配置HOST，`/sudo vi /etc/hosts`，添加 `/127.0.0.1       local.server.com`

5. 检查配置文件并重启 Apache
命令行输入 `/$ sudo apachectl -t`，提示：

 ```
 AH00526: Syntax error on line 92 of /private/etc/apache2/extra/httpd-ssl.conf:
 SSLSessionCache: 'shmcb' session cache not supported (known names: ). Maybe you need to load the appropriate socache module (mod_socache_shmcb?).
 ```
 
根据提示，编辑`//etc/apache2/httpd.conf` 文件，删除下列这些代码前的注释符号 #

 ```
 LoadModule socache_shmcb_module libexec/apache2/mod_socache_shmcb.so
 ```
再次测试，显示 `/Syntax OK`：

 ```
 $ sudo apachectl -t
 Syntax OK
 ```
 
说明测试通过，重启 Apache：
 
 ```
 $ sudo apachectl restart
 ```
 
此时，就可以使用 HTTPS 访问本地服务了，在浏览器中输入 https://local.server.com/ 检查。

文／小白不是总（简书作者）
原文链接：http://www.jianshu.com/p/d006a34a343f
著作权归作者所有，转载请联系作者获得授权，并标注“简书作者”。





