---
layout: post
title: SpringBoot+Nginx+Https下redirect问题
date: 2017-09-18
categories: blog
tags: [IT,SpringBoot]
description: 项目采用springboot框架，内嵌tomcat容器。前端采用nginx反向代理，当部署了https以后出现的重定向（redirect）的问题。用nginx反向代理tomcat，然后把nginx配置为https访问，并且nginx与tomcat之间配置为普通的http协议，当后台代码定义时redirect，实际是重定向到了http下的地址，导致浏览器上无法访问非https的地址。
---
# 问题描述
项目采用springboot框架，内嵌tomcat容器。前端采用nginx反向代理，当部署了https以后出现的重定向（redirect）的问题。用nginx反向代理tomcat，然后把nginx配置为https访问，并且nginx与tomcat之间配置为普通的http协议，当后台代码定义时redirect，实际是重定向到了http下的地址，导致浏览器上无法访问非https的地址。
# 解决方案
## 配置nginx
由于对tomcat而言收到的是普通的http请求，因此当tomcat里的应用发生转向请求时将转向为http而非https，为此我们需要告诉tomcat已被https代理，方法是增加X-Forwared-Proto和X-Forwarded-Port两个HTTP头信息。
```
server {
    listen 80;
    listen 443 ssl;
    server_name localhost;
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
    }
}
```
## 配置tomcat
springboot配置参见[Spring Boot Reference Guide-Use behind a front-end proxy server ](https://docs.spring.io/spring-boot/docs/2.0.0.M3/reference/htmlsingle/#howto-use-tomcat-behind-a-proxy-server)

基于spring-boot开发时只需在application.properties中进行配置。
该配置将指示tomcat从HTTP头信息中去获取协议信息（而非从HttpServletRequest中获取。
```
server.tomcat.remote_ip_header=x-forwarded-for
server.tomcat.protocol_header=x-forwarded-proto
server.tomcat.port-header=X-Forwarded-Port
server.use-forward-headers=true
```
以下是经常被漏掉的配置，本人一开始就忘记配置导致不成功。因为我的nginx的机器IP不在默认允许IP列表里
>Tomcat is also configured with a default regular expression that matches internal proxies that are to be trusted. By default, IP addresses in 10/8, 192.168/16, 169.254/16 and 127/8 are trusted. You can customize the valve’s configuration by adding an entry to application.properties, e.g.
```
server.tomcat.internal-proxies=192\\.168\\.\\d{1,3}\\.\\d{1,3}
```
此外，虽然我们的tomcat被nginx反向代理了，但仍可访问到其8080端口。为此可在application.properties中增加一行：
```
server.address=127.0.0.1
```
这样一来其8080端口就只能被本机访问了，其它机器访问不到。
