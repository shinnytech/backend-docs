# 后台架构与系统组成

## 行情服务结构

目前行情服务包含两套相互独立的系统：微服务行情系统（股票系统）和 MD 系统（期货系统）。两套系统采用不同的服务架构，并分别基于 [`eftrade`](https://github.com/shinnytech/eftrade) 仓库的不同分支部署。

### 微服务行情系统（股票系统）

微服务行情系统基于 [`eftrade`](https://github.com/shinnytech/eftrade) 仓库的 [`instrument_serve`](https://github.com/shinnytech/eftrade/tree/instrument_serve) 分支构建。

#### 数据范围

系统包含以下交易所及数据源的行情数据：

- `CFFEX`
- `CZCE`
- `DCE`
- `GFEX`
- `SHFE`
- `INE`
- `SSE`
- `SZSE`
- `SSWE`
- `CSI`

其中，`CSI` 目前仅包含 `CSI.000907`。

#### 服务组成与职责

核心行情服务包括：

- `mdhis`：历史行情服务，负责发送 `Kline` 和 `Tick` 数据。
- `mdreal`：实时行情服务，负责发送实时盘口 `Quote` 数据。
- `ins-serve`：合约信息服务，提供 GraphQL 合约信息查询。

上述三个服务均可独立部署和访问，并分别拥有独立的代码仓库和服务入口。各仓库仅包含服务壳代码，核心功能均通过依赖库的方式复用 `instrument_serve` 分支的代码。

系统还包含以下服务：

- `combo-chart`：提供组合历史 `Kline` 行情。
- `combo-real`：提供实时组合盘口 `Quote` 数据。
- `ins-serve-backtest`：回测合约服务，为回测用户提供历史合约信息；**该服务没有独立入口**。
- `front-mobile`：常规前置服务，对外提供统一入口，后端接入 `ins-serve`、`mdreal` 和 `mdhis`。
- `front-mobile-backtest`：回测前置服务，仅供回测用户使用，后端接入 `ins-serve-backtest`、`mdreal` 和 `mdhis`。其中，`mdreal` 和 `mdhis` 与 `front-mobile` 共用。

#### 访问架构

```text
                      微服务行情系统（股票系统）

    用户可直接访问的入口

    [用户]
       |
       +--> front-mobile            (常规统一入口)
       +--> front-mobile-backtest   (回测统一入口，仅供回测用户使用)
       +--> mdhis                   (历史 Kline & Tick)
       +--> mdreal                  (实时 Quote)
       +--> ins-serve               (GraphQL 合约信息)
       +--> combo-chart             (组合历史 Kline)
       +--> combo-real              (实时组合 Quote)


    前置服务的后端连接

    front-mobile                   front-mobile-backtest
       |                              |
       +--> ins-serve                 +--> ins-serve-backtest
       +--> mdreal                    +--> mdreal
       +--> mdhis                     +--> mdhis
```

`ins-serve-backtest` 仅通过 `front-mobile-backtest` 对外提供服务，不提供独立入口。

#### 服务入口

各服务提供以下 WebSocket 入口：

| 服务 | 地址 | 备注 |
| --- | --- | --- |
| `front-mobile` | `wss://api.shinnytech.com/t/nfmd/front/mobile` | 付费用户入口 |
| `front-mobile` | `wss://free-api.shinnytech.com/t/nfmd/front/mobile` | 免费用户入口 |
| `front-mobile-backtest` | `wss://backtest.shinnytech.com/t/nfmd/front/mobile` | 回测用户入口 |
| `mdhis` | `wss://mdhis.shinnytech.com/mdhis` | 独立入口 |
| `mdreal` | `wss://mdreal.shinnytech.com/mdreal` | 独立入口 |
| `ins-serve` | `wss://ins-serve.shinnytech.com/ins-serve` | 独立入口 |
| `combo-real` | `wss://combo-real.shinnytech.com/combo-real` | 独立入口 |
| `combo-chart` | `wss://combo-chart.shinnytech.com/combo-chart` | 独立入口 |

### MD 系统（期货系统）

MD 系统采用单体架构，使用 [`eftrade`](https://github.com/shinnytech/eftrade) 仓库的 `master` 分支代码部署。与股票系统不同，期货系统不拆分为多个独立的行情微服务。

#### 数据范围

MD 系统主要提供期货行情，不包含 `SSE`、`SZSE` 和 `CSI` 数据。

#### 服务能力

行情 WebSocket 服务仅提供以下数据：

- 历史行情：`Kline` 和 `Tick`。
- 实时行情：盘口 `Quote`。

合约信息不通过行情 WebSocket 服务提供，用户需要通过合约信息文件获取。系统提供完整版和精简版两种文件，其中精简版不包含已下市合约，文件体积更小。

#### 访问架构

```text
                         MD 系统（期货系统）

    [用户]
       |
       +--> WebSocket 行情入口
       |      +--> 历史 Kline & Tick
       |      +--> 实时 Quote
       |
       +--> 完整合约信息文件（latest.json）
       |
       +--> 精简合约信息文件（latest_slim.json）
              (不包含已下市合约)
```

#### 服务入口

| 用途 | 地址 | 说明 |
| --- | --- | --- |
| 行情服务 | `wss://u.shinnytech.com/t/md/front/mobile` | 提供历史 `Kline`、`Tick` 和实时 `Quote` |
| 完整合约信息 | `https://u.shinnytech.com/t/md/symbols/latest.json` | 包含完整的合约信息 |
| 精简合约信息 | `https://u.shinnytech.com/t/md/symbols/latest_slim.json` | 不包含已下市合约，文件体积更小 |
