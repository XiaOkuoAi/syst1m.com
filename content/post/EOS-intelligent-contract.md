---
title: "EOS Intelligent Contract"
date: 2019-08-02T12:43:34+08:00
lastmod: 2019-08-02T12:43:34+08:00
draft: false
tags: [智能合约]
categories: [智能合约]
comment: true
---
EOS智能合约测试
<!--more-->
# EOS智能合约测试
## 整体步骤功能大概如下 ##

![](https://ae01.alicdn.com/kf/Hd2932f14fc1d4978a1b645938bfb151dx.jpg)

### 本地钱包 ###
- 创建钱包

```
./cleos wallet create -n 钱包名 --file ./save.txt
```

- 开启钱包

```
cleos wallet unlock -n 钱包名 --password 密钥
```
- 账户导入私钥

```
cleos wallet import -n 钱包名 --private-key 私钥
```

- 查看钱包导入的密钥

```
cleos wallet keys
```

- 激活钱包

```
cleos wallet unlock -n jiji --password 密钥
```
- 删除钱包

```
rm -f ~/eosio-wallet/*.wallet
```
### 代币管理 ###

- 添加新代币

```
cleos push action 合约账户 addtoken '{"ext_symbol":{"symbol":"8,FEOS","contract":"发行FEOS账户"},"memo":"FEOS create by 发行FEOS账户"}' -p 合约账户@backmange
```

- 查询代币表

```
cleos get table 合约账户 合约账户 tokeninfotb
```
- 更新代币

```
cleos push action 合约账户 updatetoken '{"id":0, ext_symbol":{"symbol":"8,FEOS","contract":"发行FEOS账户"},"memo":"FEOS create by 发行FEOS账户"}' -p 合约账户@backmange
```

- 删除代币

```
cleos push action 合约账户 deltoken '{"id": 0}' -p 合约账户@backmange
```

### 交易对管理 ###
- 添加交易对

```
cleos push action 合约账户 addtokenpair '{"base_id":0, "quote_id":1, "price_precision": 7, "volume_precision": 2, "min_volume_amount": 1000000, "pair_status": 1, "memo":"addtokenpair test"}' -p 合约账户@backmange
```

- 查询交易对表

```
cleos get table 合约账户 合约账户 tokenpairtb
```

- 更新币对

```
cleos push action 合约账户 uptokenpair '{"id":1, "price_precision": 7, "volume_precision": 2, "min_volume_amount": 1000000, "pair_status": 1, "memo":"addtokenpair test"}' -p 合约账户@backmange
```

- 删除币对

```
cleos push action 合约账户 deltokenpair '{"id":0}' -p 合约账户@backmange
```

### 交易单管理 ###
- 发送买单

```
cleos push action 发行FBTC账户 transfer '["挂单账户", "交易所合约账户","100.00000000 FBTC","{\"transfer_type\":1,\"token_pair_id\":0,\"unit_price_amount\":1000,\"order_type\":1,\"order_memo\":\"\"}"]' -p 挂单账户@active -p 合约账户@multisig
```

- 查询买单表

```
cleos get table 合约账户 交易对ID ordertb
```
- 发送卖单

```
cleos push action 发行FUSDT账户 transfer '["挂单账户", "交易所合约账户","100.00000000 FUSDT","{\"transfer_type\":2,\"token_pair_id\":0,\"unit_price_amount\":1000,\"order_type\":1,\"order_memo\":\"\"}"]' -p 挂单账户@active -p 合约账户@multisig
```

- 查询卖单表

```
cleos get table 合约账户 交易对ID ordertb
```
### 订单撮合 ###
- 授权code权限

```
cleos set account permission 合约账户 active '{"threshold": 1,"keys": [{"key": "EOS7oR9AvZVu7sgwUCmPDyuKfpJEt7qFXPrHfLoHjh7rUigQZNoCM","weight": 1}],"accounts": [{"permission":{"actor":"合约账户","permission":"eosio.code"},"weight":1}]}' owner -p 
```
- 执行限价撮合

```
cleos push action 合约账户 matchlimit '{"buy_orderid":2, "sell_orderid":3, "token_pair_id": 0}' -p 合约账户@multisig
```

- 查看执行结果

```
cleos push action 合约账户 matchlimit '{"buy_orderid":2, "sell_orderid":3}' -p 合约账户@active
```

- 查询卖单
 
```
cleos get table 合约账户 交易对ID ordertb
```
- 查询买单

```
cleos get table 合约账户 交易对ID ordertb
```
- 插销订单

```
cleos push action 合约账户 cancelorder '{capi_name user, uint64_t orderid, uint64_t token_pair_id, uint8_t trans_type}' -p 挂单账户@active -p 合约账户@multisig
```

### 交易额管理 ###
- 衡量增加

```
cleos push action 合约账户 addeamount '{"ext_symbol":{"symbol":"8,FEOS","contract":"发行FEOS账户"}, "unit_limit_amount":1000}' -p 合约账户@backmange
```

- 查询elimittb表

```
cleos get table 合约账户 合约账户 elimittb
```
- 衡量更新

```
cleos push action 合约账户 upeamount '{"ext_symbol":{"symbol":"8,FEOS","contract":"发行FEOS账户"}, "unit_limit_amount":1000}' -p 合约账户@backmange
```

- 衡量删除

```
cleos push action 合约账户 deleamount '{"ext_symbol":{"symbol":"8,FEOS","contract":"发行FEOS账户"}}' -p 合约账户@backmange
```

### 抵押增加额度计算管理 ###
- 计算更新

```
cleos push action 合约账户 updamount '{"unit_limit_amount": 100}' -p 合约账户@active
```

- 查询抵押增加额度表

```
cleos get table 合约账户 合约账户 dlimittb
```

### 抵押操作 ###
- 抵押平台代币

```
cleos push action 代币发行合约账户 transfer '["抵押账户", "合约账户","100.00000000 EOS","{\"transfer_type\":3}"]' -p 抵押账户 -p 多签账户@active
```

- 查询账户额度数据

```
cleos get table 合约账户 合约账户 accounttb
```

- 查看已抵押数据

```
cleos get table 合约账户 合约账户 dtokentb
```

- 执行取消抵押

```
cleos push action 合约账户 undelegate '{"account": "抵押账户", "delegate_asset": "10.00000000 EOS"}' -p 抵押账户@active -p 合约账户@multisig
```

- 赎回到期

```
cleos push action 合约账户 refund '{"account": "抵押账户"}' -p 抵押账户@active -p 合约账户@multisig
```

### 交易及挖矿 ###
- 获取奖励

```
cleos push action 合约账户 claimrewards '{"account": "赎回账户"}' -p 赎回账户@active -p 合约账户@multisig
```
### 提取场外币 ###
- 转出场外币

```
cleos push action 发币合约账户 transfer '["提币测试账户", "合约账户","100.00000000 FBTC","{\"transfer_type\":4,\"to_name\":\"testaccount\"}"]' -p 提币测试账户 -p 多签账户@active
```

- 修改提币记录状态

```
cleos push action 合约账户 changeextst '{"id": 111, "status": 1}' -p 交易所合约账户@backmange
```
- 删除提取记录

```
cleos push action 合约账户 delextract '{"id": 111}' -p 交易所合约账户@backmange
```

- 查询提币记录

```
cleos get table 合约账户 合约账户 extracttb
```

### 合约表 ###
#### systemtb （系统设置表）

| 键值             | 类型       | 注释                                                     | 备注 |
| ---------------- | ---------- | -------------------------------------------------------- | ---- |
| id               | uint64_t   | 序号                                                     | 自增 |
| status           | uint8_t    | 根据系统的战备等级，根据设定规则，关闭合约内相应的Action |      |
| update_timepoint | time_point | 更新时间                                                 |      |

##### status （系统状态）

| 战备级别              | 类型    | 值   | 备注 |
| --------------------- | ------- | ---- | ---- |
| COMBAT_LEVEL_NORMAL   | uint8_t | 1    | 正常 |
| COMBAT_LEVEL_LOW      | uint8_t | 2    | 低级 |
| COMBAT_LEVEL_MIDDLE   | uint8_t | 3    | 中级 |
| COMBAT_LEVEL_HIGH     | uint8_t | 4    | 高级 |
| COMBAT_LEVEL_CRITICAL | uint8_t | 5    | 危急 |



#### global （全局数据）

| 键值                    | 类型       | 注释                     | 备注 |
| ----------------------- | ---------- | ------------------------ | ---- |
| id                      | uint64_t   | 序号                     | 自增 |
| last_total_limit_amount | uint64_t   | 上一次时累计的交易总额度 |      |
| update_timepoint        | time_point | 更新时间                 |      |

#### orderidtb （为计算唯一订单ID）

| 键值     | 类型      | 注释       | 备注           |
| -------- | --------- | ---------- | -------------- |
| id       | uint64_t  | 序号       | 只保留一条记录 |
| order_id | uint128_t | 唯一订单ID |                |



#### tokeninfotb （币种信息表）

| 键值             | 类型            | 注释                                                  | 备注 |
| ---------------- | --------------- | ----------------------------------------------------- | ---- |
| id               | uint64_t        | 代币id                                                | 自增 |
| ext_symbol       | extended_symbol | 包含代币符号，代币精度，代币合约                      |      |
| token_score      | uint8_t         | 代币评分(0-10) {从0到10，分数越低，属性越差，默认是5} |      |
| create_timepoint | time_point      | 创建时间点                                            |      |
| update_timepoint | time_point      | 创建或修改时间点                                      |      |
| is_exist_quote   | uint8_t         | 是否存在计价交易区，默认0                             |      |
| memo             | std::string     | 代币备注                                              |      |

##### token_score （RATING_TOKEN_VALUE 代币评分）

| 代币评分      | 类型    | 值   | 备注                      |
| ------------- | ------- | ---- | ------------------------- |
| TOKEN_SCORE_0 | uint8_t | 0    | 被DEX拉黑的代币，禁止交易 |
| TOKEN_SCORE_1 | uint8_t | 1    |                           |
| TOKEN_SCORE_2 | uint8_t | 2    |                           |
| TOKEN_SCORE_3 | uint8_t | 3    |                           |
| TOKEN_SCORE_4 | uint8_t | 4    |                           |
| TOKEN_SCORE_5 | uint8_t | 5    | 默认值                    |
| TOKEN_SCORE_6 | uint8_t | 6    |                           |



#### tokenpairtb （交易币对信息表）

| 键值              | 类型        | 注释                           | 备注 |
| ----------------- | ----------- | ------------------------------ | ---- |
| id                | uint64_t    | 交易币对id                     |      |
| base_id           | uint64_t    | 基准货币id                     |      |
| quote_id          | uint64_t    | 计价货币id                     |      |
| price_precision   | uint8_t     | 价格精度位数{计价货币}         |      |
| volume_precision  | uint8_t     | 数量精度位数{基准货币}         |      |
| min_volume_amount | uint64_t    | 最小交易数量                   |      |
| pair_status       | uint8_t     | 交易对状态 1启用，2暂停，3停止 |      |
| create_timepoint  | time_point  | 创建时间点                     |      |
| update_timepoint  | time_point  | 修改时间点                     |      |
| memo              | std::string | 交易对备注                     |      |

##### pair_status （交易对状态）

| 状态值      | 值   | 注释     | 备注 |
| ----------- | ---- | -------- | ---- |
| PAIR_NORMAL | 1    | 普通状态 |      |
| PAIR_PAUSE  | 2    | 暂停状态 |      |
| PAIR_STOP   | 3    | 停止状态 |      |



####  ordertb （交易单数据 buyordertb/sellordertb ）

| 键值              | 类型             | 注释                 | 备注 |
| ----------------- | ---------------- | -------------------- | ---- |
| id                | uint128_t        | 订单号，唯一         |      |
| transaction_id    | capi_checksum256 | 交易单id             |      |
| transfer_type     | uint8_t          | 交易类型 （买/卖）   |      |
| token_pair_id     | uint64_t         | 币对id               |      |
| creator           | capi_name        | 创建账户名           |      |
| unit_price_amount | uint64_t         | 下单单价             |      |
| balance_amount    | uint64_t         | 剩余未成交数量       |      |
| total_amount      | uint64_t         | 下单总数量           |      |
| order_type        | uint8_t          | 订单类型 (市价/限价) |      |
| status            | uint8_t          | 订单状态             |      |
| update_timepoint  | time_point       | 更新时间             |      |
| create_timepoint  | time_point       | 创建时间             |      |
| order_memo        | std::string      | 订单备注             |      |

##### order_type （交易单类型）

| 交易单类型        | 类型    | 值   | 备注 |
| ----------------- | ------- | ---- | ---- |
| ORDER_LIMIT_TYPE  | uint8_t | 1    | 限价 |
| ORDER_MARKET_TYPE | uint8_t | 2    | 市价 |

##### status （订单状态）

| 交易单状态     | 类型    | 值   | 备注                                          |
| -------------- | ------- | ---- | --------------------------------------------- |
| ORDER_PENDING  | uint8_t | 1    | 交易等待 {从未交易过}                         |
| ORDER_IN_TRANS | uint8_t | 2    | 交易进行 {部分交易}                           |
| ORDER_PAUSE    | uint8_t | 3    | 交易暂停 {预留}                               |
| ORDER_STOP     | uint8_t | 4    | 交易停止 {预留}                               |
| OEDER_ERROR    | uint8_t | 5    | 交易出错 {预留}                               |
| OEDER_CANCEL   | uint8_t | 6    | 交易撤销                                      |
| ORDER_FINISH   | uint8_t | 7    | 交易完成 {短期保留，后面通过clearram定期清理} |



#### freezeinfotb （交易下单时记录用户冻结资金的结构）

| 键值             | 类型       | 注释       | 备注 |
| ---------------- | ---------- | ---------- | ---- |
| orderid          | uint128_t  | 订单id     |      |
| freeze_amount    | uint64_t   | 冻结数量   |      |
| update_timepoint | time_point | 更新时间   |      |
| create_timepoint | time_point | 创建时间点 |      |



#### accounttb （账户信息）

| 键值                    | 类型       | 注释                                                   | 备注 |
| ----------------------- | ---------- | ------------------------------------------------------ | ---- |
| account_name            | capi_name  | 账户名                                                 |      |
| account_type            | uint8_t    | 账号类型：0, 普通账号; 1, 合约账号                     |      |
| normal_account_score    | uint8_t    | 从0到10，分数越低，账号属性越差，默认是5               |      |
| code_account_score      | uint8_t    | 合约账号评分：从0到10，分数越低，账号属性越差，默认是5 |      |
| balance_limit_amount    | uint64_t   | 交易额剩余量                                           |      |
| total_limit_amount      | uint64_t   | 交易额总量                                             |      |
| limit_update_time_point | time_point | 交易额最后刷新                                         |      |
| perblock_bucket_asset   | asset      | 待领取的出块奖励                                       |      |
| last_claim_time_point   | time_point | 最后一次领取奖励的时间                                 |      |

##### account_type （RATING_ACCOUNT_TYPE 账号类型）

| 账号类型            | 类型    | 值   | 备注                        |
| ------------------- | ------- | ---- | --------------------------- |
| NORMAL_ACCOUNT_TYPE | uint8_t | 1    | 普通账户                    |
| CODE_ACCOUNT_TYPE   | uint8_t | 2    | 合约账户 （由后台过滤更新） |

##### account_score （RATING_ACCOUNT_VALUE 账户评分）

| 账号评分        | 类型    | 值   | 备注                                             |
| --------------- | ------- | ---- | ------------------------------------------------ |
| ACCOUNT_SCORE_0 | uint8_t | 0    | 被BP拉黑或已知恶意合约账号，如实施过合约攻击行为 |
| ACCOUNT_SCORE_1 | uint8_t | 1    |                                                  |
| ACCOUNT_SCORE_2 | uint8_t | 2    | 拉黑的羊毛党账号                                 |
| ACCOUNT_SCORE_3 | uint8_t | 3    | 疑似羊毛党账号                                   |
| ACCOUNT_SCORE_4 | uint8_t | 4    |                                                  |
| ACCOUNT_SCORE_5 | uint8_t | 5    | 默认值                                           |
| ACCOUNT_SCORE_6 | uint8_t | 6    | 通过KYC人脸比对验证                              |



#### elimittb （成交消耗额度衡量数据）

| 键值              | 类型            | 注释               | 备注 |
| ----------------- | --------------- | ------------------ | ---- |
| ext_symbol        | extended_symbol | 代币扩展符号       |      |
| unit_limit_amount | uint64_t        | 单个成交消耗交易额 |      |



#### dlimittb （抵押增加额度计算数据 限定只支持平台币）

| 键值              | 类型     | 注释                   | 备注 |
| ----------------- | -------- | ---------------------- | ---- |
| id                | uint64_t | 序号                   |      |
| unit_limit_amount | uint64_t | 单个平台币获取的交易额 |      |



#### dtokentb （抵押赎回币记录）

| 键值               | 类型       | 注释       | 备注 |
| ------------------ | ---------- | ---------- | ---- |
| account_name       | capi_name  | 抵押账户名 |      |
| balance_asset      | asset      | 抵押余额   |      |
| total_asset        | asset      | 抵押总数   |      |
| undelegate_asset   | asset      | 取消抵押数 |      |
| request_time_point | time_point | 请求时间   |      |



#### brewardtb （交易挖矿奖励）

| 键值                        | 类型                               | 注释                           | 备注                         |
| --------------------------- | ---------------------------------- | ------------------------------ | ---------------------------- |
| id                          | uint64_t                           | 序号                           |                              |
| current_block_id            | uint64_t                           | 当前所在块id                   | 已替换，先做后期功能扩展保留 |
| perblock_total_bucket_asset | asset                              | 总的待分配的出块奖励           |                              |
| account_amounts             | vector<account_transaction_amount> | 期间内交易各自账户消耗的交易额 |                              |
| update_timepoint            | time_point                         | 更新时间                       |                              |

#### account_transaction_amount

| 键值                 | 类型      | 注释           | 备注 |
| -------------------- | --------- | -------------- | ---- |
| account_name         | capi_name | 交易账户名     |      |
| current_limit_amount | uint64_t  | 期间内总的消耗 |      |



#### extracttb （提币记录）

| 键值             | 类型           | 注释       | 备注 |
| ---------------- | -------------- | ---------- | ---- |
| id               | uint128_t      | 订单号     |      |
| account_name     | capi_name      | 提币账户   |      |
| to_name          | capi_name      | 提币到账户 |      |
| ext_asset        | extended_asset | 提币金额   |      |
| status           | uint8_t        | 提币状态   |      |
| update_timepoint | time_point     | 更新时间   |      |

------