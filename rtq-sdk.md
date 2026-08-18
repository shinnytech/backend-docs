# RTQ SDK 用户指南

RTQ SDK（Real-Time Quote SDK）用于在客户端中接入行情服务。客户端通过 SDK 发起订阅或查询请求，并持续读取服务端返回的 DIFF 数据，无需自行维护 WebSocket 连接。

## 功能范围

| 功能 | 说明 | 主要接口 |
| --- | --- | --- |
| 实时行情 | 订阅一个或多个合约的实时 `Quote` | `Subscribe` / `RTQSubscribe` |
| 历史行情 | 按最新数据、K 线 ID、时间焦点或交易日获取 `Kline`、`Tick` | `SetChart*` / `RTQSetChart*` |
| 合约信息 | 使用 GraphQL 查询合约信息 | `QuerySymbol` / `RTQQuerySymbol` |
| 组合历史行情 | 获取组合历史 `Kline` | `SetComboChart*` / `RTQSetComboChart*` |
| 组合实时行情 | 订阅自定义组合或交易所组合的实时 `Quote` | `SubscribeCombo`、`SubscribeExCombo` |

SDK 提供以下接入形式：

- Linux、Windows：C 接口和动态库。
- iOS、Android：通过 `gomobile` 生成的移动端接口。
- Go：原生 `lib/rtq` 接口。

## 基本使用流程

1. 按目标平台安装 SDK。
2. 创建一个 Context，并传入行情范围、客户端标识和认证信息。
3. 调用订阅或查询接口声明所需数据。
4. 持续调用 `FetchUpdate`；原生 Go 接口通过 `Ingress` 读取更新。
5. 按返回顺序将每个 DIFF 合并到客户端本地数据截面。
6. 订阅发生变化时再次调用对应接口；不再使用的数据应及时退订。

