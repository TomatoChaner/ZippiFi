# ZippiFi 功能清单

## 1. 用户管理模块

### 1.1 用户认证与授权
**详细说明**：提供用户注册、登录、认证和授权管理功能，支持基于钱包地址的身份验证。

**接口设计**：
- `POST /api/auth/register` - 用户注册
  - 入参：`{"wallet_address": "0x...", "email": "...", "username": "..."}`
  - 出参：`{"user_id": "...", "status": "created", "created_at": "..."}`

- `POST /api/auth/login` - 用户登录
  - 入参：`{"wallet_address": "0x...", "signature": "..."}`
  - 出参：`{"access_token": "...", "refresh_token": "...", "user": {...}}`

- `GET /api/users/profile` - 获取用户资料
  - 入参：N/A
  - 出参：`{"user_id": "...", "wallet_address": "...", "username": "...", "email": "...", "created_at": "..."}`

### 1.2 用户权限管理
**详细说明**：管理用户的访问权限和操作权限，支持角色分配和权限控制。

**接口设计**：
- `GET /api/users/me/permissions` - 获取当前用户权限
  - 入参：N/A
  - 出参：`{"permissions": ["trade", "withdraw", "create_strategy"], "roles": ["user", "premium"]}`

- `PUT /api/users/permissions/{user_id}` - 更新用户权限
  - 入参：`{"roles": ["admin", "trader"]}`
  - 出参：`{"status": "updated", "updated_at": "..."}`

## 2. AI代理核心模块

### 2.1 策略配置管理
**详细说明**：提供AI交易策略的创建、配置、更新和回测功能，支持多种风险等级和资产配置模式。

**接口设计**：
- `GET /api/ai-agent/strategy-templates` - 获取策略模板列表
  - 入参：N/A
  - 出参：`{"templates": [{"template_id": "conservative", "name": "保守型策略", "risk_level": "low", "default_allocation": {...}}]}`

- `POST /api/ai-agent/strategies` - 创建自定义策略
  - 入参：`{"strategy_name": "我的策略", "risk_level": "medium", "asset_allocation": {...}, "trading_params": {...}}`
  - 出参：`{"strategy_id": "str_123456", "status": "created", "created_at": "..."}`

- `PUT /api/ai-agent/strategies/{strategy_id}` - 更新策略配置
  - 入参：`{"asset_allocation": {...}, "trading_params": {...}}`
  - 出参：`{"strategy_id": "str_123456", "status": "updated", "updated_at": "..."}`

- `POST /api/ai-agent/strategies/{strategy_id}/backtest` - 策略回测
  - 入参：`{"time_period": "6months", "initial_capital": 100000, "include_fees": true}`
  - 出参：`{"backtest_id": "bt_789012", "status": "completed", "performance": {...}, "equity_curve": [...]}`

### 2.2 AI代理管理
**详细说明**：管理AI代理实例的生命周期，包括启动、暂停、重启和状态监控。

**接口设计**：
- `POST /api/ai-agent/instances/start` - 启动AI代理
  - 入参：`{"strategy_id": "str_123456", "initial_funding": 50000, "authorized_assets": [...], "auto_rebalance": true}`
  - 出参：`{"agent_id": "ai_987654", "status": "running", "started_at": "...", "funds_allocated": 50000}`

- `POST /api/ai-agent/instances/{agent_id}/pause` - 暂停AI代理
  - 入参：N/A
  - 出参：`{"agent_id": "ai_987654", "status": "paused", "paused_at": "..."}`

- `POST /api/ai-agent/instances/{agent_id}/resume` - 重启AI代理
  - 入参：N/A
  - 出参：`{"agent_id": "ai_987654", "status": "running", "resumed_at": "..."}`

- `GET /api/ai-agent/instances/{agent_id}/status` - 获取AI代理状态
  - 入参：N/A
  - 出参：`{"agent_id": "ai_987654", "status": "running", "current_aum": 52500, "total_pnl": 2500, "performance_metrics": {...}}`

- `PUT /api/ai-agent/instances/{agent_id}/strategy` - 调整运行中的代理策略
  - 入参：`{"asset_allocation": {...}, "risk_parameters": {...}}`
  - 出参：`{"agent_id": "ai_987654", "status": "strategy_updated", "update_scheduled": true, "next_rebalance_at": "..."}`

### 2.3 交易执行管理
**详细说明**：管理AI代理的交易执行过程，包括市场数据获取、交易历史记录、持仓管理等。

**接口设计**：
- `GET /api/ai-agent/market-data` - 获取市场数据
  - 入参：`?assets=BTC,ETH,USDC&timeframe=1h&limit=24`
  - 出参：`{"data": {"BTC": [{"timestamp": "...", "price": 35000, "volume": 1200}], ...}, "last_updated": "..."}`

