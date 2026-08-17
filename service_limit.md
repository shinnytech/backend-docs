# 输入报文约束

## subscribe_quote

- `ins_list` 合约数量不应超过 2000，大于 2000 应当拆分多连接

## set_chart

- `ins_list` 合约数量不应超过 20
- 同一个连接下订阅的 chart 数量不得超过 10000

## ins_query

- 每一个 `ins_query` 查询结束、客户端收到对应回包后，都应发送退订报文

## set_combo_chart

- `weights` 合约数量不应超过 20
- 同一个连接下订阅的 chart 数量不应超过 20