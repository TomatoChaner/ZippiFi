# ZippiFi 接口设计文档

## 目录
- [1. AI代理核心接口](#1-ai代理核心接口)
  - [1.1 策略配置接口](#11-策略配置接口)
  - [1.2 AI代理管理接口](#12-ai代理管理接口)
  - [1.3 交易执行接口](#13-交易执行接口)
  - [1.4 结算与审计接口](#14-结算与审计接口)
- [2. 用户管理模块接口](#2-用户管理模块接口)
- [3. 交易和资产管理模块接口](#3-交易和资产管理模块接口)
- [4. 借贷、质押和空投模块接口](#4-借贷质押和空投模块接口)

## 1. AI代理核心接口

### 1.1 策略配置接口

#### 1.1.1 获取策略模板列表
```
GET /api/ai-agent/strategy-templates
响应格式：
{
  "templates": [
    {
      "template_id": "conservative",
      "name": "保守型策略",
      "description": "低风险策略，优先考虑资产安全性",
      "risk_level": "low",
      "default_allocation": {
        "BTC": 30,
        "ETH": 20,
        "stablecoins": 40,
        "altcoins": 10
      },
      "default_params": {
        "max_trade_amount": 5000,
        "stop_loss_ratio": 5,
        "target_roi": 10
      }
    },
    // 其他策略模板...
  ]
}
```

#### 1.1.2 创建自定义策略
```
POST /api/ai-agent/strategies
请求体：
{
  "strategy_name": "我的平衡策略",
  "risk_level": "medium",
  "asset_allocation": {
    "BTC": 30,
    "ETH": 40,
    "stablecoins": 20,
    "altcoins": 10
  },
  "trading_params": {
    "max_trade_amount": 10000,
    "stop_loss_ratio": 10,
    "target_roi": 20,
    "rebalance_frequency": "daily"
  },
  "is_default": true
}
响应格式：
{
  "strategy_id": "str_123456",
  "status": "created",
  "created_at": "2023-06-15T10:30:00Z"
}
```

#### 1.1.3 更新策略配置
```
PUT /api/ai-agent/strategies/{strategy_id}
请求体：
{
  "asset_allocation": {
    "BTC": 25,
    "ETH": 45,
    "stablecoins": 20,
    "altcoins": 10
  },
  "trading_params": {
    "max_trade_amount": 12000,
    "stop_loss_ratio": 8
  }
}
响应格式：
{
  "strategy_id": "str_123456",
  "status": "updated",
  "updated_at": "2023-06-15T11:45:00Z"
}
```

#### 1.1.4 策略回测
```
POST /api/ai-agent/strategies/{strategy_id}/backtest
请求体：
{
  "time_period": "6months",
  "initial_capital": 100000,
  "include_fees": true
}
响应格式：
{
  "backtest_id": "bt_789012",
  "status": "completed",
  "performance": {
    "total_return": 15.2,
    "annualized_roi": 32.5,
    "sharpe_ratio": 1.8,
    "max_drawdown": 8.3,
    "win_rate": 65.4
  },
  "equity_curve": [
    {"date": "2023-01-01", "value": 100000},
    // 更多数据点...
  ],
  "trade_count": 42
}
```

#### 1.1.5 设置收益分配规则
```
POST /api/ai-agent/revenue-rules
请求体：
{
  "base_split_ratio": {
    "user": 85,
    "platform": 15
  },
  "tiered_split_rules": [
    {
      "threshold_roi": 20,
      "ratios": {
        "user": 80,
        "platform": 20
      }
    },
    {
      "threshold_roi": 50,
      "ratios": {
        "user": 75,
        "platform": 25
      }
    }
  ],
  "referral_bonus": 5,
  "settlement_frequency": "daily"
}
响应格式：
{
  "rule_id": "rr_345678",
  "status": "configured",
  "created_at": "2023-06-15T14:20:00Z"
}
```

### 1.2 AI代理管理接口

#### 1.2.1 启动AI代理
```
POST /api/ai-agent/instances/start
请求体：
{
  "strategy_id": "str_123456",
  "initial_funding": 50000,
  "authorized_assets": ["BTC", "ETH", "USDC", "AVAX"],
  "auto_rebalance": true
}
响应格式：
{
  "agent_id": "ai_987654",
  "status": "running",
  "started_at": "2023-06-15T15:30:00Z",
  "funds_allocated": 50000
}
```

#### 1.2.2 暂停AI代理
```
POST /api/ai-agent/instances/{agent_id}/pause
响应格式：
{
  "agent_id": "ai_987654",
  "status": "paused",
  "paused_at": "2023-06-16T09:45:00Z"
}
```

#### 1.2.3 重启AI代理
```
POST /api/ai-agent/instances/{agent_id}/resume
响应格式：
{
  "agent_id": "ai_987654",
  "status": "running",
  "resumed_at": "2023-06-16T11:20:00Z"
}
```

#### 1.2.4 获取AI代理状态
```
GET /api/ai-agent/instances/{agent_id}/status
响应格式：
{
  "agent_id": "ai_987654",
  "status": "running",
  "strategy_id": "str_123456",
  "current_aum": 52500,
  "total_pnl": 2500,
  "daily_pnl": 120,
  "performance_metrics": {
    "sharpe_ratio": 1.8,
    "max_drawdown": 5,
    "roi_since_inception": 5
  },
  "last_updated": "2023-06-16T14:30:00Z"
}
```

#### 1.2.5 调整运行中的代理策略
```
PUT /api/ai-agent/instances/{agent_id}/strategy
请求体：
{
  "asset_allocation": {
    "BTC": 25,
    "ETH": 45,
    "stablecoins": 20,
    "altcoins": 10
  },
  "risk_parameters": {
    "stop_loss_ratio": 8
  }
}
响应格式：
{
  "agent_id": "ai_987654",
  "status": "strategy_updated",
  "update_scheduled": true,
  "next_rebalance_at": "2023-06-16T20:00:00Z"
}
```

#### 1.2.6 提取收益
```
POST /api/ai-agent/instances/{agent_id}/withdraw
请求体：
{
  "amount": 1000,
  "target_asset": "USDC",
  "destination_address": "0x1234567890abcdef1234567890abcdef12345678"
}
响应格式：
{
  "withdrawal_id": "wd_234567",
  "status": "processing",
  "requested_amount": 1000,
  "estimated_fee": 5,
  "transaction_hash": null,
  "created_at": "2023-06-16T16:45:00Z"
}
```

### 1.3 交易执行接口

#### 1.3.1 获取市场数据
```
GET /api/ai-agent/market-data
查询参数：
- assets: BTC,ETH,USDC
- timeframe: 1h
- limit: 24
响应格式：
{
  "data": {
    "BTC": [
      {"timestamp": "2023-06-16T13:00:00Z", "price": 35000, "volume": 1200},
      // 更多数据点...
    ],
    "ETH": [
      // 类似BTC的数据...
    ]
  },
  "last_updated": "2023-06-16T14:00:00Z"
}
```

#### 1.3.2 获取交易历史
```
GET /api/ai-agent/instances/{agent_id}/transactions
查询参数：
- start_date: 2023-06-01
- end_date: 2023-06-16
- transaction_type: buy,sell
- asset: ETH
响应格式：
{
  "transactions": [
    {
      "tx_id": "tx_123456",
      "timestamp": "2023-06-15T14:30:00Z",
      "type": "buy",
      "asset": "ETH",
      "amount": 0.5,
      "price": 3200,
      "total_value": 1600,
      "status": "completed",
      "blockchain_tx_hash": "0xabcdef1234567890abcdef1234567890abcdef12"
    },
    // 更多交易...
  ],
  "total_count": 42,
  "page": 1,
  "page_size": 20
}
```

#### 1.3.3 获取当前持仓
```
GET /api/ai-agent/instances/{agent_id}/portfolio
响应格式：
{
  "total_value": 52500,
  "holdings": [
    {
      "asset": "BTC",
      "amount": 0.85,
      "value": 29750,
      "allocation_percentage": 56.67,
      "avg_cost": 34500
    },
    {
      "asset": "ETH",
      "amount": 5.2,
      "value": 16640,
      "allocation_percentage": 31.69,
      "avg_cost": 3150
    },
    // 更多资产...
  ],
  "last_updated": "2023-06-16T14:30:00Z"
}
```

#### 1.3.4 手动执行交易（可选功能）
```
POST /api/ai-agent/trade
请求体：
{
  "agent_id": "ai_987654",
  "type": "buy",
  "asset": "ETH",
  "amount": 0.5,
  "slippage_tolerance": 0.5,
  "expiry": 300
}
响应格式：
{
  "order_id": "ord_456789",
  "status": "pending",
  "requested_amount": 0.5,
  "estimated_price": 3200,
  "created_at": "2023-06-16T15:00:00Z"
}
```

### 1.4 结算与审计接口

#### 1.4.1 获取结算历史
```
GET /api/ai-agent/instances/{agent_id}/settlements
查询参数：
- start_date: 2023-06-01
- end_date: 2023-06-16
- settlement_type: daily,weekly,monthly
响应格式：
{
  "settlements": [
    {
      "settlement_id": "st_654321",
      "period": "2023-06-15",
      "total_revenue": 120,
      "user_share": 102,
      "platform_share": 18,
      "settlement_status": "completed",
      "transaction_hash": "0x1234567890abcdef1234567890abcdef12345678"
    },
    // 更多结算记录...
  ],
  "total_count": 15,
  "page": 1,
  "page_size": 10
}
```

#### 1.4.2 生成绩效报告
```
POST /api/ai-agent/instances/{agent_id}/generate-report
请求体：
{
  "report_type": "performance",
  "time_period": "monthly",
  "start_date": "2023-06-01",
  "end_date": "2023-06-30"
}
响应格式：
{
  "report_id": "rpt_789012",
  "status": "generated",
  "report_url": "/api/ai-agent/reports/rpt_789012.pdf",
  "metrics": {
    "total_roi": 10.5,
    "daily_roi": 0.24,
    "volatility": 12.3,
    "sharpe_ratio": 1.8,
    "max_drawdown": 5,
    "trades_executed": 42
  }
}
```

#### 1.4.3 获取风险评估
```
GET /api/ai-agent/instances/{agent_id}/risk-assessment
响应格式：
{
  "agent_id": "ai_987654",
  "overall_risk_score": 65,
  "risk_factors": [
    {
      "factor": "concentration",
      "score": 70,
      "assessment": "中等集中度风险",
      "recommendation": "考虑适度分散投资组合"
    },
    {
      "factor": "volatility",
      "score": 60,
      "assessment": "可接受的波动性",
      "recommendation": "当前风险水平与策略设置一致"
    }
  ],
  "liquidity_score": 85,
  "compliance_status": "compliant",
  "last_updated": "2023-06-16T14:30:00Z"
}
```

#### 1.4.4 导出交易数据
```
GET /api/ai-agent/instances/{agent_id}/export
查询参数：
- format: csv,json
- start_date: 2023-06-01
- end_date: 2023-06-16
- data_type: transactions,portfolio,settlements
响应格式：
{
  "export_id": "exp_345678",
  "download_url": "/api/ai-agent/exports/exp_345678.csv",
  "file_size": "1.2MB",
  "expires_at": "2023-06-17T14:30:00Z"
}
```

#### 1.4.5 区块链交易验证
```
GET /api/ai-agent/verify-transaction/{transaction_hash}
响应格式：
{
  "transaction_hash": "0xabcdef1234567890abcdef1234567890abcdef12",
  "blockchain": "ethereum",
  "block_number": 16543210,
  "timestamp": "2023-06-15T14:30:00Z",
  "status": "confirmed",
  "from_address": "0xagent_address_here",
  "to_address": "0xswap_router_address",
  "transaction_type": "swap",
  "details": {
    "input_asset": "USDC",
    "input_amount": 1600,
    "output_asset": "ETH",
    "output_amount": 0.5
  }
}
```