`FetchUpdate` 返回的是增量更新，不是每次都返回完整行情截面。客户端必须按照 [DIFF 协议](https://doc.shinnytech.com/diff/latest/general.html) 维护本地状态。

## 安装

### Linux / Windows（C 接口）

从 NAS 中的 `//nas.shinnytech.com/share/rtq-sdk/` 位置获取指定版本，并根据操作系统和 CPU 架构选择对应目录。

安装包主要包含：

- C 头文件：`librtq.h`。
- Linux 动态库：`librtq.so`。
- Windows 动态库及导入库：`rtq.dll`、`rtq.lib`。

### iOS（mobile 接口）

参考 [iOS(mobile接口)](https://github.com/shinnytech/rtq-sdk#iosmobile%E6%8E%A5%E5%8F%A3) 。

### Android（mobile 接口）

参考 [Android(mobile接口)](https://github.com/shinnytech/rtq-sdk#androidmobile%E6%8E%A5%E5%8F%A3) 。

### Go 原生接口

#### GOPATH 方式

将仓库放在：

```text
$GOPATH/src/shinnytech.com/rtq-sdk
```

业务代码中导入：

```go
import "shinnytech.com/rtq-sdk/lib/rtq"
```

#### GOMOD 方式

可将 rtq-sdk clone 到项目目录 lib/rtq-sdk 中（包含其自身 vendor），并在 go.mod 文件中配置：

```
replace (
	shinnytech.com/easyfuture => ./lib/rtq-sdk/vendor/shinnytech.com/easyfuture
	shinnytech.com/rtq-sdk => ./lib/rtq-sdk
)
```

可参考 https://github.com/shinnytech/microsrv-uptime 项目配置 GO MOD 方式用法。

## 初始化

### 公共参数

C 接口使用 `RTQInit`，移动端接口使用 `NewContext`。两者的主要参数一致：

| 参数 | 是否必填 | 说明 |
| --- | --- | --- |
| `scope` | 是 | `FUTURE` 表示期货行情，`STOCK` 表示微服务行情系统 |
| `addrs` | 否 | 自定义服务地址的 JSON；通常传 `NULL` 或空字符串以使用默认地址 |
| `agent` | 是 | 客户端名称和版本，格式为 `产品名 x.y.z`，例如 `myapp 1.2.3` |
| `hwid` | 是 | 硬件 ID，必须使用 UUID 格式 |
| `sid` | 是 | 会话 ID，必须是 19 位纳秒时间戳；每次新会话应生成新的值 |
| `token` | 否 | 用户认证 Token；无 Token 时传 `NULL` 或空字符串 |
| `debug` | 是 | 是否输出 Debug 日志；生产环境通常关闭 |
| `ctp` | 是 | 是否在 SDK 边界自动转换 CTP 格式与交易所原始格式的合约代码 |

- 当 `scope` 为 `FUTURE` 时，使用完整地址池，包含 MD 系统和微服务行情系统的所有地址，根据权重顺序选择地址连接。由于当前 MD 系统地址权重大于微服务行情系统，因此在一般情况下，`FUTURE` 意味着会优先连接 MD 系统，微服务行情系统地址成为灾备。

- 当 `scope` 为 `STOCK` 时，使用过滤后的不完整地址池，仅包含微服务行情系统地址，确保始终能够获取股票行情。

- `agent`、`hwid` 或 `sid` 缺失或格式不正确时，Context 创建失败。

- `ctp` 的含义：

    - `1` / `true`：客户端传入和收到的合约代码均使用 CTP 格式，SDK 自动完成转换。
    - `0` / `false`：SDK 不转换合约代码，客户端使用交易所原始格式。

- 如果设置了环境变量 `RTQ_ADDRS`，它会覆盖初始化参数中的 `addrs`。只有在测试或服务端明确提供自定义地址配置时才建议使用该选项。

## 对外接口

| 功能 | C 接口 | mobile 接口 | 说明 |
| --- | --- | --- | --- |
| 实时行情 | `RTQSubscribe` | `Subscribe` | 订阅实时 `Quote` |
| 历史行情 | `RTQSetChart*` | `SetChart*` | 获取 `Kline` 或 `Tick` |
| 合约信息 | `RTQQuerySymbol` | `QuerySymbol` | 执行 GraphQL 合约查询 |
| 组合历史行情 | `RTQSetComboChart*` | `SetComboChart*` | 获取组合 `Kline`，不提供组合 `Tick` |
| 自定义组合实时行情 | `RTQSubscribeCombo` | `SubscribeCombo` | 按权重订阅组合 `Quote` |
| 交易所组合实时行情 | `RTQSubscribeExCombo` | `SubscribeExCombo` | 按合约列表订阅组合 `Quote` |

发送请求后，通过 `RTQFetchUpdate` / `FetchUpdate` 统一读取 DIFF 更新。各接口的参数、更新及退订方式以 [C 接口定义](https://github.com/shinnytech/rtq-sdk/blob/master/lib/c/rtq.go) 和 [mobile 接口定义](https://github.com/shinnytech/rtq-sdk/blob/master/lib/mobile/rtq.go) 为准。

## C 接口代码示例

参考 [C 接口示例](https://github.com/shinnytech/rtq-sdk/blob/master/example/cli.c) 。

## Token、重连与合约代码

### 更新 Token

使用 `RTQUpdateToken` / `UpdateToken` 更新认证信息：

- `reconnect = 0` / `false`：新 Token 在下次连接时生效。
- `reconnect = 1` / `true`：断开当前连接并使用新 Token 重新连接。

用户登录、退出或 Token 刷新后，应及时调用该接口。

### 用户主动重试

当用户主动刷新、重试或重新进入行情页面时，可调用 `RTQUserInteraction` / `UserInteraction`，通知 SDK 立即处理等待中的连接重试。

### 合约代码格式转换

初始化时启用 `ctp` 后，RTQ 来源的数据会自动完成转换。对于从其他渠道获取的 JSON 行情数据，可按需调用：

- `RTQTranslateToCtp` / `TranslateToCtp`：交易所原始格式转为 CTP 格式。
- `RTQTranslateToExchange` / `TranslateToExchange`：CTP 格式转为交易所原始格式。

## 错误处理与使用建议

- 请求接口返回成功不代表服务端已经返回数据。调用方应继续读取 DIFF，并根据业务截面确认请求结果。
- 为不同图表和合约查询使用不同的 `chart_id`、`query_id`，更新同一需求时复用原 ID。
- 页面或功能退出时及时发送退订，避免保留不再需要的数据请求。
- 客户端不需要在断线后重复发送全部订阅；保持 Context 存活并继续读取更新即可。
- 生产环境应记录 SDK 日志和服务状态信息，便于定位网络、认证和服务可用性问题。
