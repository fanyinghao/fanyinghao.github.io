---
title: 使用Docker部署Gitlab Runner遇到的坑
date: 2018-10-19 17:18:46
tags: [docker, gitlab, gitlab runner]
---
使用Gitlab Runner来做CI/CD，从部署gitlab到注册runner都是很顺利，因为官方提供了便捷的安装脚本。当要做CD（持续部署）时，一时没想到怎么做。

我的部署是在宿主机上使用Docker分别跑起了Gitlab Runner和Nginx两个容器，用了hexo工具生成

# Docker in Docker
gitlab runner跑在了docker容器中，而runner执行每个pipline的job也是启动了一个容器实例。如果CI脚本想build一个docker镜像，或者是启动一个容器服务，这里也要运行docker程序。docker in docker in docker?

官方文档给出了三种解决方案：
https://docs.gitlab.com/ee/ci/docker/using_docker_build.html

1. Use shell executor - 意思就是那就别docker跑runner，在宿主机直接跑runner吧。就变成了 docker in docker

2. Use docker-in-docker executor - 用一个特殊的docker镜像`docker:dind`，并开通特殊权限。官方推荐此方法。

3. Use Docker socket binding - 通过`--docker-volumes /var/run/docker.sock:/var/run/docker.sock`来把宿主机的docker服务直接分享给容器实例中，这种做法简单，但又安全隐患。

具体步骤我就不复制了，进官方文档直接看吧。
