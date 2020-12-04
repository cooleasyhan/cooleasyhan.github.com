date: 2015-12-02
title: 利用nginx 转发 EBS
tags: ebs, nginx
sulg: nginx_ebs


## 需求

EBS 服务器搭建在内网上， 需要开放给外网使用。   
实现： EBS 设置单一网络入口点，  外网服务器上配置nginx 转发至内网服务器。


## 配置EBS

### 修改host文件， 在EBS app 服务器上添加相关host 

```
110.10.10.10 prod.xxx.xxx.xxx prod
```

### 修改 $CONTEXT_FILE , 配置EBS单一网络入口

http://app01.domain.name:8000  ===>   http://prod.domain.name:80  
修改前后比对
```
[applprod@app01 admin]$ diff PROD_app01.xml PROD_app01.xml.bak 
123c123
<          <chronosURL oa_var="s_chronosURL" customized="yes">http://prod.domain.name:80/oracle_smp_chronos/oracle_smp_chronos_sdk.gif</chronosURL>
---
>          <chronosURL oa_var="s_chronosURL">http://app01.domain.name:8000/oracle_smp_chronos/oracle_smp_chronos_sdk.gif</chronosURL>
125c125
<          <EndUserMonitoringURL oa_var="s_endUserMonitoringURL" customized="yes">http://prod.domain.name:80/oracle_smp_chronos/oracle_smp_chronos_sdk.gif</EndUserMonitoringURL>
---
>          <EndUserMonitoringURL oa_var="s_endUserMonitoringURL">http://app01.domain.name:8000/oracle_smp_chronos/oracle_smp_chronos_sdk.gif</EndUserMonitoringURL>
184c184
<          <externURL oa_var="s_external_url" customized="yes">http://prod.domain.name:80</externURL>
---
>          <externURL oa_var="s_external_url">http://app01.domain.name:8000</externURL>
186,187c186,187
<          <webentryhost oa_var="s_webentryhost" customized="yes">prod</webentryhost>
<          <webentrydomain oa_var="s_webentrydomain" customized="yes">domain.name</webentrydomain>
---
>          <webentryhost oa_var="s_webentryhost">app01</webentryhost>
>          <webentrydomain oa_var="s_webentrydomain">domain.name</webentrydomain>
228c228
<          <login_page oa_var="s_login_page" customized="yes">http://prod.domain.name:80/OA_HTML/AppsLogin</login_page>
---
>          <login_page oa_var="s_login_page">http://app01.domain.name:8000/OA_HTML/AppsLogin</login_page>
764c764
<       <activewebport oa_var="s_active_webport" oa_type="DUP_PORT" base="8000" step="1" range="-1" label="Active Web Port">80</activewebport>
---
>       <activewebport oa_var="s_active_webport" oa_type="DUP_PORT" base="8000" step="1" range="-1" label="Active Web Port">8000</activewebport>
994d993
<    <oa_customized/>
```

### 执行 autoconfig

### 重启app 服务



## 配置nginx 服务
### 修改Host
同app服务器
### 安装nginx
[Nginx, Php-fpm 安装](http://www.yilaguan.cc/posts/2015/11/26/nginx-php-fpm.html)
### 配置nginx

nginx.conf
```
#user  nobody;
worker_processes  6;

error_log  logs/error.log  info;

pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile                    on;
    tcp_nopush                  on;
    tcp_nodelay                 on;
    client_body_buffer_size     1024k;

    ##cache
    proxy_connect_timeout       600;
    proxy_read_timeout          600;
    proxy_send_timeout          600;
    proxy_buffer_size           16k;
    proxy_buffers               4 64k;
    proxy_busy_buffers_size     128k;
    proxy_temp_file_write_size  1024k;
    proxy_temp_path /usr/local/nginx/new_app01_temp;
    proxy_cache_path /usr/local/nginx/cache levels=1:2 keys_zone=new_app01_cache:200m inactive=1d max_size=30g;
    ##end

    #keepalive_timeout  0;
    keepalive_timeout  300s;

    ## gzip setting begin.
    gzip                on;
    gzip_min_length     1k;
    gzip_buffers        4 16k;
    gzip_http_version   1.1;
    gzip_comp_level     9;
    gzip_vary           off;
    gzip_types          text/plain text/javascript text/css text/xml application/xml;
    ## gzip setting end.

    output_buffers      4 32k;
    postpone_output     1460;
    client_header_buffer_size       128k;
    large_client_header_buffers     4 256k;

    log_format  log_info  '$remote_addr - $remote_user [$time_local] -DIRECT $upstream_addr  $request ' '"$status" $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwarded_for"' '$request_time $upstream_cache_status';
    access_log  /usr/local/nginx/logs/app01.log  log_info;

    include include/*/vhost.conf;
}
```

vhost.conf
```
server {
    listen       80;
    server_name  prod.domain.name;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;
     location ~ .*\.(gif|jpg|png|htm|html|css|js|flv|ico|swf)(.*) {
        ## proxy_next_upstream http_500 http_502 http_503 http_504 error timeout invalid_header;
         proxy_next_upstream off;
         proxy_pass     http://10.10.10.10:8000;
         #proxy_redirect     off;
         proxy_set_header   Host $host:80;
         proxy_set_header   X-Real-IP  $remote_addr;
         proxy_set_header   X-Forwarded-For $remote_addr;
         proxy_cache    new_app01_cache;
         proxy_cache_valid  200 302 1h;
         proxy_cache_valid  301 1d;
         proxy_cache_valid  any 1m;
         proxy_cache_key    $host$uri$is_args$args;
         expires        -1d;
    }

    location ~ ^/status/ {
       stub_status on;
       access_log off;
    }
     location ~ /purge(/.*) {
        proxy_cache_purge new_app01_cache $host$1$is_args$args;
        allow all;
        }
    location / {
       #proxy_redirect off;
       proxy_pass    http://10.10.10.10:8000;

       proxy_next_upstream off;
       proxy_set_header   Host         $host:80;
       proxy_set_header   X-Real-IP    $remote_addr;
       proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
       proxy_set_header REMOTE-HOST $remote_addr;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```


## Reference
[https://www.nginx.com/wp-content/uploads/2015/11/oracle-ebusiness-suite-deployment-guide-v1.pdf](https://www.nginx.com/wp-content/uploads/2015/11/oracle-ebusiness-suite-deployment-guide-v1.pdf)  
[Doc ID 387859.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=351925156011594&id=387859.1&_afrWindowMode=0&_adf.ctrl-state=j2tyjckww_63)
