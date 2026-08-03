# 微服务行情系统接口文档

微服务行情系统当前的所有服务均基于 DIFF 协议实现，具体协议说明参阅文末的 [Reference](#reference)。

## 接口文档

### 合约信息服务（ins-serve）

该服务按照 DIFF 协议要求实现，提供 GraphQL 接口的合约信息。

- [GraphQL 合约信息接口文档](https://shinnytech-eftrade.readthedocs-hosted.com/en/instrument_serve/ref/graphql.html)
- [GraphiQL 在线调试工具](https://shinny-graphiql-ins-serve-demo.oss-cn-shanghai.aliyuncs.com/)

### 实时行情服务（mdreal）

该服务按照 DIFF 协议要求实现，提供 Quote 行情。

- [实时行情接口文档](https://shinnytech-eftrade.readthedocs-hosted.com/en/instrument_serve/eftrade/api.html)

### 历史行情服务（mdhis）

该服务按照 DIFF 协议要求实现，提供历史 Kline & Tick 数据。

- [历史行情接口文档](https://shinnytech-eftrade.readthedocs-hosted.com/en/instrument_serve/eftrade/api.html)

## Reference

- [DIFF 协议](https://doc.shinnytech.com/diff/latest/general.html)
