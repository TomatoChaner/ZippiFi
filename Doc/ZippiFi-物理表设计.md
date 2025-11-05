# ZippiFi 项目数据库物理表设计

## 1. 概述

本文档定义了ZippiFi项目的数据库物理表结构，包含用户管理、资产管理、交易、借贷、AI代理、质押、空投等核心功能模块的表设计。本文档基于项目需求分析，提供了详细的表结构设计、字段说明、索引设计和数据安全考虑。

## 2. 数据库表结构设计

### 2.1 用户相关表

#### 2.1.1 `users`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 用户ID |
| `wallet_address` | `VARCHAR(255)` | `UNIQUE NOT NULL` | 钱包地址 |
| `email` | `VARCHAR(255)` | `UNIQUE` | 邮箱 |
| `username` | `VARCHAR(100)` | `UNIQUE` | 用户名 |
| `avatar_url` | `VARCHAR(500)` | | 头像URL |
| `registration_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 注册时间 |
| `last_login_time` | `TIMESTAMP` | | 最后登录时间 |
| `status` | `TINYINT` | `DEFAULT 1` | 状态(1:活跃,0:禁用) |
| `kyc_verified` | `BOOLEAN` | `DEFAULT FALSE` | KYC验证状态 |

#### 2.1.2 `user_settings`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 设置ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `theme` | `VARCHAR(20)` | `DEFAULT 'light'` | 主题设置 |
| `language` | `VARCHAR(10)` | `DEFAULT 'zh_CN'` | 语言设置 |
| `notification_enabled` | `BOOLEAN` | `DEFAULT TRUE` | 是否开启通知 |
| `slippage_tolerance` | `DECIMAL(5,2)` | `DEFAULT 0.5` | 默认滑点容忍度 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

### 2.2 资产相关表

#### 2.2.1 `assets`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 资产ID |
| `symbol` | `VARCHAR(20)` | `UNIQUE NOT NULL` | 代币符号 |
| `name` | `VARCHAR(100)` | `NOT NULL` | 代币名称 |
| `contract_address` | `VARCHAR(255)` | `UNIQUE` | 合约地址 |
| `decimals` | `INT` | `NOT NULL DEFAULT 18` | 小数位数 |
| `logo_url` | `VARCHAR(500)` | | Logo URL |
| `chain_id` | `INT` | `NOT NULL` | 链ID |
| `is_native` | `BOOLEAN` | `DEFAULT FALSE` | 是否为原生代币 |
| `is_active` | `BOOLEAN` | `DEFAULT TRUE` | 是否启用 |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 创建时间 |

#### 2.2.2 `user_assets`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 记录ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 资产ID |
| `balance` | `DECIMAL(36,18)` | `DEFAULT 0` | 余额 |
| `locked_balance` | `DECIMAL(36,18)` | `DEFAULT 0` | 锁定余额 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

#### 2.2.3 `asset_prices`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 价格ID |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 资产ID |
| `price` | `DECIMAL(36,18)` | `NOT NULL` | 价格(USD) |
| `price_source` | `VARCHAR(50)` | `NOT NULL` | 价格来源 |
| `timestamp` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 时间戳 |

### 2.3 交易相关表

#### 2.3.1 `transactions`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 交易ID |
| `tx_hash` | `VARCHAR(255)` | `UNIQUE` | 链上交易哈希 |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `type` | `VARCHAR(20)` | `NOT NULL` | 交易类型(swap/liquidity/lend/stake/ai_agent) |
| `sub_type` | `VARCHAR(20)` | | 子类型(exact_input/supply/borrow等) |
| `status` | `VARCHAR(20)` | `NOT NULL` | 状态(pending/confirmed/failed) |
| `from_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 源资产ID |
| `to_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 目标资产ID |
| `from_amount` | `DECIMAL(36,18)` | `NOT NULL` | 源金额 |
| `to_amount` | `DECIMAL(36,18)` | | 目标金额 |
| `fee_amount` | `DECIMAL(36,18)` | `DEFAULT 0` | 手续费金额 |
| `gas_fee` | `DECIMAL(36,18)` | | Gas费用 |
| `block_number` | `BIGINT` | | 区块号 |
| `transaction_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 交易时间 |
| `confirmed_at` | `TIMESTAMP` | | 确认时间 |
| `chain_id` | `INT` | `NOT NULL` | 链ID |
| `source_address` | `VARCHAR(255)` | | 源地址 |
| `destination_address` | `VARCHAR(255)` | | 目标地址 |

