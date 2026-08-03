# FA Agent 因子 DSL 规范

> 版本: v1.0 | 更新日期: 2026-08-03

---

## 一、概述

因子 DSL (Domain Specific Language) 是因子挖掘 Agent 用于表达因子计算逻辑的领域特定语言。

**设计目标**：
- 简洁：用最少的代码表达复杂的因子逻辑
- 可读：经济学含义一目了然
- 可执行：能被因子引擎解析并高效计算
- 可验证：结果可复现

---

## 二、DSL 语法

### 2.1 基本结构

```
因子表达式 = 算子(参数1, 参数2, ...)
```

### 2.2 算子分类

#### 时序算子 (ts_*)
对单个标的的时间序列进行操作。

| 算子 | 语法 | 说明 | 示例 |
|------|------|------|------|
| `ts_mean` | `ts_mean(x, n)` | 过去n期均值 | `ts_mean(close, 20)` |
| `ts_std` | `ts_std(x, n)` | 过去n期标准差 | `ts_std(ret, 20)` |
| `ts_sum` | `ts_sum(x, n)` | 过去n期累加 | `ts_sum(volume, 5)` |
| `ts_max` | `ts_max(x, n)` | 过去n期最大值 | `ts_max(high, 20)` |
| `ts_min` | `ts_min(x, n)` | 过去n期最小值 | `ts_min(low, 20)` |
| `ts_rank` | `ts_rank(x, n)` | 过去n期排名百分位 | `ts_rank(close, 252)` |
| `ts_delay` | `ts_delay(x, n)` | 延迟n期 | `ts_delay(close, 1)` |
| `ts_delta` | `ts_delta(x, n)` | 与n期前的差值 | `ts_delta(close, 5)` |
| `ts_pct` | `ts_pct(x, n)` | 与n期前的变化率 | `ts_pct(close, 5)` |
| `ts_corr` | `ts_corr(x, y, n)` | 过去n期相关系数 | `ts_corr(ret, volume, 20)` |
| `ts_cov` | `ts_cov(x, y, n)` | 过去n期协方差 | `ts_cov(ret, mkt_ret, 60)` |
| `ts_zscore` | `ts_zscore(x, n)` | 过去n期标准化 | `ts_zscore(volume, 20)` |
| `ts_skew` | `ts_skew(x, n)` | 过去n期偏度 | `ts_skew(ret, 60)` |
| `ts_kurt` | `ts_kurt(x, n)` | 过去n期峰度 | `ts_kurt(ret, 60)` |
| `ts_argmax` | `ts_argmax(x, n)` | 过去n期最大值位置 | `ts_argmax(high, 20)` |
| `ts_argmin` | `ts_argmin(x, n)` | 过去n期最小值位置 | `ts_argmin(low, 20)` |
| `ts_decay` | `ts_decay(x, n)` | 指数衰减加权 | `ts_decay(ret, 10)` |

#### 截面算子 (cs_*)
对同一时刻的所有标的进行操作。

| 算子 | 语法 | 说明 | 示例 |
|------|------|------|------|
| `cs_rank` | `cs_rank(x)` | 截面排名百分位 [0,1] | `cs_rank(pe)` |
| `cs_zscore` | `cs_zscore(x)` | 截面标准化 | `cs_zscore(roe)` |
| `cs_demean` | `cs_demean(x)` | 截面去均值 | `cs_demean(ret)` |
| `cs_winsorize` | `cs_winsorize(x, q)` | 截面缩尾 | `cs_winsorize(pe, 0.01)` |
| `cs_neutralize` | `cs_neutralize(x, groups)` | 行业/规模中性化 | `cs_neutralize(value, industry)` |
| `cs_quantile` | `cs_quantile(x, n)` | 截面分组 | `cs_quantile(pe, 10)` |

#### 基础算子
| 算子 | 语法 | 说明 | 示例 |
|------|------|------|------|
| `log` | `log(x)` | 自然对数 | `log(market_cap)` |
| `abs` | `abs(x)` | 绝对值 | `abs(ret)` |
| `sign` | `sign(x)` | 符号函数 | `sign(momentum)` |
| `sqrt` | `sqrt(x)` | 平方根 | `sqrt(variance)` |
| `pow` | `pow(x, n)` | 幂运算 | `pow(ret, 2)` |
| `max` | `max(x, y)` | 取大 | `max(ret, 0)` |
| `min` | `min(x, y)` | 取小 | `min(pe, 100)` |
| `if_else` | `if_else(cond, x, y)` | 条件选择 | `if_else(pe>0, pe, nan)` |