- `GET /api/ai-agent/instances/{agent_id}/transactions` - 获取交易历史
  - 入参：`?start_date=2023-06-01&end_date=2023-06-16&transaction_type=buy,sell`
  - 出参：`{"transactions": [{"tx_id": "tx_123456", "timestamp": "...", "type": "buy", "asset": "ETH", "amount": 0.5, ...}], "total_count": 42}`

- `GET /api/ai-agent/instances/{agent_id}/portfolio` - 获取当前持仓
  - 入参：N/A
  - 出参：`{"total_value": 52500, "holdings": [{"asset": "BTC", "amount": 0.85, "value": 29750, "allocation_percentage": 56.67}], "last_updated": "..."}`

- `POST /api/ai-agent/trade` - 手动执行交易
  - 入参：`{"agent_id": "ai_987654", "type": "buy", "asset": "ETH", "amount": 0.5, "slippage_tolerance": 0.5}`
  - 出参：`{"order_id": "ord_456789", "status": "pending", "requested_amount": 0.5, "estimated_price": 3200}`

### 2.4 结算与审计
**详细说明**：管理收益结算、绩效报告生成、风险评估和交易数据导出功能。

**接口设计**：
- `POST /api/ai-agent/revenue-rules` - 设置收益分配规则
  - 入参：`{"base_split_ratio": {"user": 85, "platform": 15}, "tiered_split_rules": [...], "settlement_frequency": "daily"}`
  - 出参：`{"rule_id": "rr_345678", "status": "configured", "created_at": "..."}`

- `GET /api/ai-agent/instances/{agent_id}/settlements` - 获取结算历史
  - 入参：`?start_date=2023-06-01&end_date=2023-06-16&settlement_type=daily`
  - 出参：`{"settlements": [{"settlement_id": "st_654321", "period": "2023-06-15", "total_revenue": 120, "user_share": 102, ...}], "total_count": 15}`

- `POST /api/ai-agent/instances/{agent_id}/generate-report` - 生成绩效报告
  - 入参：`{"report_type": "performance", "time_period": "monthly", "start_date": "2023-06-01", "end_date": "2023-06-30"}`
  - 出参：`{"report_id": "rpt_789012", "status": "generated", "report_url": "...", "metrics": {...}}`

- `GET /api/ai-agent/instances/{agent_id}/risk-assessment` - 获取风险评估
  - 入参：N/A
  - 出参：`{"agent_id": "ai_987654", "overall_risk_score": 65, "risk_factors": [...], "liquidity_score": 85}`

- `GET /api/ai-agent/instances/{agent_id}/export` - 导出交易数据
  - 入参：`?format=csv&start_date=2023-06-01&end_date=2023-06-16&data_type=transactions`
  - 出参：`{"export_id": "exp_345678", "download_url": "...", "file_size": "1.2MB", "expires_at": "..."}`

- `GET /api/ai-agent/verify-transaction/{transaction_hash}` - 区块链交易验证
  - 入参：N/A
  - 出参：`{"transaction_hash": "0xabcdef...", "blockchain": "ethereum", "status": "confirmed", "details": {...}}`

- `POST /api/ai-agent/instances/{agent_id}/withdraw` - 提取收益
  - 入参：`{"amount": 1000, "target_asset": "USDC", "destination_address": "0x..."}`
  - 出参：`{"withdrawal_id": "wd_234567", "status": "processing", "requested_amount": 1000, "estimated_fee": 5}`

## 3. 交易模块

### 3.1 Swap交易
**详细说明**：提供代币兑换功能，支持同链原子结算和跨链多阶段流程，兼容收税代币与多跳路径。

**接口设计**：
- `POST /api/swap/quote` - 获取兑换报价
  - 入参：`{"from_chain": "ethereum", "to_chain": "ethereum", "from_token": "USDC", "to_token": "ETH", "amount": 1000, "slippage": 0.5}`
  - 出参：`{"quote_id": "q_123456", "estimated_amount_out": 0.31, "min_amount_out": 0.308, "path": ["USDC", "ETH"], "fee": 2}`

- `POST /api/swap/execute` - 执行兑换
  - 入参：`{"quote_id": "q_123456", "wallet_address": "0x...", "deadline": 10}`
  - 出参：`{"transaction_hash": "0x...", "status": "pending", "execution_id": "ex_123456"}`

### 3.2 限价单（Limit）
**详细说明**：支持设置目标价格，达到价格条件后自动执行交易。

**接口设计**：
- `POST /api/orders/limit` - 创建限价单
  - 入参：`{"from_token": "USDC", "to_token": "ETH", "amount": 1000, "target_price": 3200, "expiry": 86400}`
  - 出参：`{"order_id": "limit_123456", "status": "pending", "created_at": "..."}`

