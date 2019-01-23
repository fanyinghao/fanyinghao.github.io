---
title: Docker容器怎么访问宿主机的服务
date: 2019-01-23 16:28:28
tags:
---
If I have already launched mysql database service in the host server, and now I want allow the service in a docker container instance to access the host's mysql service. What should I do?

如果宿主机上已经部署了mysql服务, 怎样才能让docker 得容器实例中的程序访问到mysql服务呢？

I found answer from the stackoverflow:

通过搜索，我找到了以下相关线索：

> For example, on my system:
> ```bash
> $ ip addr show docker0
> 7: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
> link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
> inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
> valid_lft forever preferred_lft forever
> inet6 fe80::f4d2:49ff:fedd:28a0/64 scope link 
> valid_lft forever preferred_lft forever
> ```
> And inside a container:
> ```bash
> $ ip route show
> default via 172.17.0.1 dev eth0 
> 172.17.0.0/16 dev eth0  src 172.17.0.4 
> ```
> It's fairly easy to extract this IP address using a simple shell script:
> ```bash
> #!/bin/sh
> hostip=$(ip route show | awk '/default/ {print $3}')
> echo $hostip
> ```
> You may need to modify the iptables rules on your host to permit connections from Docker containers. Something like this will do the trick:
> ```bash
> $ iptables -A INPUT -i docker0 -j ACCEPT
> ```
> This would permit access to any ports on the host from Docker containers. Note that:
> - iptables rules are ordered, and this rule may or may not do the right thing depending on what other rules come before it.
> - you will only be able to access host services that are either (a) listening on INADDR_ANY (aka 0.0.0.0) or that are explicitly listening on the docker0 interface.

REF：https://stackoverflow.com/questions/31324981/how-to-access-host-port-from-docker-container/31328031