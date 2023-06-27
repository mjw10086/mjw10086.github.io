---
title:  "《gRPC: Up And Running》读书笔记"
toc: true
date: 2023-06-27
tags: 
- 软件开发
- 框架学习
categories: 
- 软件开发
---

书的内容不多，是我最喜欢的o'reilly出版社出版的，gRPC不算复杂，但是关键是要应用。

<!--more-->

# 概述

gRPC主要应用在微服务架构中服务间通信，其竞争者有REST、MQ，gRPC的主要优点就是和程序语言融合紧密，相比起REST来说有种强弱类型语言对比的感觉，而相比于MQ来说，缺少了MQ的持久化机制，尽管gRPC是今天的主角，但是很多情况下，gRPC不能替代REST和MQ。向外供人调用的端口使用REST更合适，因为REST广泛且学习成本低，而在需要持久化的情况下，gRPC也替代不了MQ。

# hello world程序

搭建一个gRPC应用的步骤很简单：

1. 编写proto文件定义接口的方法和消息格式
2. 使用protoc等工具生成对应的Protocol Buffers相关的源代码
3. 在具体项目中根据生成的源代码编写服务器或者客户端来实现监听或者通信

由于框架版本的问题，网上很多教程包括本书的hello world教程都不能正常跑，在这里需要初学者花一些功夫解决这个问题。

# 基础概念

针对gRPC提供了四种通信模式，分别是Unary RPC、Server Streaming、Client Streaming 和 Bidirectional Streaming，主要区别在于请求和相应的数量问题，Unary RPC采用的是一个请求一个相应的模式，而Streaming系列则是代表多个消息，因此Server Streaming就客户端一个请求，而服务器返回多个响应，以此类推理解其他的通信模式。在这里的关键是要理解四种通信模式的使用场景，在实际项目中合理选择。

另外gRPC还有三种传输实现，分别是HTTP/2.0、Cronet和in-process，一般来说都使用HTTP/2.0，Cronet是针对移动端的优化实现。

gRPC的具体实现细节并不需要了解太深，主要是如何编码，在这里暂时省略。

# 功能支持

gRPC在基础的通信基础上提供了额外的功能支持：

- interceptor（拦截器）：拦截器有两种类型，分别是Unary和Stream，两者的函数签名并不相同，拦截器的主要作用相当于web中的中间件。而且要注意，拦截器是从后向前依次执行的。
- deadline（超时）：deadline和timeoute是两种机制，deadline是确定时间点，而timeout是时间段，而且设置的位置也不一样，两者都可以设置在客户端，但是deadline主要是发送给服务器，而timeout则一般都是在客户端设置。两者可以结合使用。
- cancellation（取消）：cancellation是通过Context设置的，没什么额外要说的
- error handling（错误处理）：服务端返回错误是利用status，客户端则根据status的code来做处理
- multiplexing（多路服用）：这里的mutiplexing就是在一个gRPC服务器上注册多个gRPC服务
- metadata（元数据）：十分类似于在http的header上添加自定义的key value
- Name Resolver（域名解析）：使用域名而不是IP来进行通信
- load balancing（负载均衡）：负载均衡主要分为两种实现，客户端实现和服务器端代理实现，前者并不会知晓服务器的情况，因此有局限性，而服务器端代理则可以利用nginx这样的代理软件来实现
- compression（压缩）：利用gzip来对消息进行压缩，节省带宽

# 安全性

关于gRPC的安全性，主要依赖于两个部分，TLS和用户认证，TLS支持单向TLS和双向TLS，关于TLS的知识建议自行了解。用户认证也很简单，主要就是通过metadata传输认证信息，再结合interceptor来实现认证。具体的几种认证手段则不多说了。

# 生产环境

关于测试核心注意的就是使用mock工具做单元测试，还有就是ghz可以做负载测试。部署在docker和k8s则属于相关工具的使用，不谈了。

在程序运行中的观测是十分重要的，利用OpenCensus和Prometheus，可以观测特定指标并进行tracing功能，日志系统则配合interceptor工作。

# 生态系统

在本章介绍了几个工具，分别是：

- gRPC Gateway：将REST转化为gRPC服务
- Transcoding：将REST请求转化为gRPC请求
- Reflection Protocol：通过反射来了解服务器的服务和消息定义
- Middleware：也就是interceptor的功能
- Health Checking / Health Probe

当个了解，然后各取所需
