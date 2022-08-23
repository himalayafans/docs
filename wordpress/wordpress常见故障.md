# WordPress 常见故障

### 发布文章时，提示发布失败，您可能已掉线

原因：域名通过在cloudflare中被设置为加密模式（HTTPS），而WordPress网站设置中为HTTP方式。在chrome浏览器中有一个安全限制，HTTPS的网站不能调用HTTP方式的API。

解决方案：在WordPress网站设置中，将域名设置为相同，即https开头。

![图片](https://user-images.githubusercontent.com/50398598/181907980-fe82939c-6b0e-4da9-b3e0-cc62a0e6cdad.jpg)



### WordPress登录后台时，提示：重定向次数过多

原因：修改WordPress后台的网站地址，将HTTP修改成HTTPS，导致不能登录。

解决方案：修改wp-config.php文件，在文件开头添加以下代码：

```php
$_SERVER['HTTPS'] = 'on';
define('FORCE_SSL_LOGIN', true);
define('FORCE_SSL_ADMIN', true);
```

### 上传主题时报错：413 Request Entity Too Large

原因：Nginx与PHP默认上传文件有限制，需要修改配置文件增加上传文件大小。

#### 配置Nginx

编辑nginx.conf文件

```
vim /etc/nginx/nginx.conf
```

在http段中，添加以下代码：

```
client_max_body_size 20m;
```

![Snipaste_2022-07-31_03-34-05](https://user-images.githubusercontent.com/50398598/181994422-13a1290d-edfd-4b35-a1ae-ab715fbadb27.jpg)

重启nginx

```
sudo systemctl restart nginx
```

#### 配置PHP

编辑php.ini配置文件

```
vim /etc/php/8.1/fpm/php.ini
```

在配置文件中，找到以下配置项，将其值改为：

```ini
; 最大文件上传大小
upload_max_filesize = 20M
; 最大post体大小
post_max_size = 20M
; 最大执行时间，即超时时间(下面配置了300秒)
max_execution_time = 300
```

重启PHP：

```
sudo systemctl restart php8.1-fpm
```

