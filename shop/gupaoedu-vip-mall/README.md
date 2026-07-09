# gpMall 微服务商城项目

## 项目简介

本项目是一个基于 Spring Boot 与 Spring Cloud Alibaba 的电商微服务系统，围绕商城常见业务进行模块拆分，包含商品、用户、购物车、订单、支付、权限、秒杀、页面展示、网关等核心能力。

项目采用 Maven 多模块结构，通过 Nacos 完成服务注册与配置管理，通过 Spring Cloud Gateway 提供统一入口，通过 OpenFeign 完成服务间调用，并结合 Sentinel、JWT、Redis、RocketMQ 等技术实现网关治理、权限认证、缓存加速和秒杀削峰。

## 项目结构

```text
gupaoedu-vip-mall
├── mall-api
│   ├── mall-cart-api              # 购物车 API 与模型
│   ├── mall-dw-api                # 数据统计相关 API
│   ├── mall-goods-api             # 商品 API 与模型
│   ├── mall-order-api             # 订单 API 与模型
│   ├── mall-page-api              # 页面生成 API
│   ├── mall-pay-api               # 支付 API 与模型
│   ├── mall-permission-api        # 权限 API 与模型
│   ├── mall-search-api            # 搜索 API 与模型
│   ├── mall-seckill-api           # 秒杀 API 与模型
│   └── mall-user-api              # 用户 API 与模型
├── mall-gateway
│   └── mall-api-gateway           # API 网关，负责路由、鉴权、限流、秒杀入口
├── mall-service
│   ├── mall-canal-service         # 数据变更监听与同步服务
│   ├── mall-cart-service          # 购物车服务
│   ├── mall-dw-service            # 数据统计服务
│   ├── mall-file-service          # 文件服务
│   ├── mall-goods-service         # 商品服务
│   ├── mall-order-service         # 订单服务
│   ├── mall-pay-service           # 支付服务
│   ├── mall-permission-service    # 权限服务
│   ├── mall-search-service        # 搜索服务
│   ├── mall-seckill-service       # 秒杀服务
│   └── mall-user-service          # 用户服务
├── mall-util
│   ├── mall-common                # 通用工具类、统一响应、JWT、分页等
│   └── mall-service-dependency    # 微服务公共依赖与基础配置
├── mall-web
│   ├── mall-page-web              # 商品详情页、秒杀详情页生成
│   └── mall-search-web            # 商城搜索页面
├── sql                            # 数据库脚本
└── pom.xml                        # Maven 父工程
```

## 技术栈

| 技术 | 用途 |
| --- | --- |
| Spring Boot | 微服务基础开发框架 |
| Spring Cloud Alibaba | 微服务治理基础 |
| Nacos | 服务注册发现、配置管理 |
| Spring Cloud Gateway | 统一 API 网关、路由转发 |
| OpenFeign | 微服务之间的声明式 HTTP 调用 |
| Sentinel | 网关限流、流量控制 |
| JWT | 用户身份认证与令牌校验 |
| MySQL | 业务数据持久化 |
| MyBatis-Plus | 数据访问层开发 |
| Redis | 缓存、权限数据缓存、秒杀库存、排队标识 |
| Redisson | 分布式锁、布隆过滤器等 Redis 高级能力 |
| RocketMQ | 消息异步处理，支撑秒杀排队削峰 |


## 核心模块说明

### mall-api

公共 API 契约层，主要存放各业务模块的实体模型和 Feign 接口。其他服务通过依赖对应 API 模块完成远程调用，减少重复定义。

### mall-gateway

统一网关入口，主要职责包括：

- 根据请求路径转发到不同微服务。
- 使用 JWT 校验用户身份。
- 从 Redis 读取权限数据并进行接口权限判断。
- 使用 Sentinel 对网关路由进行限流。
- 对秒杀请求进行前置判断和排队处理。

### mall-service

业务服务层，按照领域拆分为商品、用户、订单、支付、权限、秒杀等服务。每个服务可以独立启动、独立注册到 Nacos，并通过 OpenFeign 调用其他服务。

### mall-util

公共工具与公共依赖模块，包含统一响应对象、JWT 工具、MD5 工具、分页对象、Redis 配置等通用能力。



## 技术亮点

### 1. 微服务模块拆分清晰

项目按照 API 契约、业务服务、网关、前台页面、公共工具进行拆分。业务服务之间通过 OpenFeign 调用，服务注册与配置统一交给 Nacos 管理，整体结构符合常见微服务商城项目的分层方式。

### 2. Spring Cloud Gateway 统一入口

所有外部请求先进入网关，再由网关根据路由规则转发到具体服务。网关层集中处理鉴权、权限校验、限流、秒杀入口控制等通用逻辑，减少业务服务重复实现。

### 3. JWT + Redis 实现认证与权限校验

项目使用 JWT 作为用户身份令牌，网关解析 token 后进行身份校验。权限数据缓存在 Redis 中，网关可以快速判断用户角色是否拥有目标接口访问权限。

### 4. Sentinel 实现网关限流

项目在网关层接入 Sentinel，对指定路由和 API 分组配置流量控制规则。当请求量超过阈值时，Sentinel 可以在入口层拦截流量，保护后端服务。

### 5. Redis 支撑高频访问场景

Redis 在项目中承担多种职责：

- 缓存权限数据，提升网关权限校验速度。
- 缓存热点秒杀商品库存，减少数据库压力。
- 保存用户秒杀排队标识，避免重复排队。
- 配合 Redisson 实现分布式锁和布隆过滤器。

### 6. RocketMQ 实现秒杀削峰

秒杀请求进入网关后，热点商品请求会先写入 Redis 排队标识，再发送到 RocketMQ 队列。秒杀服务异步消费消息并创建订单，从而把瞬时高并发请求转化为后端可控的异步处理流程。





## 总结

本项目通过 Spring Cloud Alibaba 生态完成了一个商城微服务系统的基础搭建，涵盖服务注册发现、网关路由、接口鉴权、权限缓存、接口限流、服务间调用、秒杀削峰和页面静态化等能力，适合作为学习微服务商城架构和高并发秒杀设计的参考项目。