#### 2.3.2 `swap_orders`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 订单ID |
| `transaction_id` | `BIGINT` | `FOREIGN KEY REFERENCES transactions(id)` | 交易ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `sell_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 卖出资产ID |
| `buy_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 买入资产ID |
| `sell_amount` | `DECIMAL(36,18)` | `NOT NULL` | 卖出金额 |
| `buy_amount` | `DECIMAL(36,18)` | | 买入金额 |
| `slippage` | `DECIMAL(5,2)` | `NOT NULL` | 滑点容忍度 |
| `deadline` | `TIMESTAMP` | `NOT NULL` | 截止时间 |
| `order_type` | `VARCHAR(20)` | `NOT NULL` | 订单类型(market/limit/twap) |
| `route` | `TEXT` | | 交易路径(JSON) |
| `price_impact` | `DECIMAL(10,6)` | | 价格影响 |
| `status` | `VARCHAR(20)` | `NOT NULL` | 状态(pending/filled/expired/canceled) |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 创建时间 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

#### 2.3.3 `liquidity_positions`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 头寸ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `pool_address` | `VARCHAR(255)` | `NOT NULL` | 池地址 |
| `asset0_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 资产0 ID |
| `asset1_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 资产1 ID |
| `liquidity` | `DECIMAL(36,18)` | `NOT NULL` | 流动性数量 |
| `asset0_amount` | `DECIMAL(36,18)` | `NOT NULL` | 资产0金额 |
| `asset1_amount` | `DECIMAL(36,18)` | `NOT NULL` | 资产1金额 |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 创建时间 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

### 2.4 借贷相关表

#### 2.4.1 `lending_markets`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 市场ID |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 资产ID |
| `symbol` | `VARCHAR(20)` | `NOT NULL` | 市场符号 |
| `total_supply` | `DECIMAL(36,18)` | `DEFAULT 0` | 总供应量 |
| `total_borrow` | `DECIMAL(36,18)` | `DEFAULT 0` | 总借款量 |
| `utilization_rate` | `DECIMAL(10,6)` | `DEFAULT 0` | 利用率 |
| `supply_apy` | `DECIMAL(10,6)` | `DEFAULT 0` | 存款年化收益率 |
| `borrow_apy` | `DECIMAL(10,6)` | `DEFAULT 0` | 借款年化收益率 |
| `liquidity_index` | `DECIMAL(36,18)` | `DEFAULT 1` | 流动性指数 |
| `borrow_index` | `DECIMAL(36,18)` | `DEFAULT 1` | 借款指数 |
| `ltv` | `DECIMAL(5,2)` | | 贷款价值比 |
| `liquidation_threshold` | `DECIMAL(5,2)` | | 清算阈值 |
| `liquidation_penalty` | `DECIMAL(5,2)` | | 清算罚金 |
| `is_active` | `BOOLEAN` | `DEFAULT TRUE` | 是否激活 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

#### 2.4.2 `user_lending_positions`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 头寸ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `market_id` | `BIGINT` | `FOREIGN KEY REFERENCES lending_markets(id)` | 市场ID |
| `supplied_amount` | `DECIMAL(36,18)` | `DEFAULT 0` | 存款金额 |
| `borrowed_amount` | `DECIMAL(36,18)` | `DEFAULT 0` | 借款金额 |
| `is_collateral` | `BOOLEAN` | `DEFAULT FALSE` | 是否用作抵押 |
| `borrow_rate_mode` | `VARCHAR(20)` | | 借款利率模式(stable/variable) |
| `supplied_liquidity_index` | `DECIMAL(36,18)` | `DEFAULT 1` | 存款时流动性指数 |
| `borrowed_borrow_index` | `DECIMAL(36,18)` | `DEFAULT 1` | 借款时借款指数 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

