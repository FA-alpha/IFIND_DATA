# FA Agent MCP 工具接口规范

> 版本: v1.0 | 更新日期: 2026-08-03

---

## 一、概述

MCP (Model Context Protocol) 是 Agent 调用工具的标准化协议。每个工具定义清晰的输入/输出 schema，确保 Agent 调用确定性、可复现。

**设计原则**：
- LLM 负责决策（调哪个工具、传什么参数）
- 工具负责计算（确定性、可复现）
- 工具结果结构化，便于 LLM 解读

---

## 二、通用接口格式

### 请求格式

```json
{
  "tool_name": "string",
  "request_id": "string",
  "params": {
    // 工具特定参数
  }
}
```

### 响应格式

```json
{
  "request_id": "string",
  "status": "success | error",
  "result": {
    // 工具特定返回
  },
  "error": {
    "code": "string",
    "message": "string"
  },
  "metadata": {
    "execution_time_ms": 123,
    "data_snapshot_id": "string"
  }
}
```

---

## 三、因子挖掘 Agent 工具集

### 3.1 因子DSL引擎

**工具名**: `factor_dsl_engine`

**功能**: 解析并执行因子 DSL 表达式

```yaml
# 输入
params:
  expression: "cs_rank(ts_zscore(volume, 20))"  # DSL表达式
  universe: "A股"                                # 股票池
  start_date: "2025-08-01"
  end_date: "2026-08-01"
  
# 输出
result:
  factor_values:                    # DataFrame序列化
    columns: ["date", "symbol", "factor_value"]
    data: [...]
  stats:
    coverage: 0.95
    nan_ratio: 0.02
    unique_values: 4521
```

### 3.2 IC计算器

**工具名**: `ic_calculator`

**功能**: 计算因子 IC、IR、衰减曲线

```yaml
# 输入
params:
  factor_values:                    # 因子值DataFrame
    columns: ["date", "symbol", "factor_value"]
    data: [...]
  forward_returns:                  # 未来收益
    columns: ["date", "symbol", "return_1d", "return_5d", "return_20d"]
    data: [...]
  horizons: [1, 5, 20]              # 计算周期

# 输出
result:
  ic_series:                        # 每日IC
    columns: ["date", "ic_1d", "ic_5d", "ic_20d"]
    data: [...]
  summary:
    ic_mean:
      "1d": 0.035
      "5d": 0.028
      "20d": 0.018
    ic_std:
      "1d": 0.08
      "5d": 0.06
      "20d": 0.05
    ic_ir:
      "1d": 0.44
      "5d": 0.47
      "20d": 0.36
    t_stat:
      "1d": 3.2
      "5d": 2.8
      "20d": 2.1
  decay_curve:                      # 衰减曲线
    lags: [1, 2, 3, 5, 10, 20]
    autocorr: [1.0, 0.92, 0.85, 0.72, 0.51, 0.28]
```

### 3.3 因子相关性检查器

**工具名**: `factor_correlation_checker`

**功能**: 检查新因子与现有因子池的相关性

```yaml
# 输入
params:
  new_factor:
    columns: ["date", "symbol", "factor_value"]
    data: [...]
  existing_factors:                 # 现有因子池ID列表
    - "FAC-001"
    - "FAC-002"
    - "FAC-003"

# 输出
result:
  max_correlation: 0.32
  most_correlated_factor: "FAC-002"
  correlation_matrix:
    - ["", "FAC-001", "FAC-002", "FAC-003"]
    - ["new", 0.15, 0.32, 0.08]
  is_redundant: false               # corr > 0.7 为冗余
```

---

## 四、模型研究 Agent 工具集

### 4.1 AutoML训练器

**工具名**: `automl_trainer`

**功能**: 自动选择模型、调参、训练

```yaml
# 输入
params:
  features:                         # 特征矩阵
    columns: ["date", "symbol", "fac1", "fac2", "fac3"]
    data: [...]
  target:                           # 目标变量
    columns: ["date", "symbol", "target"]
    data: [...]
  model_families: ["lightgbm", "xgboost", "catboost"]
  train_end_date: "2025-12-31"
  val_start_date: "2026-01-01"
  val_end_date: "2026-06-30"
  objective: "ic"                   # 优化目标: ic | sharpe | mse

# 输出
result:
  best_model:
    family: "lightgbm"
    hyperparams:
      num_leaves: 31
      learning_rate: 0.05
      n_estimators: 200
    model_uri: "s3://models/MDL-20260803-001.pkl"
  metrics:
    train:
      ic: 0.08
      sharpe: 2.5
    validation:
      ic: 0.05
      sharpe: 1.8
  feature_importance:
    - ["fac1", 0.35]
    - ["fac2", 0.28]
    - ["fac3", 0.22]
```

### 4.2 过拟合检测器

**工具名**: `overfit_detector`

**功能**: 计算过拟合概率 (PBO)

