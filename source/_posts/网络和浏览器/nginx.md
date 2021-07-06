---
title: nginx
## date: 2020-11-30 14:46:37
tags: [nginx,网络]
---

![nginx.png](https://i.loli.net/2020/12/01/HMLZ7bmBI23PkxD.png)

## 1 负载均衡

当用户访问web的时候，首先访问到的是负载均衡器，再通过负载均衡器将请求转发给后台服务器。
负载均衡的几种常用方式：

- 轮询（默认）

```javacript
// nginx.config
upstream backserver {
  server 192.168.0.1;
  server 192.168.0.2;
}
```

- 权重（weight）—指定不同ip的权重，权重越高，访问越大，适用于不同性能的服务器

```javacript
// nginx.config
upstream backserver {  
  server 192.168.0.1 weight=2;
  server 192.168.0.2 weight=8;
}

```

- 相应时间来分配—公平竞争，谁快谁处理， 不过需要依赖第三方插件nginx-upstream-fair

```javacript
// nginx.config
upstream backserver {
  server 192.168.0.1;
  server 192.168.0.2;
  fair;
}
server {
  listen 80;
  server_name localhost;
  location {
    proxy_pass  http://backserver;  
  }
}

```

nginx自带健康检查模块`（ngx_upstream_module）`，本质上是服务器心跳的检查，通过定时轮询向集群里的服务器健康检查请求，用来检查服务器是否处于异常状态。
健康检查涉及到两个配置属性：

- fail_timeout：设定服务器认为不可用的时间段以及统计失败尝试次数的时间段，默认为10S
- max_fails：设定Nginx与服务器通信尝试失败的次数，默认为1次

```javacript
upstream backserver{
  server 192.168.0.1 max_fails=1 fail_timeout=40s;
  server 192.168.0.2  max_fails=1 fail_timeout=40s;
}
```

## 2 反向代理

nginx代理服务器承担着中间人的角色，起到分配和沟通的作用。

- 反向代理的作用：
  - 防火墙作用：让用户无法通过请求直接访问真正的服务器，通过nginx过滤掉没有权限或者非法请求，来保障内部服务器的安全。
  - 负载均衡
- 如何使用反向代理：通过location功能匹配指定的URI，然后把接收的符合匹配URI的请求通过proxy_pass转移给之前定义好的upstream节点池。

```javacript
// nginx.config
server{
    listen 80;
    server_name localhost;
    location{
        proxy_pass http://127.0.0.1:8000;（upstream）  
    }
}

```

## 3 Https设置

Nginx常用来配置Https认证，主要有两个步骤：签署第三方可信任的SSL证书 和 配置Https

- 签署第三方可信任的SSL：
配置Https要用到私钥example.key文件和example.crt证书文件，而申请证书文件的时候要用到example.csr文件。

- Nginx配置Https：
要开启Https服务，在配置文件信息块（server）必须使用监听命令listen的ssl参数和定义服务器证书文件和私钥文件：

```javacript
// nginx.config
server{
  #ssl参数
  listen   443 ssl;
  //监听443端口，因为443端口是https的默认端口。80为http的默认端口  server_name  example.com;
  #证书文件
  ssl_certificate    example.com.crt;
  #私钥文件
  ssl_certificate_key    example.com.key;
}

```

- ssl_certificate：证书的绝对路径
- ssl_certificate_key：密钥的绝对路径;

## 4 其他常用设置

### （1）IP白名单

通过配置nginx白名单，规定哪些IP可以访问你的服务器，防爬虫必备

### （2）适配PC与移动环境

获取请求客户端的userAgent，从而知道当前用户终端是PC端还是移动端，并重定向到PC站和H5站

### （3）配置gzip

开启nginx gzip压缩，静态资源的大小会大大减少，从而节省带宽，提高传输效率。

### （4）配置跨域请求

当出现403跨域错误或其他跨域错误时，给nginx服务器配置相应的header参数

## 5 如何使用nginx

本地nginx的基本使用：

- 启动——`sudo nginx`
- 修改`nginx.conf`配置——`vim /usr/local/etc/nginx/nginx.conf`
- 检查语法是否正常——`sudo nginx –t`
- 重启`nginx`——`sudo nginx –s reload`
- 创建软连接——便于管理多个`nginx`

当我们需要管理多个网站的`nginx`，`nginx`文件放在一起是最好的管理方式。
我们把配置文件丢在`/nginx/conf.d/`文件夹下，假如我们程序文件夹下有一个`nginx`配置文件：`/home/app.nginx.conf`，只需要给这个文件创建一个软连接到`/nginx/conf.d/`下即可：
`ln -s /home/app/app.example.com.nginx.conf /etc/nginx/conf.d/app.nginx.conf`
这样操作之后，我们修改配置文件后，`/nginx/conf.d/`下与之对应的配置文件也会被修改，修改后重启`nginx`即可使用新的配置属性。

## 6 停止Nginx服务的四种方法

- 从容停止服务
这种方法较stop相比就比较温和一些了，需要进程完成当前工作后再停止。

```
nginx -s quit
```

- 立即停止服务
这种方法比较强硬，无论进程是否在工作，都直接停止进程。

```
nginx -s stop
```

- systemctl 停止
systemctl属于Linux命令

```
systemctl stop nginx.service
```

- killall 方法杀死进程
直接杀死进程，在上面无效的情况下使用，态度强硬，简单粗暴！

```
killall nginx
```
