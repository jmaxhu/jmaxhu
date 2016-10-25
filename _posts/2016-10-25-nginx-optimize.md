---
layout: post
header-img: img/nginx-logo.png
title: Nginx+Php配置优化 
tags: nginx,php
---

# Nginx 配置优化

## Nginx Tip 1. 决定 Nginx worker_processes 和 worker_connections

**max_clients = worker_processes * worker_connections**

Nginx 的默认设置为：

```
  worker_processes 1;
  worker_connections 1024;
```

合理的设置应该是：

```
  worker_processes [处理器核心数量]
```

检查处理器核心数量的方法如下：

```
  cat /proc/cpuinfo | grep processor
  processor : 0
  processor : 1
```

这表示2核心， 所以  worker_processes 应该设置如下：

```
  worker_processes 2;
```

worker_connections 设置为 1024 一般就可以了。

## Nginx Tip 2. 隐藏 Nginx Server Tokens 和版本号

隐藏 server Tokens 和版本号主要基于案例考虑。隐藏很简单，在 **http/server/location** 中设置如下：

```
  server_tokens off;
```

## Nginx Tip 3. Nginx 请求和最大上传内容(client_max_body_size)

如果应用有上传功能，则可能需要加大允许上传的最大大小。 可以设置 **http/server/location** 下的 **client_max_body_size** 配置项。
默认是 1M，如果想设置成 20M，并且增大缓冲区大小，配置如下：

```
  client_max_body_size 20m;
  client_body_buffer_size 128k;
```

如果访问时得到如下错误，则说明 **client_max_body_size** 太小了：
"Request Entity Too Large" (413)

## Nginx Tip 4. Nginx 静态资源缓存控制

浏览器缓存对于节省资源和带宽很重要。不变化的资源可以配置如下：

```
  location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$
    access_log     off;
    log_not_found  off;
    expires        360d;
```

## Nginx Top 5. Nginx Pass PHP 请求到 PHP-FPM

可以使用默认的 tcp/ip 或 Unix socket 连接。常用配置如下：

```
location ~* \.php$ {
    fastcgi_index     index.php;
    fastcgi_pass      127.0.0.1:9000;
    #fastcgi_pass     unix:/var/run/php-fpm/php-fpm.sock;
    include           fastcgi_params;
    fastcgi_param     SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    fastcgi_param     SCRIPT_NAME  $fastcgi_script_name;
} 
```

PHP-FPM 也可以运行在另一台机器上。

## Nginx Tip 6. 防止访问隐藏文件

一般要限制点(.)开头的文件访问，比如 .htaccsss, .git 等。配置如下：

```
location ~ /\. {
    access_log  off;
    log_not_found off;
    deny all;
}
```

# PHP-FPM 配置优化

## PHP-FPM Tip 1. 全局设置

设置 emergency_restart_threshold, emergency_restart_interval 和 process_control_timeout.
默认这些值都是 off。但可以考虑设置如下：

```
emergency_restart_threshold 10
emergency_restart_interval 1m
process_control_timeout 10s
```

这样设置的意思是，如果10个 PHP-FPM 子进程在1分钟里以 SIGSEGV 或 SIGUBS 退出，则自动重启。
这个配置还设置了10秒时间允许子进程也主进程进行信号交互。

## PHP-FPM Tip 2. Pool 配置

PHP-FPM 允许为不同的站点配置不同的 pools。比如有如下两个配置文件。

```
/etc/php-fpm.d/site.conf
/etc/php-fpm.d/blog.conf
```

两个文件的的配置示例：

**/etc/php-fpm.d/site.conf**

```
[site]
listen = 127.0.0.1:9000
user = site
group = site
request_slowlog_timeout = 5s
slowlog = /var/log/php-fpm/slowlog-site.log
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 5
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 4
pm.max_requests = 200
listen.backlog = -1
pm.status_path = /status
request_terminate_timeout = 120s
rlimit_files = 131072
rlimit_core = unlimited
catch_workers_output = yes
env[HOSTNAME] = $HOSTNAME
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

**/etc/php-fpm.d/blog.conf**

```
[blog]
listen = 127.0.0.1:9001
user = blog
group = blog
request_slowlog_timeout = 5s
slowlog = /var/log/php-fpm/slowlog-blog.log
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 4
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
pm.max_requests = 200
listen.backlog = -1
pm.status_path = /status
request_terminate_timeout = 120s
rlimit_files = 131072
rlimit_core = unlimited
catch_workers_output = yes
env[HOSTNAME] = $HOSTNAME
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

## PHP-FPM Tip 3. PHP-FPM Pool Process Manager (pm) 配置

使用 PHP-FPM process manager 的最好方式是使用动态进程管理。一般可以用 top 等命令得到每个
PHP-FPM进程的内存占用量，通常的区间是(20-40M)，假设每个PHP-FPM进程占用30M内存，假设可以有4G内存可以用来分配给PHP-FPM，
则最大的PHP-FPM进程数为 4*1024/30=136.3

所以 pm.max_children  的值可以设置为 136.示例配置如下：

```
pm.max_children = 9
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 4
pm.max_requests = 200
```

其中：

pm.max_children pm 设置为 static 时表示创建的子进程的数量，pm 设置为 dynamic 时表示最大可创建的子进程的数量。必须设置。
pm.start_servers 设置启动时创建的子进程数目。仅在 pm 设置为 dynamic 时使用。默认值：min_spare_servers + (max_spare_servers - min_spare_servers) / 2。
pm.min_spare_servers 设置空闲服务进程的最低数目。仅在 pm 设置为 dynamic 时使用。必须设置。
pm.max_spare_servers 设置空闲服务进程的最大数目。仅在 pm 设置为 dynamic 时使用。必须设置。
pm.max_requests 设置每个子进程重生之前服务的请求数。对于可能存在内存泄漏的第三方模块来说是非常有用的。如果设置为 '0' 则一直接受请求，等同于 PHP_FCGI_MAX_REQUESTS 环境变量。默认值：0。

完