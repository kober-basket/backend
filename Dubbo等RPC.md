# Dubbo

![](https://cn.dubbo.apache.org/imgs/v3/concepts/architecture-2.png)
Dubbo是阿里开源的RPC框架，整体架构分两层：服务治理抽象控制面、数据面。
- **服务治理控制面**。服务治理控制面不是特指如注册中心类的单个具体组件，而是对 Dubbo 治理体系的抽象表达。控制面包含协调服务发现的注册中心、流量管控策略、Dubbo Admin 控制台等，如果采用了 Service Mesh 架构则还包含 Istio 等服务网格控制面。
- **Dubbo 数据面**。数据面代表的是通过集群部署的所有Dubbo进程。这些进程之间通过特定的 RPC 协议实现数据交换，Dubbo 定义了**微服务应用开发与调用规范**，并负责完成**数据传输的编解码工作**。
    - **服务消费者Consumer**：发起业务调用或 RPC 通信的 Dubbo 进程
    - **服务提供者Provider**：接收业务调用或 RPC 通信的 Dubbo 进程

注册中心（一般用nacos、zookeeper等）
服务治理
服务提供者
服务消费者
通信协议

## Dubbo架构演进
| 趋势            | Dubbo 2.x             | Dubbo 3.x                                 |
|-----------------|-----------------------|-------------------------------------------|
| 架构风格        | 单体微服务架构         | 云原生架构                                 |
| 通信协议        | Dubbo 协议为主         | Triple（兼容 gRPC）成为主流                |
| 注册中心        | ZooKeeper              | 推荐 Nacos                                 |
| 服务治理        | Dubbo 自身实现         | 可对接 Istio、Sentinel 等                  |
| 多语言支持      | Java 为主             | 多语言友好（Triple + Protobuf）            |
| 配置管理        | 无统一方案             | 推荐 Nacos 配置中心                         |
| 生态融合        | Dubbo 独立生态         | Spring Boot、Kubernetes 兼容性强            |


## Dubbo的RPC协议
Dubbo 从设计上不绑定任何一款特定通信协议，能够支持`HTTP/2、REST、gRPC、JsonRPC、Thrift、Hessian2` 等几乎所有主流的通信协议，可根据需要选择不同的通信协议。
> 也可将内部私有协议接入到Dubbo体系。如阿里通过扩展支持HSF协议，实现内部HSF框架→Dubbo3框架的整体迁移。

![](https://cn.dubbo.apache.org/imgs/v3/what/protocol.png)

## 与HSF框架对比
HSF框架的架构，[参考文档](https://developer.aliyun.com/article/1529236)
![HSF架构](https://ucc.alicdn.com/pic/developer-ecology/ul7uyfipjg3ku_9748908c4f1546d48983cd8c21d21493.png?x-oss-process=image%2Fresize%2Cw_1400%2Cm_lfit%2Fformat%2Cwebp)
### HSF的三种调用方式
1. 同步调用
2. 异步调用
![](https://ucc.alicdn.com/pic/developer-ecology/ul7uyfipjg3ku_7c36fd62d90d43a1a73c71893a188ad8.png?x-oss-process=image%2Fresize%2Cw_1400%2Cm_lfit%2Fformat%2Cwebp)
3. CallBack异步调用
![](https://ucc.alicdn.com/pic/developer-ecology/ul7uyfipjg3ku_3d4f5099136b4e7b8bd2f589aca855ea.png?x-oss-process=image%2Fresize%2Cw_1400%2Cm_lfit%2Fformat%2Cwebp)




# RPC协议
## 常见RPC协议对比
| 协议名称      | 底层传输协议 | 数据格式       | IDL 支持 | 交互模型                  | 跨语言支持 | 典型框架/系统         | 优点                                 | 缺点                                  |
|---------------|----------------|----------------|-----------|----------------------------|--------------|--------------------------|--------------------------------------|---------------------------------------|
| Dubbo Protocol | TCP           | 自定义二进制   | 无        | 同步请求-响应              | Java 为主    | Apache Dubbo             | 高性能、成熟稳定                     | 扩展性差，跨语言支持弱               |
| gRPC          | HTTP/2         | Protobuf（默认）| `.proto`  | unary, server-streaming, client-streaming, bidirectional | 强        | gRPC 官方、Dubbo3 Triple | 高性能、跨语言、标准化协议           | Protobuf 学习成本高，调试较麻烦     |
| Triple (Dubbo3) | HTTP/2        | Protobuf（默认）| `.proto`  | unary, server-streaming, client-streaming, bidirectional | 强 | Apache Dubbo3            | 兼容 gRPC，支持 Dubbo 生态，云原生友好 | 初期生态还在完善中                   |
| JSON-RPC      | HTTP           | JSON           | 无        | 同步请求-响应              | 强           | FastJSON、Netty JSON-RPC | 简单、易读、调试方便                 | 性能不如二进制协议                   |
| RESTful HTTP  | HTTP/1.1       | JSON/XML       | 无        | 请求-响应（同步）          | 强           | Spring Cloud Feign/Ribbon | 易调试、易集成浏览器和移动端         | 性能较低，不适合高频调用             |
| Hessian       | HTTP/TCP       | 自定义二进制   | 无        | 同步请求-响应              | 中           | Dubbo（早期版本）        | 轻量级、Java 友好                    | 不够标准化，跨语言支持一般           |




gRPC基于**HTTP/2和protoBuf**，支持多路复用、全双工流式、标头压缩、同步异步处理。因此gRPC使用更少的网络资源，拥有更高性能。

![gRPC架构图](https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/6761552565f04c039c970312_6149d279ba7cdebc475a9621_gRPC%2520Architecture.png)

## 协议对比
Triple 协议 = HTTP/2 + Protobuf + Dubbo 扩展元数据
gRPC 协议 = HTTP/2 + Protobuf

**协议基本定义**
|特性       |	Triple             |	gRPC                    |
|---        |---                   |--                          |
|定义	    |Dubbo 自研的 RPC 协议  |	Google 开源的标准 RPC 协议|
|所属生态	|Apache Dubbo 生态      |	Google 开源项目，CNCF 成员|
|设计目标	|兼容 gRPC 语义并扩展 Dubbo 功能|	跨语言、高性能、标准化的 RPC 框架|

**协议结构对比**
| 层级               | Triple                                | gRPC                                 |
|--------------------|----------------------------------------|--------------------------------------|
| 底层协议           | HTTP/2                                | HTTP/2                               |
| 数据格式           | Protobuf（默认）                      | Protobuf（默认）                     |
| 接口描述语言（IDL）| `.proto` 文件                         | `.proto` 文件                        |
| 交互模型           | 支持 unary, server-streaming, client-streaming, bidirectional-streaming | 支持 unary, server-streaming, client-streaming, bidirectional-streaming |
| 元数据             | 支持 Dubbo 扩展元数据（如路由标签、分组、超时设置等） | 标准 HTTP Headers（Metadata）        |
| 编码方式           | 默认使用 Protobuf，可扩展其他序列化方式 | 默认使用 Protobuf，也支持 JSON 等    |


## protoBuf协议


# RPC协议——Triple


