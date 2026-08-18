# 行情服务后台文档

## Step 1

- 阅读 [后台架构与系统组成](./infra.md) 了解后台架构与系统组成，与后端开发人员沟通，确认客户端需要使用的行情入口。

## Step 2

- 阅读 [微服务行情系统接入](./microsrv_startup.md) 对接微服务行情系统接口。

- 推荐使用 RTQ-SDK 管理行情连接，参考 [RTQ SDK 用户指南](./rtq-sdk.md)。

- 参考 [客户端标识规范](https://shinnytech.atlassian.net/wiki/spaces/BACK/pages/2519007236) 的要求规范化客户端标识信息。

## Step 3

- 阅读 [输入报文约束及 Best Practice](./service_limit.md) 了解后台服务报文约束规则以及推荐的用法。

- 阅读 [服务端流控规则及应对方案](https://shinnytech.atlassian.net/wiki/spaces/BACK/pages/2638381057) 了解后台服务流量控制规则及正确的应对策略。
