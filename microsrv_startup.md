# 微服务行情系统接入

## 连接要求

- 客户端必须开启压缩，否则可能无法正常连接行情

## 接口文档

微服务行情系统当前的所有服务均基于 [DIFF 协议](https://doc.shinnytech.com/diff/latest/general.html) 实现，必须首先参考 DIFF 协议要求进行后续开发工作。

### 合约信息服务（ins-serve）

该服务提供 GraphQL 接口的合约信息。

文档：

- [GraphQL 合约信息接口文档](https://shinnytech-eftrade.readthedocs-hosted.com/en/instrument_serve/ref/graphql.html)

交互式调试工具，**可通过该工具中的 Docs 组件获取字段的定义**：

- [GraphiQL 在线调试工具](https://shinny-graphiql-ins-serve-demo.oss-cn-shanghai.aliyuncs.com/)

### 实时行情服务（mdreal）

该服务提供 Quote 行情。

- [实时行情接口文档](https://shinnytech-eftrade.readthedocs-hosted.com/en/instrument_serve/eftrade/api.html)

### 历史行情服务（mdhis）

该服务提供历史 Kline & Tick 数据。

- [历史行情接口文档](https://shinnytech-eftrade.readthedocs-hosted.com/en/instrument_serve/eftrade/api.html)

## 注意事项

