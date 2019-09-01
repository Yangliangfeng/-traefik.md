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

3. Go安装consul扩展
    go get github.com/hashicorp/consul
    
4. consul在github上的地址
    https://github.com/hashicorp/consul/tree/master/api
    
5. consul文档
    https://godoc.org/github.com/hashicorp/consul/api
```
* uuid工具的使用
```
1. 安装：
  go get github.com/google/uuid
```
* 服务熔断
```
1. 第三方库
   https://github.com/afex/hystrix-go
   本质：隔离远程服务请求，防止级联故障
   
2. 安装
   go get github.com/afex/hystrix-go
   
3. 熔断器的配置
    hystrix.CommandConfig{
		Timeout: 2000, //超时时间，单位：毫秒
		MaxConcurrentRequests: 3, //最大并发数
		RequestVolumeThreshold:3, //熔断器请求阀值，意思是有3个请求才进行错误百分比计算。默认是20
		ErrorPercentThreshold: 20, //错误百分比。默认50（50%）
		SleepWindow:int(time.Second * 100),//熔断器打开后，过多久后去尝试服务是否可用。默认5秒，生产环境中一般设置5-20秒之间。
	}
    熔断器的三种状态：
    1）关闭： 默认状态。如果请求次数异常超过设定比例，则打开熔断器
    
    2）打开：当熔断器打开的时候，直接执行降级方法
    
    3）半开：定期的尝试发起请求来确认系统是否恢复。如果恢复了，熔断器将转为关闭状态或者保持打开
```