- `GET /api/orders/limit` - 获取限价单列表
  - 入参：`?status=pending,completed,cancelled`
  - 出参：`{"orders": [{"order_id": "limit_123456", "status": "pending", "target_price": 3200, ...}], "total_count": 5}`

### 3.3 TWAP交易
**详细说明**：时间加权平均价格交易，将大额兑换拆分为多批执行，降低价格冲击。

**接口设计**：
- `POST /api/orders/twap` - 创建TWAP订单
  - 入参：`{"from_token": "USDC", "to_token": "ETH", "total_amount": 10000, "parts": 10, "interval": 3600}`
  - 出参：`{"twap_id": "twap_123456", "status": "active", "created_at": "...", "executed_parts": 0, "total_parts": 10}`

### 3.4 法币兑换
**详细说明**：支持加密货币与法币之间的兑换功能。

**接口设计**：
- `POST /api/fiat/quote` - 获取法币兑换报价
  - 入参：`{"crypto": "BTC", "fiat": "USD", "amount": 1, "direction": "crypto_to_fiat"}`
  - 出参：`{"quote_id": "fiat_123456", "rate": 35000, "total_fiat_amount": 35000, "fee": 175}`

## 4. 流动性模块

### 4.1 AMM流动性管理
**详细说明**：提供流动性添加、移除和收益管理功能。

**接口设计**：
- `POST /api/liquidity/add` - 添加流动性
  - 入参：`{"token_a": "USDC", "token_b": "ETH", "amount_a": 1000, "amount_b": 0.31, "slippage": 0.5}`
  - 出参：`{"position_id": "lp_123456", "liquidity_tokens": 45.2, "transaction_hash": "0x..."}`

- `POST /api/liquidity/remove` - 移除流动性
  - 入参：`{"position_id": "lp_123456", "percentage": 100}`
  - 出参：`{"status": "processing", "transaction_hash": "0x...", "estimated_tokens": {"USDC": 1020, "ETH": 0.32}}`

- `GET /api/liquidity/positions` - 获取流动性头寸
  - 入参：N/A
  - 出参：`{"positions": [{"position_id": "lp_123456", "tokens": ["USDC", "ETH"], "liquidity_tokens": 45.2, "value_usd": 2050}], "total_value_usd": 2050}`

### 4.2 流动性收益分析
**详细说明**：提供流动性挖矿收益分析和历史数据。

**接口设计**：
- `GET /api/liquidity/yields/{position_id}` - 获取头寸收益
  - 入参：N/A
  - 出参：`{"position_id": "lp_123456", "total_fees_earned": 25.6, "apr": 45.2, "daily_yield": 0.85}`

## 5. 借贷模块

### 5.1 借贷市场管理
**详细说明**：管理借贷市场信息，包括利率、可用资产、抵押率等。

**接口设计**：
- `GET /api/lend/markets` - 获取借贷市场列表
  - 入参：`?chain=ethereum`
  - 出参：`{"markets": [{"asset": "USDC", "supply_apy": 4.5, "borrow_apy": 6.2, "total_supply": 1000000, "total_borrow": 800000}]}`

### 5.2 存款与借款
**详细说明**：提供资产存款和借款功能，支持抵押借款和信用借款。

**接口设计**：
- `POST /api/lend/supply` - 存款
  - 入参：`{"asset": "USDC", "amount": 1000}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "supplied_amount": 1000}`

- `POST /api/lend/borrow` - 借款
  - 入参：`{"asset": "USDC", "amount": 500, "collateral_asset": "ETH"}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "borrowed_amount": 500, "health_factor": 2.5}`

- `POST /api/lend/repay` - 还款
  - 入参：`{"asset": "USDC", "amount": 250}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "repaid_amount": 250, "remaining_debt": 250}`

- `POST /api/lend/withdraw` - 提款
  - 入参：`{"asset": "USDC", "amount": 500}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "withdrawn_amount": 500}`

### 5.3 借贷头寸管理
**详细说明**：管理用户的借贷头寸，包括抵押率监控、清算风险评估等。

**接口设计**：
- `GET /api/lend/positions` - 获取借贷头寸
  - 入参：N/A
  - 出参：`{"supplied": [{"asset": "USDC", "amount": 1000, "value_usd": 1000, "earned_interest": 45}], "borrowed": [{"asset": "ETH", "amount": 0.05, "value_usd": 160, "accrued_interest": 5}], "health_factor": 2.8}`

- `GET /api/lend/health` - 获取健康因子
  - 入参：N/A
  - 出参：`{"health_factor": 2.8, "liquidation_threshold": 1.5, "collateral_ratio": 6.25, "risk_level": "safe"}`

## 6. 质押模块

### 6.1 质押池管理
**详细说明**：管理各种质押池的信息，包括APY、锁定期、质押资产等。