```yaml
# 输入
params:
  model_uri: "s3://models/MDL-20260803-001.pkl"
  features: {...}
  target: {...}
  n_splits: 10                      # 交叉验证折数

# 输出
result:
  pbo: 0.32                         # 过拟合概率
  is_oos_degradation_ratio: 0.72   # 样本外/样本内比值
  cv_results:
    - {fold: 1, is_sharpe: 2.1, oos_sharpe: 1.5}
    - {fold: 2, is_sharpe: 2.3, oos_sharpe: 1.8}
    # ...
  conclusion: "PASS"                # PASS if pbo < 0.5
```

---

## 五、优化器 Agent 工具集

### 5.1 组合优化器

**工具名**: `portfolio_optimizer`

**功能**: cvxpy 组合优化

```yaml
# 输入
params:
  expected_returns:                 # 预期收益
    columns: ["symbol", "expected_return", "confidence"]
    data: [...]
  covariance_matrix:                # 协方差矩阵
    symbols: ["000001.SZ", "000002.SZ", ...]
    data: [[...], [...], ...]
  constraints:
    max_weight: 0.05
    min_weight: 0.0
    max_industry_weight:
      银行: 0.15
      科技: 0.15
    max_leverage: 1.5
    max_turnover: 0.3               # 相对上期
  objective: "max_sharpe"           # max_sharpe | min_variance | risk_parity
  previous_weights:                 # 上期权重（用于计算换手）
    columns: ["symbol", "weight"]
    data: [...]

# 输出
result:
  optimal_weights:
    columns: ["symbol", "weight"]
    data: [["000001.SZ", 0.032], ["000002.SZ", 0.028], ...]
  metrics:
    expected_return: 0.15
    expected_risk: 0.12
    expected_sharpe: 1.25
    turnover: 0.18
  constraint_status:                # 约束是否binding
    max_weight: false
    银行_industry: true             # 行业约束生效
    max_leverage: false
```

### 5.2 协方差估计器

**工具名**: `covariance_estimator`

**功能**: 估计协方差矩阵

```yaml
# 输入
params:
  returns:                          # 收益率矩阵
    columns: ["date", "symbol", "return"]
    data: [...]
  method: "shrinkage"               # sample | shrinkage | factor_model
  lookback_days: 252

# 输出
result:
  covariance_matrix:
    symbols: [...]
    data: [[...], [...], ...]
  estimation_method: "ledoit_wolf_shrinkage"
  effective_samples: 245
```

---

## 六、回测 Agent 工具集

### 6.1 回测引擎

**工具名**: `backtest_engine`

**功能**: 执行策略回测

```yaml
# 输入
params:
  strategy_config:
    rebalance_freq: "weekly"
    initial_capital: 10000000
    commission: 0.0003
    slippage: 0.001
  signals:                          # 每期目标权重
    columns: ["date", "symbol", "target_weight"]
    data: [...]
  price_data:                       # 价格数据
    columns: ["date", "symbol", "open", "high", "low", "close", "volume"]
    data: [...]
  benchmark: "000300.SH"

# 输出
result:
  nav_series:                       # 净值曲线
    columns: ["date", "nav", "benchmark_nav"]
    data: [...]
  summary:
    total_return: 0.35
    annual_return: 0.18
    annual_volatility: 0.12
    sharpe: 1.5
    max_drawdown: 0.08
    calmar: 2.25
    win_rate: 0.58
    profit_loss_ratio: 1.8
  trades:                           # 交易记录
    columns: ["date", "symbol", "direction", "quantity", "price", "commission"]
    data: [...]
```

### 6.2 压力测试器

**工具名**: `stress_tester`

**功能**: 历史情景压力测试

```yaml
# 输入
params:
  portfolio_weights:
    columns: ["symbol", "weight"]
    data: [...]
  scenarios:
    - name: "2015股灾"
      start_date: "2015-06-12"
      end_date: "2015-08-26"
    - name: "2024熔断"
      start_date: "2024-01-02"
      end_date: "2024-01-10"
    - name: "custom_scenario"
      shocks:                       # 自定义冲击
        - {factor: "market", shock: -0.15}
        - {factor: "size", shock: 0.05}

# 输出
result:
  scenario_results:
    - scenario: "2015股灾"
      portfolio_return: -0.25
      benchmark_return: -0.45
      relative_return: 0.20
      max_drawdown: 0.28
    - scenario: "2024熔断"
      portfolio_return: -0.08
      benchmark_return: -0.12
      relative_return: 0.04
      max_drawdown: 0.10
```

### 6.3 多重检验校正器

**工具名**: `multiple_testing_corrector`

**功能**: 防止策略过拟合的多重检验

```yaml
# 输入
params:
  strategy_sharpe: 1.8
  num_strategies_tested: 50         # 测试了多少个策略
  method: "bonferroni"              # bonferroni | fdr | romano_wolf

# 输出
result:
  adjusted_pvalue: 0.08
  is_significant: true              # 校正后是否显著
  haircut_ratio: 0.65              # 预期夏普打折
  expected_oos_sharpe: 1.17        # 预期样本外夏普
```

---

## 七、交易执行 Agent 工具集