#### 2.4.3 `user_account_health`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 记录ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id) UNIQUE` | 用户ID |
| `health_factor` | `DECIMAL(36,18)` | `DEFAULT 0` | 健康因子 |
| `total_collateral_value` | `DECIMAL(36,18)` | `DEFAULT 0` | 总抵押价值(USD) |
| `total_borrow_value` | `DECIMAL(36,18)` | `DEFAULT 0` | 总借款价值(USD) |
| `is_liquidatable` | `BOOLEAN` | `DEFAULT FALSE` | 是否可清算 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

#### 2.4.4 `liquidations`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 清算ID |
| `tx_hash` | `VARCHAR(255)` | `UNIQUE` | 链上交易哈希 |
| `target_user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 被清算用户ID |
| `liquidator_user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 清算人用户ID |
| `debt_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 债务资产ID |
| `collateral_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 抵押资产ID |
| `repay_amount` | `DECIMAL(36,18)` | `NOT NULL` | 还款金额 |
| `seize_amount` | `DECIMAL(36,18)` | `NOT NULL` | 获得抵押金额 |
| `liquidation_penalty` | `DECIMAL(5,2)` | `NOT NULL` | 清算罚金 |
| `liquidation_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 清算时间 |
| `block_number` | `BIGINT` | | 区块号 |

### 2.5 AI代理相关表

#### 2.5.1 `ai_agent_configs`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 配置ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `name` | `VARCHAR(100)` | `NOT NULL` | 代理名称 |
| `strategy_type` | `VARCHAR(50)` | `NOT NULL` | 策略类型(arbitrage/yield_farming/hodl) |
| `risk_level` | `VARCHAR(20)` | `NOT NULL` | 风险等级(low/medium/high) |
| `asset_allocation` | `JSON` | | 资产配置(JSON) |
| `is_active` | `BOOLEAN` | `DEFAULT FALSE` | 是否激活 |
| `max_transaction_value` | `DECIMAL(36,18)` | | 最大交易价值 |
| `min_profit_threshold` | `DECIMAL(10,6)` | | 最小利润阈值 |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 创建时间 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

#### 2.5.2 `ai_agent_executions`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 执行ID |
| `agent_config_id` | `BIGINT` | `FOREIGN KEY REFERENCES ai_agent_configs(id)` | 代理配置ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `transaction_id` | `BIGINT` | `FOREIGN KEY REFERENCES transactions(id)` | 交易ID |
| `execution_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 执行时间 |
| `status` | `VARCHAR(20)` | `NOT NULL` | 状态(pending/success/failed) |
| `reason` | `TEXT` | | 执行原因/失败原因 |
| `profit` | `DECIMAL(36,18)` | `DEFAULT 0` | 利润 |
| `execution_details` | `JSON` | | 执行详情(JSON) |

#### 2.5.3 `ai_agent_audit_logs`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 日志ID |
| `agent_config_id` | `BIGINT` | `FOREIGN KEY REFERENCES ai_agent_configs(id)` | 代理配置ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `action_type` | `VARCHAR(50)` | `NOT NULL` | 操作类型 |
| `action_details` | `JSON` | | 操作详情(JSON) |
| `timestamp` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 时间戳 |
| `actor` | `VARCHAR(100)` | | 操作者(ai/user/system) |

#### 2.5.4 `ai_agent_performance`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 性能ID |
| `agent_config_id` | `BIGINT` | `FOREIGN KEY REFERENCES ai_agent_configs(id)` | 代理配置ID |
| `total_profit` | `DECIMAL(36,18)` | `DEFAULT 0` | 总利润 |
| `total_loss` | `DECIMAL(36,18)` | `DEFAULT 0` | 总亏损 |
| `total_trades` | `INT` | `DEFAULT 0` | 总交易次数 |
| `win_rate` | `DECIMAL(10,6)` | `DEFAULT 0` | 胜率 |
| `sharpe_ratio` | `DECIMAL(10,6)` | | 夏普比率 |
| `last_updated` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 最后更新时间 |

### 2.6 质押相关表

