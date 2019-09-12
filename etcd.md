### Etcd的基本使用

* 介绍
```
1. etcd是一个高可用的键值存储系统,场景主要是：

1）主要用于共享配置
2）服务注册与发现
3）分布式锁

2. etcd是由CoreOS开发并维护的,灵感来自于 ZooKeeper，它使用Go语言编写

3. 下载地址
  https://github.com/etcd-io/etcd/releases（下载比较慢）
  
  http://www.jtthink.com/download/detail?did=227（备用，下载速度比较快）
```
* docker启用服务
```
1. docker下载镜像
  docker pull golang:1.12-alpine

2. 创建存放配置文件和数据的文件夹
  mkdir -p etcd/conf etcd/data

3. 启动容器
  docker run -it --name etcd \
  -p 2379:2379 \
  -v /home/yang/etcd:/etcd \
  golangd:1.12-alpine sh （注意 -d 不能启动服务）
  
4. 拷贝etcd的服务端和客户端到容器（进入到etcd解压后的文件夹）
  docker cp etcd etcd:/usr/bin && docker cp etcdctl etcd:/usr/bin 

5. 创建etcd.yml配置文件
  name: $(hostname -s)
  data_dir: /etcd/data
  listen-client-urls: http://0.0.0.0:2379

6. 启动etcd
  1、docker exec -it etcd sh (进入容器)
  2、chmod +x /usr/bin/etcd  (增加可执行权限)
  3、etcd --version  查看版本
  4、 etcd --config-file  /etcd/conf/etcd.yml
```
* etcd的基本使用一
```
1. 介绍
  Etcdctl是客户端工具，由于历史原因，存在v2和v3版本。

2. 设置环境变量（版本3）
  export ETCDCTL_API=3

3. 查看版本
  etcdctl version

4. 增删改查
   1）查
   etcdctl get /user/101/name  
   etcdctl get /user/101/age
   etcdctl get /user/101 --prefix  //查询前缀为/user/101的所有键的值
  
   2）增
   etcdctl put /user/101/name shenyi
   etcdctl put /user/101/age  19
   
   3）删
   etcdctl del /user/101 --prefix （删除这个前缀的键）
   etcdctl del /user/101/age
   
```
* docker模拟集群的启动
```
1. 启动命令
  docker run -d  --name etcd1 --network etcdnet --ip 172.25.0.101 -p 23791:2379 -e ETCDCTL_API=3   
  -v /home/yang/etcd:/etcd etcd:1.1 etcd --config-file  /etcd/conf/etcd.yml
```
* Go的调用etcd的API
```
1. 安装
  go get go.etcd.io/etcd/clientv3

2. 文档
  https://godoc.org/go.etcd.io/etcd/clientv3
```
* 租约模块命令
```
1. 设置租约
    etcdctl lease grant 20  ------设置过期20秒的grant租约
    
2. 查看租约列表
   etcdctl lease list
   
3. 查看租约剩余时间
   etcdctl lease timetolive +ID  ------ID是租约的ID
   
4. 删除租约
   etcdctl lease revoke + ID  -------ID租约的ID
   
5. 保持租约始终存活
   etcdctl lease keep-alive + ID
   
6. 可以查看租约下的所有key
   etcdctl lease timetolive + ID --keys

7. 把key和租约关联
   etcdctl put /user yang --lease=ID
```
* Confd 配置管理工具
```
1. 地址
  https://github.com/kelseyhightower/confd

2. 通过docker的Dockerfile安装confd
   FROM golang:1.12-alpine  as confd
   ARG CONFD_VERSION=0.16.0
   ADD https://github.com/kelseyhightower/confd/archive/v${CONFD_VERSION}.tar.gz /tmp/
   RUN apk add --no-cache \
       bzip2 \
       make && \
   mkdir -p /go/src/github.com/kelseyhightower/confd && \
   cd /go/src/github.com/kelseyhightower/confd && \
   tar --strip-components=1 -zxf /tmp/v${CONFD_VERSION}.tar.gz && \
   go install github.com/kelseyhightower/confd && \
   rm -rf /tmp/v${CONFD_VERSION}.tar.gz
 ENTRYPOINT ["/go/bin/confd"]

3. 根据dockerfile构建镜像
   docker build -t confd:1.1 .
```
* Confd 的启动流程
```
1. 前期准备----新建文件夹
   mkdir -p /home/shenyi/confdfiles/{conf.d,templates,dest}
   ## conf.d 放配置文件，templates 放模板文件
   
   在conf.d里面创建配置叫做myconfig.toml
   内容如下：
   [template]
    src = "myconfig.conf.tmpl"
    dest = "/etc/confd/dest/myconfig.conf"
    keys = [
        "/myconfig/mysql/user",
        "/myconfig/mysql/pass",
    ]
    
    templates里面创建一个文件叫做myconfig.conf.tmpl
    内容如下：
      [this is myconfig]
      database_user= {{getv "/myconfig/mysql/user"}}
      database_pass = {{getv "/myconfig/mysql/pass"}}


3. 启动
  docker run -it --rm  --name confd -v /home/yang/confdfiles:/etc/confd confd:1.1 
  -onetime -backend etcdv3 -node http://192.168.2.252:23791
  ## onetime 只运行一次

4. 定时任务
  docker run -d --rm  --name confd -v /home/yang/confdfiles:/etc/confd confd:1.1
  -interval 5 -backend etcdv3 -node http://192.168.2.252:23791
```
* 交叉编译（在window上编译，在Linux运行）
```
1. 在goland编译器新建build.bat文件:
  set GOARCH=amd64
  set GOOS=linux
  go build xxx.go  ##xxx.go编译的文件

  
```

