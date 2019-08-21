### 服务注册与发现之Consul

* 安装与启动
```
1. 下载
    docker pull consul

2. 启动
  docker run -d --name=cs -p 8500:8500  \
  consul agent -server -bootstrap  -ui -client 0.0.0.0
  
  1）-server  代表以服务端的方式启动
  2） -boostrap 指定自己为leader，而不需要选举
  3） -ui 启动一个内置管理web界面
  4） -client 指定客户端可以访问的IP。设置为0.0.0.0 则任意访问，否则默认本机可以访问。 

```
