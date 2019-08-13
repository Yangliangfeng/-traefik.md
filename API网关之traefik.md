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
-v /home/yang/tk:/etc/traefik  \
traefik:1.7-alpine

# 注意：/etc/traefik 官方指定的配置文件的路径
```
* traefik配置文件
```
1. routes.toml
    [api]

    [file]
      filename="/etc/traefik/rules.toml"
      watch=true
2. traefik.toml
      [backends]
        [backends.pro1]
          [backends.pro1.servers.userservice1]
            url="http://192.168.1.10:8001"
            weight=1
          [backends.pro1.servers.userservice2]
            url="http://192.168.1.10:8002"
            weight=2
          [backends.pro2.servers.userservices1]
            url="http://192.168.1.10:8003"

      [frontends]
        [frontends.pro1]
          backend = "pro1"
           [frontends.pro1.routes.apirule1]
             rule="Host:services.yang.me;Path:/v1/users"
           [frontends.pro1.routes.apirule2]
             rule="Method:GET"
        [frontends.pro1v2]
          backend="pro2"
           [frontends.pro1v2.routes.apirule1]
             rule="Host:services.yang.me;Path:/v2/users"
           [frontends.pro1v2.routes.apirule2]
             rule="Method:GET,POST"
```
