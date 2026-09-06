# Stock AI MAX v2

一个可运行、可部署为在线网页的美股 AI 概率分析系统。

> 目标不是制造“稳赚”或“100%准确”的幻觉，而是尽可能提高数据覆盖、减少回测泄漏、校准概率，并在数据差时明确降低置信度。

## MAX 版核心升级

### 1. 6 类机器学习模型

- Logistic Regression
- Random Forest
- XGBoost
- LightGBM
- Extra Trees
- Histogram Gradient Boosting

每个模型分别预测未来 1 / 3 / 5 个交易日上涨概率。

### 2. 时间顺序训练 + 概率校准

不再随机打乱时间序列。训练流程采用：

`Train → Calibration → Test`

并记录：Accuracy、Balanced Accuracy、Precision、Recall、ROC AUC、Brier Score、Log Loss、MCC。

概率会做 Platt-style 校准，避免模型轻易输出虚假的 90%+。

### 3. 更丰富的历史特征

除原有 RSI / MACD / EMA / Bollinger / ATR / VWAP / Volume / Support / Resistance / Momentum / ROC 外，加入：

- 1/2/3/5/10/20日收益
- 5/10/20/60日波动率
- ADX / +DI / -DI
- Stochastic
- MFI
- OBV
- Volume Z-score
- Gap
- Intraday range
- Close location
- EMA slope
- Bollinger position
- Price position in 20/50-day range

### 4. 大盘上下文进入 ML

训练模型时可加入：

- SPY
- QQQ
- VIX
- 10Y Treasury
- 股票相对 SPY / QQQ 强弱

这些是按历史日期对齐，不把未来数据泄漏给模型。

### 5. 多来源新闻雷达

新闻系统会并行尝试：

- yfinance
- SEC EDGAR
- Alpha Vantage
- FMP
- Massive / Polygon
- Finnhub
- Marketaux

并进行：

- 去重
- 时间衰减
- 来源可靠度权重
- 新闻重要度
- bullish / neutral / bearish
- -100 到 +100 情绪
- 催化剂识别
- 风险识别
- Breaking news 数量
- 新闻覆盖评分
- 活跃数据源数量

如果设置 OpenAI API Key，会对最高重要度新闻做一次批量 LLM 分类；失败会自动回退到本地模型。

### 6. SEC 官方事件

可抓取公司近期重要 SEC filing，例如：

- 8-K
- 10-Q
- 10-K
- 6-K / 20-F
- Form 4
- SC 13D / SC 13G

SEC filing 本身不会被自动判定为 bullish 或 bearish，除非提供的文本确实支持方向判断。

### 7. Options 情绪

通过可用期权链计算：

- Put / Call Volume
- Put / Call Open Interest
- ATM IV
- Implied Move
- Options Score

如果没有期权或数据不可用，该因子自动退出 ensemble，不会填假数据。

### 8. 相对强度

比较股票对：

- SPY
- QQQ
- 所属 Sector ETF

并加入 5 / 20 / 60 日相对表现。

半导体公司会优先使用 SMH 作为行业参考。

### 9. 更完整宏观

包括：

- SPY / QQQ
- VIX
- 10Y
- 2Y
- 10Y-2Y yield curve
- CPI / Core CPI / PPI
- Nonfarm Payrolls
- Unemployment
- Fed Funds
- Real 10Y
- Financial Conditions
- Dollar Index
- Oil / Gold
- Semiconductor ETF

FRED Key 可选；没有 key 会尝试 CSV fallback。

### 10. 改进 Monte Carlo

不再只使用正态 GBM。MAX 默认使用历史 log return 的经验 bootstrap，更容易保留 fat-tail / gap 风险。

输出：

- P05
- P10 Bear
- P25
- P50 Base
- P75
- P90 Bull
- P95
- 目标价 1 / 3 / 5 日触及概率

AI 概率只对模拟漂移做小幅调整，避免循环强化导致过度自信。

### 11. Walk-forward 回测

除了普通 chronological holdout，还加入 expanding-window walk-forward：

`历史训练 → 下一段测试 → 扩展训练 → 下一段测试 ...`

这比把全部历史随机切分更接近真实运行。

### 12. 数据质量 / 信号质量

系统显示：

- Model Confidence
- Data Quality
- News Coverage
- Signal Quality
- Model disagreement

如果模型分歧大或数据覆盖差，会显示：

`信号不足 / 等待确认`

而不是硬给强势买卖结论。

## 网页功能

- 🚀 一键深度分析
- 🛰️ 新闻雷达
- 🎯 目标价概率
- 🏆 股票排行榜
- 🧪 回测实验室
- 📊 模型准确率
- ⚙️ 数据源状态

## 本地启动（懂 Python 时使用）

Python 推荐 3.12。

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
pip install -r requirements.txt
streamlit run app.py
```

## 在线网页部署（推荐）

看 `DEPLOY_ONLINE.md`。

部署成功后你只需要打开一个 `streamlit.app` 网页，不再碰 BAT / CMD。

## API Key

复制 `.env.example` 为 `.env`，或者在云端 Secrets 中填写。

所有 API Key 都通过环境变量读取，代码中不写死。

## 测试

```bash
pytest -q
```

当前测试覆盖：

- 技术指标
- 上下文特征
- 新闻去重
- 6模型训练
- 概率校准
- Monte Carlo
- SQLite
- Chronological backtest
- Walk-forward smoke test

## 重要限制

1. 不存在能保证“最高准确率”的股票模型。
2. 不存在能保证抓到“全部新闻”的单一 API。
3. 新闻数据、财报数据和行情源可能有延迟、权限限制或 rate limit。
4. 历史准确率不能保证未来准确率。
5. 1-5 日短线价格含大量不可预测噪音，概率应结合风险/置信度一起看。

MAX 的设计原则是：**覆盖更多信息 + 降低数据泄漏 + 概率校准 + 真实回测 + 缺数据时承认不确定性。**
