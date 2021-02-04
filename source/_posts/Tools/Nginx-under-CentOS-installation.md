---
title: CentOS下Nginx支持SSL协议的编译安装
date: 2020-07-04 18:24:35
tags: 
    - Nginx
    - Liunx
    - Server
    - CentOS
categories:
    - Nginx
---

## Nginx及其衍生的其他相关优秀开源产品
* [NGINX](http://nginx.org/) 是开源、高性能、高可靠的 Web 和反向代理服务器
* [Tengine](https://tengine.taobao.org/) Tengine是由淘宝网发起基于Nginx的Web服务器项目
* [OpenResty](https://openresty.org/cn/) OpenResty® 是一个基于 Nginx 与 Lua 的高性能 Web 平台


## 下载Nginx稳定版包并解压
```bash
wget http://nginx.org/download/nginx-x.x.x.tar.gz
tar zxvf nginx-x.x.x.tar.gz
cd nginx-x.x.x
```
<!-- more -->

## 补全需要的库依赖
```bash
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```

## 执行编译安装
所有文件放在同一个位置,便于统一管理
```bash
./configure --prefix=/usr/local/nginx  \
--conf-path=/usr/local/nginx/etc/nginx.conf  \
--user=nginx --group=nginx  \
--error-log-path=/usr/local/nginx/nginxlog/error.log  \
--http-log-path=/usr/local/nginx/nginxlog/access.log  \
--pid-path=/usr/local/nginx/pids/nginx.pid  \
--lock-path=/usr/local/nginx/locks/nginx.lock  \
--with-http_ssl_module  \
--with-http_stub_status_module  \
--with-http_gzip_static_module  \
--http-client-body-temp-path=/usr/local/nginx/tmp/client  \
--http-proxy-temp-path=/usr/local/nginx/tmp/proxy  \
--http-fastcgi-temp-path=/usr/local/nginx/tmp/fastcgi  \
--http-uwsgi-temp-path=/usr/local/nginx/tmp/uwsgi  \
--http-scgi-temp-path=/usr/local/nginx/tmp/scgi
```
编译结果
```bash
Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/etc"
  nginx configuration file: "/usr/local/nginx/etc/nginx.conf"
  nginx pid file: "/usr/local/nginx/pids/nginx.pid"
  nginx error log file: "/usr/local/nginx/nginxlog/error.log"
  nginx http access log file: "/usr/local/nginx/nginxlog/access.log"
  nginx http client request body temporary files: "/usr/local/nginx/tmp/client"
  nginx http proxy temporary files: "/usr/local/nginx/tmp/proxy"
  nginx http fastcgi temporary files: "/usr/local/nginx/tmp/fastcgi"
  nginx http uwsgi temporary files: "/usr/local/nginx/tmp/uwsgi"
  nginx http scgi temporary files: "/usr/local/nginx/tmp/scgi
```
执行安装
```bash
make && make install
```
## 安装后处理
查看文件结构
```bash
cd /usr/local/nginx/
tree
.
├── etc
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── html
│   ├── 50x.html
│   └── index.html
├── nginxlog
├── pids
└── sbin
    └── nginx
```
完善Nginx目录结构
```bash
mkdir -pv /usr/local/nginx/tmp/{client,proxy,fastcgi,uwsgi,scgi}
```
## Nginx启动和监听
```bash
# 启动
/usr/local/nginx/sbin/nginx
# 检查是否启动
ss -tnlp | grep :80
```