#### 2.6.1 `staking_pools`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 质押池ID |
| `name` | `VARCHAR(100)` | `NOT NULL` | 池子名称 |
| `stake_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 质押资产ID |
| `reward_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 奖励资产ID |
| `total_staked` | `DECIMAL(36,18)` | `DEFAULT 0` | 总质押量 |
| `apr` | `DECIMAL(10,6)` | `DEFAULT 0` | 年化收益率 |
| `start_time` | `TIMESTAMP` | | 开始时间 |
| `end_time` | `TIMESTAMP` | | 结束时间 |
| `is_active` | `BOOLEAN` | `DEFAULT TRUE` | 是否激活 |

#### 2.6.2 `user_staking_positions`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 头寸ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `pool_id` | `BIGINT` | `FOREIGN KEY REFERENCES staking_pools(id)` | 质押池ID |
| `staked_amount` | `DECIMAL(36,18)` | `NOT NULL` | 质押金额 |
| `reward_debt` | `DECIMAL(36,18)` | `DEFAULT 0` | 奖励债务 |
| `last_update_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 最后更新时间 |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 创建时间 |

### 2.7 空投相关表

#### 2.7.1 `airdrop_campaigns`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 活动ID |
| `name` | `VARCHAR(100)` | `NOT NULL` | 活动名称 |
| `description` | `TEXT` | | 活动描述 |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 空投资产ID |
| `total_amount` | `DECIMAL(36,18)` | `NOT NULL` | 总空投量 |
| `start_time` | `TIMESTAMP` | | 开始时间 |
| `end_time` | `TIMESTAMP` | | 结束时间 |
| `status` | `VARCHAR(20)` | `DEFAULT 'planned'` | 状态(planned/active/ended/completed) |

#### 2.7.2 `user_airdrop_claims`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 领取ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `campaign_id` | `BIGINT` | `FOREIGN KEY REFERENCES airdrop_campaigns(id)` | 活动ID |
| `claim_amount` | `DECIMAL(36,18)` | `NOT NULL` | 可领取金额 |
| `claimed_amount` | `DECIMAL(36,18)` | `DEFAULT 0` | 已领取金额 |
| `claim_status` | `VARCHAR(20)` | `DEFAULT 'eligible'` | 状态(eligible/claimed/expired) |
| `claim_time` | `TIMESTAMP` | | 领取时间 |
| `tx_hash` | `VARCHAR(255)` | | 交易哈希 |

### 2.8 通知相关表

#### 2.8.1 `notifications`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 通知ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `type` | `VARCHAR(50)` | `NOT NULL` | 通知类型 |
| `title` | `VARCHAR(200)` | `NOT NULL` | 标题 |
| `content` | `TEXT` | `NOT NULL` | 内容 |
| `is_read` | `BOOLEAN` | `DEFAULT FALSE` | 是否已读 |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 创建时间 |
| `related_id` | `BIGINT` | | 相关ID(交易ID/代理ID等) |
| `related_type` | `VARCHAR(50)` | | 相关类型 |
| `priority` | `TINYINT` | `DEFAULT 3` | 优先级(1-高, 2-中, 3-低) |

### 2.9 风险控制相关表

#### 2.9.1 `risk_parameters`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 参数ID |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id) UNIQUE` | 资产ID |
| `max_daily_withdrawal` | `DECIMAL(36,18)` | | 最大日提款限额 |
| `max_transaction_amount` | `DECIMAL(36,18)` | | 最大单笔交易金额 |
| `min_collateral_ratio` | `DECIMAL(10,6)` | | 最小抵押率 |
| `circuit_breaker_threshold` | `DECIMAL(10,6)` | | 熔断阈值百分比 |
| `is_enabled` | `BOOLEAN` | `DEFAULT TRUE` | 是否启用 |
| `updated_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 更新时间 |

#### 2.9.2 `circuit_breakers`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 熔断ID |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 资产ID |
| `triggered_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 触发时间 |
| `reason` | `VARCHAR(255)` | `NOT NULL` | 触发原因 |
| `price_before` | `DECIMAL(36,18)` | | 触发前价格 |
| `price_after` | `DECIMAL(36,18)` | | 触发后价格 |
| `duration_minutes` | `INT` | `NOT NULL` | 熔断持续时间(分钟) |
| `status` | `VARCHAR(20)` | `DEFAULT 'active'` | 状态(active/expired) |