#### 运算符
| 运算符 | 说明 | 示例 |
|--------|------|------|
| `+` | 加法 | `close + open` |
| `-` | 减法 | `high - low` |
| `*` | 乘法 | `price * volume` |
| `/` | 除法 | `profit / revenue` |
| `>` | 大于 | `pe > 20` |
| `<` | 小于 | `pb < 1` |
| `==` | 等于 | `industry == 'bank'` |
| `&` | 逻辑与 | `(pe < 20) & (roe > 0.1)` |
| `|` | 逻辑或 | `(pe < 10) | (pb < 0.5)` |

---

## 三、内置变量

### 3.1 行情变量

| 变量名 | 说明 |
|--------|------|
| `open` | 开盘价 |
| `high` | 最高价 |
| `low` | 最低价 |
| `close` | 收盘价 |
| `volume` | 成交量 |
| `amount` | 成交额 |
| `vwap` | 成交量加权均价 |
| `ret` | 日收益率 |
| `turnover` | 换手率 |

### 3.2 财务变量

| 变量名 | 说明 |
|--------|------|
| `pe` | 市盈率 |
| `pe_ttm` | 市盈率TTM |
| `pb` | 市净率 |
| `ps` | 市销率 |
| `roe` | 净资产收益率 |
| `roa` | 总资产收益率 |
| `market_cap` | 总市值 |
| `float_cap` | 流通市值 |
| `revenue` | 营业收入 |
| `profit` | 净利润 |
| `gross_margin` | 毛利率 |
| `net_margin` | 净利率 |

### 3.3 分组变量

| 变量名 | 说明 |
|--------|------|
| `industry` | 申万一级行业 |
| `sector` | 大类板块 |
| `size_group` | 市值分组 |

---

## 四、因子示例

### 4.1 动量因子

```python
# 20日动量
momentum_20d = ts_pct(close, 20)

# 动量反转（近期涨多了反转）
reversal_5d = -1 * ts_pct(close, 5)

# 相对强度（排名动量）
rel_strength = cs_rank(ts_pct(close, 60))
```

### 4.2 估值因子

```python
# EP（盈利收益率）
ep = 1 / pe_ttm

# BP（账面价值比）
bp = 1 / pb

# 相对PE（相对行业中位数）
rel_pe = pe_ttm / cs_neutralize(pe_ttm, industry)
```

### 4.3 波动因子

```python
# 20日波动率
volatility_20d = ts_std(ret, 20) * sqrt(252)

# 特质波动率（去市场后的残差波动）
idio_vol = ts_std(ret - beta * mkt_ret, 60)

# 振幅
amplitude = (high - low) / ts_delay(close, 1)
```

### 4.4 流动性因子

```python
# Amihud非流动性
illiq = ts_mean(abs(ret) / amount, 20)

# 换手率
avg_turnover = ts_mean(turnover, 20)

# 成交额占比
volume_ratio = amount / ts_mean(amount, 60)
```

### 4.5 资金流因子

```python
# 量价相关性（资金流向）
vol_price_corr = ts_corr(ret, volume, 20)

# 异常成交量
abnormal_volume = volume / ts_mean(volume, 20) - 1
```

### 4.6 质量因子

```python
# ROE稳定性
roe_stability = -1 * ts_std(roe, 4)  # 季度数据

# 盈利加速度
profit_accel = ts_delta(ts_pct(profit, 4), 4)
```

### 4.7 复合因子

```python
# 低估值+高成长
value_growth = cs_rank(ep) + cs_rank(ts_pct(profit, 4))

# GARP（合理价格的成长）
garp = cs_rank(ts_pct(profit, 4)) / cs_rank(pe_ttm)

# 价值陷阱过滤（低估值但盈利稳定）
safe_value = if_else(
    roe > 0.05,
    cs_rank(ep),
    nan
)
```

---

## 五、因子预处理流程

```
原始因子值
    │
    ▼
┌─────────────┐
│  缺失值处理  │  → 行业中位数填充 / 截面中位数 / 删除
└─────────────┘
    │
    ▼
┌─────────────┐
│  去极值     │  → MAD / Winsorize (3σ / 1%分位)
└─────────────┘
    │
    ▼
┌─────────────┐
│  标准化     │  → Z-Score / Rank
└─────────────┘
    │
    ▼
┌─────────────┐
│  中性化     │  → 行业中性 / 市值中性 (回归残差)
└─────────────┘
    │
    ▼
标准化因子值
```

### 预处理 DSL

```python
# 完整预处理流程
factor_processed = cs_zscore(
    cs_neutralize(
        cs_winsorize(raw_factor, 0.01),
        [industry, log(market_cap)]
    )
)
```

---

## 六、DSL 引擎实现

### 6.1 解析器

