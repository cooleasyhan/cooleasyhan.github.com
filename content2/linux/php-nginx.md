title: Nginx, Php-fpm
author: YiHan
date: 2015-11-26
sulg: nginx_php


## 安装php

```
yum install libxml2-devel bzip2-devel libcurl-devel libjpeg-devel libpng-devel freetype-devel openssl-devel  
./configure --prefix=/usr/local/php-5.6 --with-config-file-path=/usr/local/php-5.6/etc --with-bz2 --with-curl --enable-ftp --enable-sockets --disable-ipv6 --with-gd --with-freetype-dir=/usr/local --enable-gd-native-ttf --with-iconv-dir=/usr/local --enable-mbstring --enable-calendar --with-gettext --with-libxml-dir=/usr/local --with-zlib --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-mysql=mysqlnd --enable-dom --enable-xml --with-libdir=lib64 --with-openssl --enable-pdo --enable-fpm --enable-bcmath  

make
make install
```


## 安装Nginx
下载pcre, nginx
```
wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.37/pcre-8.37.tar.bz2
wget http://nginx.org/download/nginx-1.8.0.tar.gz

tar -xvf ......

cd /path_to_nginx_src
./configure --prefix=/usr/local/nginx --with-pcre=/path_to_pcre_src
make
make install
```


## 配置php-fpm
```
cp /usr/local/php-5.6/etc/php-fpm.conf.default /usr/local/php-5.6/et/php-fpm.conf

```
### 例子1
```
[global]
pid = /var/run/php-fpm.pid
error_log = /data/php/log/php-fpm.log
log_level = warning
emergency_restart_interval = 24h
process_control_timeout = 5s
daemonize = yes
 
rlimit_files = 10240
 
[www]
user = nobody
group = nobody
listen = 127.0.0.1:9000
listen.owner = nobody
listen.group = nobody
listen.mode = 0660
 
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 256
pm.start_servers = 64
pm.min_spare_servers = 32
pm.max_spare_servers = 128
 
pm.max_requests = 51200
pm.status_path = /status
 
 
slowlog = /data/php/log/$pool.log.slow
 
request_slowlog_timeout = 10
 
 
rlimit_files = 10240
```

## 启动php-fpm
```
/usr/local/php-5-6/sbin/php-fpm
```


## 配置nginx
### 简单配置
```
server {
	listen       80;
	server_name  localhost; 
	location / {
		root   /u01/www/;
		index  index.html index.php;
	}

        location ~ .php
        {
	        root /u01/www/;
		fastcgi_pass   127.0.0.1:9000;
        	fastcgi_index  index.php;
        	include        fastcgi.conf;
	}
}
```

### ThinkPhp 框架配置,兼容4种url模式
```
server {
    listen       81;
    server_name  localhost;
    root /u01/www/opsfnd/;
    index  index.html index.htm index.php;
    error_page  404              /404.html;
    location = /404.html {
        return 404 'Sorry, File not Found!';
    }
    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html; # windows用户替换这个目录
    }
    location / {
        try_files $uri @rewrite;
    }
    location @rewrite {
        set $static 0;
        if  ($uri ~ \.(css|js|jpg|jpeg|png|gif|ico|woff|eot|svg|css\.map|min\.map)$) {
            set $static 1;
        }
        if ($static = 0) {
            rewrite ^/(.*)$ /index.php?s=/$1;
        }
    }
    location ~ /Uploads/.*\.php$ {
        deny all;
    }
    location ~ \.php/ {
       if ($request_uri ~ ^(.+\.php)(/.+?)($|\?)) { }
       fastcgi_pass 127.0.0.1:9000;
       include fastcgi_params;
       fastcgi_param SCRIPT_NAME     $1;
       fastcgi_param PATH_INFO       $2;
       fastcgi_param SCRIPT_FILENAME $document_root$1;
    }
    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    location ~ /\.ht {
        deny  all;
    }
 }

```



## Reference
[http://www.thinkphp.cn/topic/34380.html](http://www.thinkphp.cn/topic/34380.html)