### 2.10 预言机相关表

#### 2.10.1 `oracle_data`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 数据ID |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 资产ID |
| `price` | `DECIMAL(36,18)` | `NOT NULL` | 预言机价格 |
| `timestamp` | `TIMESTAMP` | `NOT NULL` | 预言机时间戳 |
| `source` | `VARCHAR(50)` | `NOT NULL` | 预言机来源 |
| `is_reliable` | `BOOLEAN` | `DEFAULT TRUE` | 是否可靠 |
| `confidence_interval` | `DECIMAL(10,6)` | | 置信区间 |

### 2.11 闪电贷相关表

#### 2.11.1 `flash_loans`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 闪电贷ID |
| `tx_hash` | `VARCHAR(255)` | `UNIQUE` | 链上交易哈希 |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 借款资产ID |
| `amount` | `DECIMAL(36,18)` | `NOT NULL` | 借款金额 |
| `fee` | `DECIMAL(36,18)` | `NOT NULL` | 手续费 |
| `target_contract` | `VARCHAR(255)` | `NOT NULL` | 目标合约地址 |
| `status` | `VARCHAR(20)` | `NOT NULL` | 状态(success/failed) |
| `execution_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 执行时间 |
| `block_number` | `BIGINT` | | 区块号 |

### 2.12 流动性挖矿相关表

#### 2.12.1 `liquidity_mining_pools`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 矿池ID |
| `name` | `VARCHAR(100)` | `NOT NULL` | 矿池名称 |
| `description` | `TEXT` | | 矿池描述 |
| `pool_address` | `VARCHAR(255)` | `NOT NULL` | 池合约地址 |
| `reward_asset_id` | `BIGINT` | `FOREIGN KEY REFERENCES assets(id)` | 奖励资产ID |
| `reward_rate` | `DECIMAL(36,18)` | `NOT NULL` | 奖励速率(每秒) |
| `start_time` | `TIMESTAMP` | `NOT NULL` | 开始时间 |
| `end_time` | `TIMESTAMP` | | 结束时间 |
| `total_rewards_distributed` | `DECIMAL(36,18)` | `DEFAULT 0` | 已分配奖励总量 |
| `is_active` | `BOOLEAN` | `DEFAULT TRUE` | 是否激活 |

#### 2.12.2 `user_liquidity_mining_positions`表

| 字段名 | 数据类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PRIMARY KEY AUTO_INCREMENT` | 头寸ID |
| `user_id` | `BIGINT` | `FOREIGN KEY REFERENCES users(id)` | 用户ID |
| `mining_pool_id` | `BIGINT` | `FOREIGN KEY REFERENCES liquidity_mining_pools(id)` | 矿池ID |
| `staked_amount` | `DECIMAL(36,18)` | `DEFAULT 0` | 质押金额 |
| `reward_debt` | `DECIMAL(36,18)` | `DEFAULT 0` | 奖励债务 |
| `pending_reward` | `DECIMAL(36,18)` | `DEFAULT 0` | 待领取奖励 |
| `last_update_time` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 最后更新时间 |
| `created_at` | `TIMESTAMP` | `DEFAULT CURRENT_TIMESTAMP` | 创建时间 |

## 3. 索引设计

为了提高查询性能，建议在以下字段上建立索引：

