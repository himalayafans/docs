# WordPress 环境搭建（Ubuntu 22）

## 升级系统（仅首次）

更新服务器存储库

```
sudo apt update
```

将服务器包升级到最新版本

```
sudo apt -y upgrade
```

## 安装MySQL 8

1. 该命令将自动安装MySQL 8和所必须的依赖包

```
sudo apt install mysql-server
```

2. 验证安装

```
mysql -V
```

3. 进入MySQL SHELL

```
mysql -u root -p
```

4. 默认没有密码，设置一个新密码（许多早期版本的方式已失效，目前按下列方式修改成功）

```
 ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';
 flush privileges;
```

> MySQL版本：mysql Ver 8.0.30-0ubuntu0.22.04.1 for Linux on x86_64 ((Ubuntu))

5. 验证密码是否修改成功（可以看到root那一行的 authentication_string不为空即可）

```
 use mysql;
 select Host, User, authentication_string from user;
```

修改成功后，exit退出MySQL shell，重新运行mysql -u root -p将提示需要输入密码。

6. 创建数据库

```
create database wordpress;
```

7. 相关的服务命令

```
sudo service mysql stop
sudo service mysql start
sudo service mysql restart
sudo service mysql status
```

## 安装Nginx

1. 安装命令（apt 会帮你把 Nginx 和所必备的依赖安装到服务器中）

```
sudo apt install nginx 
```

2. 调整防火墙

在测试 Nginx 之前，需要调整防火墙，让他允许 Nginx 服务通过。Nginx `ufw` 在安装时会把自身注册成为服务。

```
sudo ufw app list
```

会输出以下结果：

```
Available applications:
  Nginx Full 同时打开端口 80（正常、未加密的 Web 流量）和端口 443（TLS/SSL 加密流量）
  Nginx HTTP 仅打开端口 80（正常、未加密的网络流量）
  Nginx HTTPS 仅打开端口 443（TLS/SSL 加密流量）
  OpenSSH
```

输入以下命令，让配置生效

```
sudo ufw allow 'Nginx Full'
```

验证配置是否正确

```
sudo ufw status
```

验证nginx是否启动成功（此时输入服务器IP，应该能看到默认页面）

```
systemctl status nginx
```

  设置为开机启动

```
sudo systemctl enable nginx --now
```

管理 Nginx 进程的命令

```
sudo systemctl stop nginx  #停止Web服务器
sudo systemctl start nginx #启动web服务器
sudo systemctl restart nginx #停止并启动web服务器
sudo systemctl reload nginx #如果配置更改，Nginx可以重新加载而不会断开连接
sudo systemctl disable nginx #禁止自动启动
sudo systemctl enable nginx #启用自动启动
```

## 安装PHP

查看仓库内PHP版本（默认为PHP 8.1）

```
apt-cache show php
```

安装 PHP 8.1 FPM for Nginx

```
sudo apt install php8.1-fpm php8.1-common php8.1-mysql php8.1-xml php8.1-xmlrpc php8.1-curl php8.1-gd php8.1-imagick php8.1-cli php8.1-dev php8.1-imap php8.1-mbstring php8.1-opcache php8.1-soap php8.1-zip php8.1-redis php8.1-intl -y
```

NGINX 需要**php-fpm**（快速进程管理器）包来处理 WordPress 安装的 PHP 页面。请记住，一旦 PHP 安装结束，FPM 服务将自动运行。

一旦安装完成，下面命令将可用：

```
php-fpm8.1 -v
php -verson
```

## 配置PHP站点

创建站点目录

```
sudo mkdir /var/www/html/wordpress
```

创建Nginx配置文件

```
sudo vim /etc/nginx/sites-available/wordpress.conf
```

内容如下

```
server {             
            listen 80;             
            root /var/www/html/wordpress;             
            index index.php index.html;             
            server_name 购买的域名;             

            access_log /var/log/nginx/www.access.log;             
            error_log /var/log/nginx/www.error.log;
            client_max_body_size 20M;

            location / {                            
                 try_files $uri $uri/ /index.php?$args;
            }

            location ~ \.php$ {
            include snippets/fastcgi-php.conf;                            
                 fastcgi_pass unix:/run/php/php8.1-fpm.sock;             
            }

            location ~ /\.ht {                            
                 deny all;             
            }

            location = /favicon.ico {                            
                 log_not_found off;                            
                 access_log off;             
            }             

            location = /robots.txt {                            
                 allow all;                            
                 log_not_found off;                            
                 access_log off;             
            }             

            location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {                            
                 expires max;                            
                 log_not_found off;             
            }
}
```

检查配置是否正确

```
sudo nginx -t
```

重新载入配置

```
sudo systemctl reload nginx
```

创建测试用的php文件

```
vim /var/www/html/wordpress/info.php
```

PHP文件内容如下：

```php
<?php
phpinfo();
?>
```

启用站点（创建一个符号链接）

```
sudo ln -s /etc/nginx/sites-available/wordpress.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

在配置域名后，应该能访问该域名（域名/info.php），如果是在cloudflare里配置，需要将加密模式为 灵活，否则将访问失败。

![图片](https://user-images.githubusercontent.com/50398598/181902836-ff6b95dd-8ee6-409e-99e8-c4b32ac5a379.jpg)

## 安装WordPress

下载wordpress

```
wget https://wordpress.org/latest.zip
```

解压到指定目录

```
sudo unzip latest.zip -d /var/www/html/
```

## 配置WordPress

进入WordPress目录

```
cd /var/www/html/wordpress/
```

重命名配置文件

```
sudo mv wp-config-sample.php wp-config.php
```

编辑配置文件

```
sudo vim wp-config.php
```

> 主要修改：DB_NAME、DB_USER、DB_PASSWORD

设置目录权限

```
sudo chown -R www-data:www-data /var/www/html/wordpress/
sudo find /var/www/html/wordpress -type d -exec chmod 755 {} \;
sudo find /var/www/html/wordpress -type f -exec chmod 644 {} \;
```

配置完成，访问域名应该能进入WordPress的安装界面
