---
title: "11-Dubbo启动器DubboBootstrap添加协议配置信息ProtocolConfig"
linkTitle: "11-Dubbo启动器DubboBootstrap添加协议配置信息ProtocolConfig"
date: 2022-08-11
author: 宋小生
description: >
    [Dubbo 3.0.8源码解析] ProtocolConfig协议配置是RPC调用过程中一些必要信息的基础。
---

# 11-Dubbo启动器DubboBootstrap添加协议配置信息ProtocolConfig
## 11.1 简介
先贴个代码用来参考:

```java
 DubboBootstrap bootstrap = DubboBootstrap.getInstance();
 bootstrap.application(new ApplicationConfig("dubbo-demo-api-provider"))
            .registry(new RegistryConfig("zookeeper://127.0.0.1:2181"))
            .protocol(new ProtocolConfig(CommonConstants.DUBBO, -1))
            .service(service)
            .start()
            .await();
 
```

上个博客我们说了 RegistryConfig对象的创建,启动器对象在启动之前是要初始化一些配置信息的,这里我们来看这一行代码协议配置信息:
```java
.protocol(new ProtocolConfig(CommonConstants.DUBBO, -1))
```


## 11.2  协议的配置相关

下面的配置来源于官网

服务提供者协议配置。对应的配置类： org.apache.dubbo.config.ProtocolConfig。同时，如果需要支持多协议，可以声明多个 <dubbo:protocol> 标签，并在 <dubbo:service> 中通过 protocol 属性指定使用的协议。

| 属性          | 对应URL参数   | 类型           | 是否必填 | 缺省值                                                       | 作用     | 描述                                                         | 兼容性         |
| ------------- | ------------- | -------------- | -------- | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ | -------------- |
| id            |               | string         | 可选     | dubbo                                                        | 配置关联 | 协议BeanId，可以在<dubbo:service protocol="">中引用此ID，如果ID不填，缺省和name属性值一样，重复则在name后加序号。 | 2.0.5以上版本  |
| name          | <protocol>    | string         | **必填** | dubbo                                                        | 性能调优 | 协议名称                                                     | 2.0.5以上版本  |
| port          | <port>        | int            | 可选     | dubbo协议缺省端口为20880，rmi协议缺省端口为1099，http和hessian协议缺省端口为80；如果**没有**配置port，则自动采用默认端口，如果配置为**-1**，则会分配一个没有被占用的端口。Dubbo 2.4.0+，分配的端口在协议缺省端口的基础上增长，确保端口段可控。 | 服务发现 | 服务端口                                                     | 2.0.5以上版本  |
| host          | <host>        | string         | 可选     | 自动查找本机IP                                               | 服务发现 | -服务主机名，多网卡选择或指定VIP及域名时使用，为空则自动查找本机IP，-建议不要配置，让Dubbo自动获取本机IP | 2.0.5以上版本  |
| threadpool    | threadpool    | string         | 可选     | fixed                                                        | 性能调优 | 线程池类型，可选：fixed/cached                               | 2.0.5以上版本  |
| threads       | threads       | int            | 可选     | 200                                                          | 性能调优 | 服务线程池大小(固定大小)                                     | 2.0.5以上版本  |
| iothreads     | threads       | int            | 可选     | cpu个数+1                                                    | 性能调优 | io线程池大小(固定大小)                                       | 2.0.5以上版本  |
| accepts       | accepts       | int            | 可选     | 0                                                            | 性能调优 | 服务提供方最大可接受连接数                                   | 2.0.5以上版本  |
| payload       | payload       | int            | 可选     | 8388608(=8M)                                                 | 性能调优 | 请求及响应数据包大小限制，单位：字节                         | 2.0.5以上版本  |
| codec         | codec         | string         | 可选     | dubbo                                                        | 性能调优 | 协议编码方式                                                 | 2.0.5以上版本  |
| serialization | serialization | string         | 可选     | dubbo协议缺省为hessian2，rmi协议缺省为java，http协议缺省为json | 性能调优 | 协议序列化方式，当协议支持多种序列化方式时使用，比如：dubbo协议的dubbo,hessian2,java,compactedjava，以及http协议的json等 | 2.0.5以上版本  |
| accesslog     | accesslog     | string/boolean | 可选     |                                                              | 服务治理 | 设为true，将向logger中输出访问日志，也可填写访问日志文件路径，直接把访问日志输出到指定文件 | 2.0.5以上版本  |
| path          | <path>        | string         | 可选     |                                                              | 服务发现 | 提供者上下文路径，为服务path的前缀                           | 2.0.5以上版本  |
| transporter   | transporter   | string         | 可选     | dubbo协议缺省为netty                                         | 性能调优 | 协议的服务端和客户端实现类型，比如：dubbo协议的mina,netty等，可以分拆为server和client配置 | 2.0.5以上版本  |
| server        | server        | string         | 可选     | dubbo协议缺省为netty，http协议缺省为servlet                  | 性能调优 | 协议的服务器端实现类型，比如：dubbo协议的mina,netty等，http协议的jetty,servlet等 | 2.0.5以上版本  |
| client        | client        | string         | 可选     | dubbo协议缺省为netty                                         | 性能调优 | 协议的客户端实现类型，比如：dubbo协议的mina,netty等          | 2.0.5以上版本  |
| dispatcher    | dispatcher    | string         | 可选     | dubbo协议缺省为all                                           | 性能调优 | 协议的消息派发方式，用于指定线程模型，比如：dubbo协议的all, direct, message, execution, connection等 | 2.1.0以上版本  |
| queues        | queues        | int            | 可选     | 0                                                            | 性能调优 | 线程池队列大小，当线程池满时，排队等待执行的队列大小，建议不要设置，当线程池满时应立即失败，重试其它服务提供机器，而不是排队，除非有特殊需求。 | 2.0.5以上版本  |
| charset       | charset       | string         | 可选     | UTF-8                                                        | 性能调优 | 序列化编码                                                   | 2.0.5以上版本  |
| buffer        | buffer        | int            | 可选     | 8192                                                         | 性能调优 | 网络读写缓冲区大小                                           | 2.0.5以上版本  |
| heartbeat     | heartbeat     | int            | 可选     | 0                                                            | 性能调优 | 心跳间隔，对于长连接，当物理层断开时，比如拔网线，TCP的FIN消息来不及发送，对方收不到断开事件，此时需要心跳来帮助检查连接是否已断开 | 2.0.10以上版本 |
| telnet        | telnet        | string         | 可选     |                                                              | 服务治理 | 所支持的telnet命令，多个命令用逗号分隔                       | 2.0.5以上版本  |
| register      | register      | boolean        | 可选     | true                                                         | 服务治理 | 该协议的服务是否注册到注册中心                               | 2.0.8以上版本  |
| contextpath   | contextpath   | String         | 可选     | 缺省为空串                                                   | 服务治理 |                                                              | 2.0.6以上版本  |


 同样官网提供的参数里面并未包含所有的属性 下面我就将其余的属性列举一下方便学习参考:
 