- `users` 表：`email` (唯一索引)、`wallet_address` (唯一索引)、`created_at`
- `user_assets` 表：`user_id`、`asset_id`、复合索引 `(user_id, asset_id)`
- `assets` 表：`symbol` (唯一索引)、`contract_address` (唯一索引)
- `transactions` 表：`user_id`、`tx_hash` (唯一索引)、`created_at`、`status`
- `swap_orders` 表：`user_id`、`order_id` (唯一索引)、`status`
- `liquidity_positions` 表：`user_id`、`pool_address`
- `lending_markets` 表：`asset_id` (唯一索引)
- `user_lending_positions` 表：`user_id`、`lending_market_id`、复合索引 `(user_id, lending_market_id)`
- `ai_agent_configs` 表：`user_id`、`agent_id` (唯一索引)、`status`
- `ai_agent_executions` 表：`agent_id`、`created_at`
- `ai_agent_audit_logs` 表：`agent_id`、`created_at`
- `staking_pools` 表：`pool_address` (唯一索引)
- `user_staking_positions` 表：`user_id`、`staking_pool_id`、复合索引 `(user_id, staking_pool_id)`
- `airdrop_campaigns` 表：`campaign_id` (唯一索引)
- `user_airdrop_claims` 表：`user_id`、`airdrop_campaign_id`、复合索引 `(user_id, airdrop_campaign_id)`
- `notifications` 表：`user_id`、`is_read`、`created_at`、`priority`
- `risk_parameters` 表：`asset_id` (唯一索引)
- `circuit_breakers` 表：`asset_id`、`status`、`triggered_at`
- `oracle_data` 表：`asset_id`、`timestamp`、复合索引 `(asset_id, timestamp)`
- `flash_loans` 表：`tx_hash` (唯一索引)、`user_id`、`status`、`execution_time`
- `liquidity_mining_pools` 表：`pool_address` (唯一索引)、`is_active`
- `user_liquidity_mining_positions` 表：`user_id`、`mining_pool_id`、复合索引 `(user_id, mining_pool_id)`

## 4. 数据安全考虑

### 4.1 用户隐私保护

- 敏感信息（如密码）应使用bcrypt或Argon2等安全哈希算法进行加密存储
- 用户钱包地址等敏感信息考虑进行脱敏处理
- 实施基于角色的访问控制(RBAC)，严格限制数据访问权限
- 个人身份信息(PII)应遵循GDPR等隐私法规要求

### 4.2 交易数据安全

- 交易数据应进行完整性校验，可使用哈希链或Merkle树验证
- 敏感的交易操作需要多重签名确认
- 定期备份交易数据，并实施异地备份策略
- 对所有交易数据进行不可篡改记录，必要时使用区块链存证

### 4.3 系统安全

- 实施数据库层面的最小权限原则，严格控制访问权限
- 使用参数化查询防止SQL注入攻击
- 定期进行安全审计和渗透测试
- 实施数据库连接池加密和传输层安全(TLS)
- 对高价值资产操作实施时间锁定和限额控制

### 4.4 风险控制安全

- 熔断机制和风险参数应实时监控并记录变更日志
- 预言机数据应进行多源验证，防止单点故障
- 大额交易应进行实时风险评估和预警
- 异常交易模式应进行自动识别和阻断

## 5. 扩展性考虑

### 5.1 水平扩展

- 考虑分库分表策略，特别是对交易数据、AI代理执行记录等大数据量的表
- 使用读写分离架构提高查询性能，主库处理写操作，从库处理读操作
- 实施数据库分片，根据用户ID或时间范围进行数据分区
- 考虑引入NoSQL数据库作为补充，处理非结构化或半结构化数据

### 5.2 垂直扩展

- 考虑将热点数据与冷数据分离存储，对历史数据进行归档处理
- 使用缓存机制（如Redis）加速热点数据访问
- 对预言机数据等高频更新数据采用时序数据库存储
- 实施数据生命周期管理，自动归档和清理过期数据

### 5.3 接口扩展性

- 设计灵活的API接口，支持未来功能扩展
- 考虑使用微服务架构，实现模块解耦和独立扩展
- 实施事件驱动架构，通过消息队列处理异步任务
- 提供API版本控制机制，确保向后兼容性

### 5.4 功能扩展性

- 预留字段和扩展点，支持未来新功能的快速集成
- 设计模块化的数据模型，便于添加新的资产类型和交易类型
- 支持多链部署，为不同区块链提供统一的数据访问接口
- 建立可配置的规则引擎，支持灵活的业务规则调整

## 6. 总结

本文档定义了ZippiFi项目的核心数据库表结构，涵盖了用户、资产、交易、借贷、AI代理、质押和空投等功能模块。通过合理的表设计和索引优化，可以满足项目的性能和扩展性需求。