```python
# src/factor_engine/parser.py

import ast
from typing import Dict, Any

class FactorDSLParser:
    """因子DSL解析器"""
    
    ALLOWED_OPERATORS = {
        # 时序算子
        'ts_mean', 'ts_std', 'ts_sum', 'ts_max', 'ts_min',
        'ts_rank', 'ts_delay', 'ts_delta', 'ts_pct',
        'ts_corr', 'ts_cov', 'ts_zscore', 'ts_decay',
        # 截面算子
        'cs_rank', 'cs_zscore', 'cs_demean', 'cs_winsorize', 'cs_neutralize',
        # 基础算子
        'log', 'abs', 'sign', 'sqrt', 'pow', 'max', 'min', 'if_else',
    }
    
    def parse(self, expression: str) -> Dict[str, Any]:
        """
        解析DSL表达式为AST
        
        Args:
            expression: DSL表达式字符串
            
        Returns:
            解析后的AST字典
        """
        tree = ast.parse(expression, mode='eval')
        return self._convert_ast(tree.body)
    
    def _convert_ast(self, node) -> Dict[str, Any]:
        """递归转换AST节点"""
        if isinstance(node, ast.Call):
            func_name = node.func.id
            if func_name not in self.ALLOWED_OPERATORS:
                raise ValueError(f"Unknown operator: {func_name}")
            
            return {
                'type': 'call',
                'func': func_name,
                'args': [self._convert_ast(arg) for arg in node.args]
            }
        
        elif isinstance(node, ast.Name):
            return {'type': 'variable', 'name': node.id}
        
        elif isinstance(node, ast.Constant):
            return {'type': 'constant', 'value': node.value}
        
        elif isinstance(node, ast.BinOp):
            return {
                'type': 'binop',
                'op': type(node.op).__name__,
                'left': self._convert_ast(node.left),
                'right': self._convert_ast(node.right)
            }
        
        # ... 其他节点类型
```

### 6.2 执行器

```python
# src/factor_engine/executor.py

import pandas as pd
import numpy as np
from typing import Dict, Any

class FactorDSLExecutor:
    """因子DSL执行器"""
    
    def __init__(self, data: pd.DataFrame):
        """
        Args:
            data: 包含所有变量的DataFrame，索引为(date, symbol)
        """
        self.data = data
    
    def execute(self, ast: Dict[str, Any]) -> pd.Series:
        """执行AST，返回因子值"""
        
        if ast['type'] == 'call':
            return self._execute_call(ast)
        elif ast['type'] == 'variable':
            return self.data[ast['name']]
        elif ast['type'] == 'constant':
            return ast['value']
        elif ast['type'] == 'binop':
            return self._execute_binop(ast)
    
    def _execute_call(self, ast: Dict) -> pd.Series:
        """执行函数调用"""
        func_name = ast['func']
        args = [self.execute(arg) for arg in ast['args']]
        
        # 时序算子
        if func_name == 'ts_mean':
            return args[0].groupby('symbol').rolling(args[1]).mean()
        elif func_name == 'ts_std':
            return args[0].groupby('symbol').rolling(args[1]).std()
        elif func_name == 'ts_rank':
            return args[0].groupby('symbol').rolling(args[1]).apply(
                lambda x: (x[-1] > x).sum() / len(x)
            )
        # ... 其他算子
        
        # 截面算子
        elif func_name == 'cs_rank':
            return args[0].groupby('date').rank(pct=True)
        elif func_name == 'cs_zscore':
            grouped = args[0].groupby('date')
            return (args[0] - grouped.transform('mean')) / grouped.transform('std')
        # ... 其他算子
```

---

## 七、因子元数据

每个因子必须包含的元数据：

```yaml
factor_meta:
  factor_id: "FAC-20260803-001"
  expression: "cs_rank(ts_zscore(volume, 20) * ts_mean(ret, 5))"
  
  # 分类
  factor_class: "量价"
  sub_class: "资金流"
  
  # 适用范围
  universe: ["A股"]
  horizon: "1d"
  
  # 经济学含义（必填！）
  economic_logic: |
    该因子捕捉异常成交量与短期收益的交互效应。
    当某股票成交量异常放大（相对自身历史）且伴随正向收益时，
    可能暗示有知情交易者在建仓，预示后续上涨。
  
  # 初检指标
  evidence:
    ic_mean: 0.035
    ic_ir: 0.42
    turnover: 0.25
    coverage: 0.95
    corr_to_existing: 0.32
  
  # 创建信息
  created_by: "因子挖掘Agent"
  created_at: "2026-08-03T10:30:00"
  
  # 状态
  status: "CANDIDATE"  # CANDIDATE | APPROVED | DEPRECATED
```

---

*文档版本: v1.0 | 更新日期: 2026-08-03*
