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
* 匹配器详解
```
1. Path
   完全匹配器。当匹配到/v2/users 进行转发。 注意：它会把/v2/users这个路径转发给后端
   
2. PathStrip
   同Path .区别在于它会把 /v2/users  去掉（strip掉）。把/转发给后端
   
3. PathPrefix 
   区别在于  它会把/v2/users 转发给后端的同时，把后面的路径也转发过去
   譬如PathPrefix:/v2/users
   如果我请求地址是  /v2/users/abc    则会转发/v2/users/abc
   
4. PathPrefixStrip
   如果我请求地址是  /v2/users/abc    则会转发/abc  。因为它把 /v2/users 这个前缀去掉了
```
* Entrypoint
```
entrypoint是traefik的网络入口，作用如下：

1. 监听端口

2. 处理SSL、redirect、控制白名单、basic auth 认证、设置代理

3. 在经过entrypoint后，流量会被转发到一个匹配的frontend上

4. frontend把请求传送到backend中定义的server中

5. server会把请求转发到真正的服务上去
```