**接口设计**：
- `GET /api/stake/pools` - 获取质押池列表
  - 入参：`?chain=ethereum`
  - 出参：`{"pools": [{"pool_id": "pool_123", "asset": "ZIP", "apy": 85, "total_staked": 1000000, "lock_period_days": 30}]}`

### 6.2 质押操作
**详细说明**：提供质押、解除质押和领取奖励功能。

**接口设计**：
- `POST /api/stake/deposit` - 质押资产
  - 入参：`{"pool_id": "pool_123", "amount": 1000}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "staked_amount": 1000, "start_date": "..."}`

- `POST /api/stake/withdraw` - 解除质押
  - 入参：`{"pool_id": "pool_123", "amount": 500}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "withdrawn_amount": 500, "penalty": 0}`

- `POST /api/stake/claim` - 领取奖励
  - 入参：`{"pool_id": "pool_123"}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "claimed_amount": 125, "pending_rewards": 0}`

### 6.3 质押状态查询
**详细说明**：查询质押状态、奖励和锁定期信息。

**接口设计**：
- `GET /api/stake/positions` - 获取质押头寸
  - 入参：N/A
  - 出参：`{"positions": [{"pool_id": "pool_123", "staked_amount": 1000, "earned_rewards": 125, "lock_end_date": "...", "apy": 85}]}`

## 7. 空投模块

### 7.1 空投活动管理
**详细说明**：管理空投活动信息，包括资格、奖励和领取状态。

**接口设计**：
- `GET /api/airdrop/campaigns` - 获取空投活动列表
  - 入参：N/A
  - 出参：`{"campaigns": [{"campaign_id": "air_123", "name": "创世空投", "status": "active", "total_rewards": 1000000, "end_date": "..."}]}`

- `GET /api/airdrop/eligibility` - 查询空投资格
  - 入参：`?campaign_id=air_123`
  - 出参：`{"campaign_id": "air_123", "eligible": true, "reward_amount": 1000, "claim_status": "available"}`

### 7.2 空投领取
**详细说明**：提供空投奖励领取功能。

**接口设计**：
- `POST /api/airdrop/claim` - 领取空投
  - 入参：`{"campaign_id": "air_123"}`
  - 出参：`{"transaction_hash": "0x...", "status": "processing", "claimed_amount": 1000}`

### 7.3 防女巫机制
**详细说明**：实现防女巫攻击机制，包括地址验证、行为分析等。

**接口设计**：
- `GET /api/airdrop/user-score` - 获取用户评分
  - 入参：N/A
  - 出参：`{"user_score": 85, "risk_level": "low", "verification_status": "verified", "factors": [...]}`

- `POST /api/airdrop/verify` - 验证用户身份
  - 入参：`{"verification_method": "sms", "verification_code": "123456"}`
  - 出参：`{"status": "verified", "verification_level": "medium", "score_boost": 15}`

## 8. 基础设施模块

### 8.1 市场数据服务
**详细说明**：提供实时市场数据、K线图数据和交易深度信息。

**接口设计**：
- `GET /api/market/tickers` - 获取市场行情
  - 入参：`?symbols=BTC/USDC,ETH/USDC&chain=ethereum`
  - 出参：`{"tickers": [{"symbol": "BTC/USDC", "price": 35000, "24h_change": 2.5, "volume": 12000000}]}`

- `GET /api/market/klines` - 获取K线数据
  - 入参：`?symbol=BTC/USDC&interval=1h&limit=100`
  - 出参：`{"symbol": "BTC/USDC", "interval": "1h", "data": [[1623456789, 34900, 35100, 34800, 35000, 1200]]}`

### 8.2 区块链适配器
**详细说明**：提供多链支持，标准化区块链交互接口。

**接口设计**：
- `GET /api/chains` - 获取支持的区块链列表
  - 入参：N/A
  - 出参：`{"chains": [{"id": "ethereum", "name": "Ethereum", "rpc_url": "...", "native_token": "ETH", "explorer": "..."}]}`

- `GET /api/chains/{chain_id}/tokens` - 获取链上代币列表
  - 入参：`?limit=100`
  - 出参：`{"tokens": [{"address": "0x...", "symbol": "USDC", "decimals": 6, "name": "USD Coin"}]}`

### 8.3 健康检查
**详细说明**：提供系统健康状态检查和监控功能。

**接口设计**：
- `GET /api/health` - 系统健康检查
  - 入参：N/A
  - 出参：`{"status": "healthy", "version": "1.0.0", "uptime": "86400s", "services": {"database": "ok", "blockchain": "ok", "api": "ok"}}`

- `GET /api/health/metrics` - 获取系统指标
  - 入参：N/A
  - 出参：`{"cpu_usage": 0.35, "memory_usage": 0.42, "active_connections": 150, "request_rate": 50}`