| 变量 |	类型 |	说明 |
|--|--|--|
|threadname|String|线程池名称|
|corethreads|Integer|线程池核心线程大小|
|alive|Integer|线程池keepAliveTime，默认单位时间单位。毫秒|
|exchanger|String|交换器配置信息如何交换|
|prompt|String|命令行提示符|
|status|String|状态检查|
|sslEnabled|Boolean|ssl是否启用|


## 11.3 协议配置对象创建与添加
前面例子中调用的代码
```java
 .protocol(new ProtocolConfig(CommonConstants.DUBBO, -1))
```
这里我们配置了协议类型为dubbo 端口为-1则会分配一个没有被占用的端口

继续看下DubboBootstrap的protocol方法

```java
public DubboBootstrap protocol(ProtocolConfig protocolConfig) {
		//配置信息转List
        return protocols(singletonList(protocolConfig));
    }
```

继续看protocols方法 ,这个代码与前面两个博客中看到的向配置管理器添加配置对象的逻辑是一样的
这里就不说了可以看前面的博客[《9-Dubbo启动器DubboBootstrap添加应用程序的配置信息ApplicationConfig》](https://blog.elastic.link/2022/07/10/dubbo/9-dubbo-qi-dong-qi-dubbobootstrap-tian-jia-ying-yong-cheng-xu-de-pei-zhi-xin-xi-applicationconfig/)
```java
   public DubboBootstrap protocols(List<ProtocolConfig> protocolConfigs) {
        if (CollectionUtils.isEmpty(protocolConfigs)) {
            return this;
        }
        for (ProtocolConfig protocolConfig : protocolConfigs) {
            protocolConfig.setScopeModel(applicationModel);
            configManager.addProtocol(protocolConfig);
        }
        return this;
    }
```

原文：[Dubbo启动器DubboBootstrap添加协议配置信息ProtocolConfig](https://blog.elastic.link/2022/07/10/dubbo/11-dubbo-qi-dong-qi-dubbobootstrap-tian-jia-xie-yi-pei-zhi-xin-xi-protocolconfig/)