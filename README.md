# iFind 量化数据平台

> 基于同花顺 iFind 数据接口的多因子量化数据采集与管理系统

## 📋 项目概述

本项目旨在对接同花顺 iFind 金融数据平台，构建标准化的量化数据采集、存储与管理系统，为多因子选股、策略回测、实盘交易提供数据支撑。

## 🎯 核心目标

1. **全量数据对接** - 对接 iFind 平台所有可用数据接口
2. **因子库构建** - 构建覆盖技术面、基本面、资金面的完整因子库
3. **数据质量保证** - PIT 时点数据，避免回测中的未来函数问题
4. **高效存储** - 采用 PostgreSQL + Parquet 混合存储架构
5. **增量更新** - 日更新机制，优化 API 流量使用

## 📁 文档目录

| 文档 | 说明 |
|------|------|
| [需求文档](docs/00-需求文档.md) | 项目背景、功能需求、非功能需求 |
| [因子列表](docs/01-因子列表.md) | 60+ 量化因子完整清单及 iFind 指标映射 |
| [数据架构设计](docs/02-数据架构设计.md) | 技术架构、数据库设计、采集模块设计 |

## 🔧 技术栈

- **数据采集**: Python + iFinDPy SDK
- **元数据存储**: PostgreSQL
- **行情/因子存储**: Apache Parquet
- **缓存**: Redis
- **调度**: APScheduler / Cron

## 📊 数据覆盖

| 数据类型 | 覆盖范围 |
|---------|---------|
| 股票行情 | 全A股日线/分钟线 |
| 估值因子 | PE/PB/PS/市值等 |
| 财务因子 | ROE/ROA/毛利率/增长率等 |
| 资金因子 | 主力资金/融资融券 |
| 板块数据 | 指数成分/行业分类 |

## 🚀 快速开始

```bash
# 1. 克隆项目
git clone https://github.com/FA-alpha/IFIND_DATA.git
cd IFIND_DATA

# 2. 安装依赖
pip install -r requirements.txt

# 3. 配置 iFind 账号
cp config/config.example.yaml config/config.yaml
# 编辑 config.yaml 填入账号密码

# 4. 初始化数据库
python scripts/init_db.py

# 5. 启动数据采集
python -m src.scheduler.daily_update
```

## 📈 API 流量规划

| 版本 | 行情数据 | 基本面数据 | 适用场景 |
|------|---------|-----------|---------|
| 免费版 | 550万/月 | 180万/月 | 原型开发 |
| 试用版 | 1.5亿/周 | 500万/周 | 策略回测 |
| 正式版 | 1.5亿/周 | 500万/周 | 生产环境 |

## 📝 License

MIT License

---

*Maintained by FourierAlpha Team*