### 7.1 交易网关

**工具名**: `trading_gateway`

**功能**: 执行交易指令

```yaml
# 输入
params:
  orders:
    - symbol: "000001.SZ"
      direction: "BUY"
      quantity: 1000
      order_type: "LIMIT"
      price: 10.50
      algo: "TWAP"
      duration_minutes: 30
    - symbol: "000002.SZ"
      direction: "SELL"
      quantity: 500
      order_type: "MARKET"

# 输出
result:
  executions:
    - order_id: "ORD-001"
      symbol: "000001.SZ"
      filled_quantity: 1000
      avg_price: 10.48
      slippage: -0.0019
      status: "FILLED"
    - order_id: "ORD-002"
      symbol: "000002.SZ"
      filled_quantity: 500
      avg_price: 15.22
      slippage: 0.0005
      status: "FILLED"
  summary:
    total_orders: 2
    filled_orders: 2
    total_commission: 15.50
    total_slippage_cost: 1.23
```

### 7.2 撤单计数器

**工具名**: `cancel_counter`

**功能**: 监控撤单次数

```yaml
# 输入
params:
  query_date: "2026-08-03"

# 输出
result:
  today_cancel_count: 320
  limit: 500
  remaining: 180
  warning_level: "YELLOW"           # GREEN | YELLOW | RED
  recommendation: "建议减少挂单频率"
```

---

## 八、风控 Agent 工具集

### 8.1 风险计算器

**工具名**: `risk_calculator`

**功能**: 计算 VaR/CVaR 等风险指标

```yaml
# 输入
params:
  portfolio_weights:
    columns: ["symbol", "weight"]
    data: [...]
  returns_history:                  # 历史收益
    columns: ["date", "symbol", "return"]
    data: [...]
  confidence_level: 0.95
  method: "historical"              # historical | parametric | monte_carlo

# 输出
result:
  var_95: 0.025                     # 95% VaR
  cvar_95: 0.035                    # 95% CVaR
  annual_volatility: 0.15
  max_drawdown_expected: 0.12
  beta: 0.85
```

### 8.2 暴露分析器

**工具名**: `exposure_analyzer`

**功能**: 分析组合风险暴露

```yaml
# 输入
params:
  portfolio_weights:
    columns: ["symbol", "weight"]
    data: [...]
  
# 输出
result:
  industry_exposure:
    银行: 0.18
    科技: 0.15
    消费: 0.12
    # ...
  factor_exposure:
    size: 0.25
    value: -0.15
    momentum: 0.32
    volatility: -0.08
  concentration:
    top5_weight: 0.28
    top10_weight: 0.45
    hhi: 0.032
  breaches:
    - type: "industry"
      name: "银行"
      value: 0.18
      limit: 0.15
      is_breach: true
```

### 8.3 市场状态识别器

**工具名**: `regime_detector`

**功能**: 识别市场状态

```yaml
# 输入
params:
  market_data:
    columns: ["date", "index_close", "volume", "volatility"]
    data: [...]
  method: "hmm"                     # hmm | rule_based

# 输出
result:
  current_regime: "震荡"            # 牛市 | 熊市 | 震荡
  regime_probability:
    牛市: 0.15
    熊市: 0.25
    震荡: 0.60
  regime_history:
    columns: ["date", "regime", "probability"]
    data: [...]
  regime_change_detected: false
```

---

## 九、复盘 Agent 工具集

### 9.1 归因引擎

**工具名**: `attribution_engine`

**功能**: 收益/风险归因分析

```yaml
# 输入
params:
  portfolio_returns:
    columns: ["date", "return"]
    data: [...]
  factor_returns:                   # 因子收益
    columns: ["date", "size", "value", "momentum", "industry_银行", ...]
    data: [...]
  portfolio_exposures:              # 组合暴露
    columns: ["date", "size", "value", "momentum", "industry_银行", ...]
    data: [...]

# 输出
result:
  attribution:
    total_return: 0.15
    factor_attribution:
      size: 0.03
      value: -0.02
      momentum: 0.05
      industry_银行: 0.02
    selection_return: 0.07          # 选股贡献
    timing_return: 0.00             # 择时贡献
  top_contributors:
    - {source: "momentum", contribution: 0.05}
    - {source: "selection", contribution: 0.07}
  top_detractors:
    - {source: "value", contribution: -0.02}
```

### 9.2 衰减检测器

**工具名**: `decay_detector`

**功能**: 检测因子衰减

```yaml
# 输入
params:
  factor_id: "FAC-001"
  ic_history:
    columns: ["date", "ic"]
    data: [...]
  lookback_periods:
    recent: 60                      # 近期天数
    baseline: 252                   # 基准天数

# 输出
result:
  is_decayed: true
  baseline_ic_mean: 0.05
  recent_ic_mean: -0.01
  ic_change: -0.06
  t_statistic: -3.2
  p_value: 0.002
  recommendation: "建议暂停使用该因子"
```

---

*文档版本: v1.0 | 更新日期: 2026-08-03*
