## 1. 监听socket文件

```shell
# 配置php-fpm的监听方式
vim /usr/local/php/etc/php-fpm.conf
[www]
listen = /dev/shm/php-cgi.sock
listen.backlog = -1
listen.allowed_clients = 127.0.0.1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www

```





```shell
# 配置nginx
vim /usr/local/nginx/conf/nginx.conf
...

location ~ \.php(.*)$ {
	fastcgi_pass /dev/shm/php-cgi.sock;
	fastcgi_index index.php;
	fastcgi_param SCRIPT_FILENAME 			$document_root$fastcgi_script_name;
	include fastcgi_params;
}

```









## 2. 监听网络地址

```shell
# 配置php-fpm的监听方式
vim /usr/local/php/etc/php-fpm.conf

[www]
listen = 127.0.0.1:9000
listen.backlog = -1
listen.allowed_clients = 127.0.0.1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www

```



```shell
# 配置nginx
vim /usr/local/nginx/conf/nginx.conf
...

location ~ \.php(.*)$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include  fastcgi_params;
}
```



