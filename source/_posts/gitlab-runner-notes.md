---
title: 使用Docker部署Gitlab Runner遇到的坑
date: 2018-10-19 17:18:46
tags: [docker, gitlab, gitlab runner]
---
使用Gitlab Runner来做CI/CD，从部署gitlab到注册runner都是很顺利，因为官方提供了便捷的安装脚本。当要做CD（持续部署）时，一时没想到怎么做。

我的部署是在宿主机上使用Docker分别跑起了Gitlab Runner和Nginx两个容器，用了hexo工具生成


