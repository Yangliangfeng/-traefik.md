###    traefik入门及使用

* 介绍
```
1. 一款很强大的http反向代理和负载均衡软件

2. 支持和K8S、docker (swarm)、consul、etcd、zookeeper等交互

3. 关键在于：traefik可以监听你的服务组件(看第二点),  当服务被添加、移除、杀死或更新都会被检测到 
```
* 基于docker配置traefik
```
1. 下载镜像
docker pull traefik:1.7-alpine

2. docker启动
docker run -d -p 8080:8080 -p 80:80  --name tk  \
-v /home/shenyi/tk/traefik.toml:/etc/traefik/traefik.toml  \
traefik:1.7-alpine

# 注意：/etc/traefik/traeefik.toml 官方指定的配置文件的路径
```
