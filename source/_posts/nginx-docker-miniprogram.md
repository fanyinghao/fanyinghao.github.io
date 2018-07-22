---
title: 利用 Nginx 做反向代理解决微信小程序业务域名限制问题
date: 2018-07-12 20:29:10
tags: [nginx, docker, 小程序]
category: 项目总结
---

最近做了一个世界杯比赛日程的小程序，只有查看日程的功能，所以很快就发布上线了。后边想加入用户预测功能，但是时间紧、前后端实现又比较复杂，突然灵机一动想到用webview嵌入FIFA官网的比赛预测页面。

微信小程序支持通过webview来内嵌网页，但是要求业务域名预先审核配置，就是说只能是你自己拥有的并且已经备案的域名。明显，我并不拥有FIFA官网，因而无法配置为业务域名。不过我可以利用[Nginx](https://nginx.org/)做反向代理（非透明代理，与之相反的是透明代理），利用自己的域名（https://api.wecode.net.cn ），把网页请求转发到FIFA官网（https://www.fifaofficial.cn ），这样我就不用开发就直接展示FIFA官方内容。

## 反向代理网页内容

一个网页的展示，首先得有HTML内容。把 https://api.wecode.net.cn 页面反向代理到 https://www.fifaofficial.cn ，这时请求就会得到目标网页的HTML内容返回。

``` properties
location ^~/ {
    proxy_set_header Accept-Encoding "";
    proxy_set_header Referer "https://www.fifaofficial.cn/";
    proxy_pass http://www.fifaofficial.cn/;

    add_header Access-Control-Allow-Origin *;
}
```

这个不仅能处理index页面的请求，因为`^~/`路由的是所有host为api.wecode.net.cn的所有请求。所以，给请求的返回加入`Access-Control-Allow-Origin`头，可以避免一些请求数据的跨域问题。

## 反向代理静态资源
在网页里，通常都会包含很多静态资源的引用，如css、js、font等，同时都是使用cdn加速，所以会是使用不同的域名。

``` html
<link href="//cdn.fifaofficial.cn/assets/css/76151aa27c3d7972aa5c.styles.css" rel="stylesheet">
```

第一，把html中的静态资源引用域名替换为自己的域名下，所以刚才的配置中要加入`sub_filter`([文档](https://nginx.org/en/docs/http/ngx_http_sub_module.html))替换声明

``` properties
location ^~/ {
    proxy_set_header Accept-Encoding "";
    proxy_set_header Referer "https://www.fifaofficial.cn/";
    proxy_pass http://www.fifaofficial.cn/;

    add_header Access-Control-Allow-Origin *;

    sub_filter 'cdn.fifaofficial.cn' 'api.wecode.net.cn';
    sub_filter_types text/css text/xml text/html text/javascript application/json application/javascript;
    sub_filter_once off;
}
```

第二，反向代理静态资源文件。如

``` 
cdn.fifaofficial.cn/assets/.....css    ===> api.wecode.net.cn/styles/.....css 
```

Nginx配置为

``` properties
location ~* \.(?:css|js|ttf|woff|svg|ico|png|jpg)$ {
    proxy_set_header Accept-Encoding "";
    proxy_set_header Referer "https://www.fifaofficial.cn/";
    proxy_pass http://cdn.fifaofficial.cn/$request_uri;

    add_header Access-Control-Allow-Origin *;

    sub_filter 'cdn.fifaofficial.cn' 'api.wecode.net.cn';
    sub_filter_types text/css text/xml text/html text/javascript application/javascript application/json;
    sub_filter_once off;
}
```

由于css、js等文件内也会引用别的资源，所以也是需要加入`sub_filter`替换声明的。

## 配置SSL证书
微信小程序要求服务器使用SSL协议，所以也需要配置。

``` properties
server {
    listen 443 ssl http2;
    server_name  api.wecode.net.cn;

    #ssl                      on;
    ssl_certificate          /etc/nginx/certs/api.wecode.net.cn_bundle.crt;
    ssl_certificate_key      /etc/nginx/certs/api.wecode.net.cn.key;

    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    ...

}
```

## 使用Docker部署Nginx服务
最近我也在学习容器服务，就尝试把Nginx服务部署到一个[Docker](https://www.docker.org)容器中。安装好Docker后，执行

``` bash
  $ docker container run \ 
    --rm \  
    --name mynginx \  
    --volume "$PWD/conf":/etc/nginx \
    --volume "$PWD/logs":/var/log/nginx/ \
    -p 127.0.0.1:443:443 \
    -d \
    nginx
```

上面的Nginx配置文件`default.conf`就是保存在`/conf`目录下，所以把文件目录`/conf`映射到`/etc/nginx`。`/logs`目录映射即是存放日志文件，方便查看Nginx服务的日志。另外，`-p`就是映射端口。

这时就可以访问 https://api.wecode.net.cn 就得到我想要的内容了，轻松把FIFA官方内容嵌入到我的小程序里。

## 源代码

完整代码放到了 https://github.com/fanyinghao/nginx-docker-miniprogram