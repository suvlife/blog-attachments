# 美股实时数据源

## Yahoo Finance Chart API (v8) — 免费、无需认证

获取美股实时行情和历史K线数据。

**注意**: Yahoo Finance v7 quote API 已限制访问（返回 "Unauthorized"），v10 quoteSummary 需要 crumb 认证。**v8 chart API 是目前可用的免费无认证接口**。

### 实时行情 + 近N日K线

```
GET https://query1.finance.yahoo.com/v8/finance/chart/{TICKER}?range={range}&interval={interval}
```

**range 参数**:
| 值 | 含义 | 适用场景 |
|---|------|---------|
| 1d | 1天 | 盘中实时 |
| 5d | 5天 | 近一周 |
| 1mo | 1个月 | 近期走势 |
| 3mo | 3个月 | 季度分析 |
| 6mo | 6个月 | 中期结构 |
| 1y | 1年 | 52周高低 |
| 5y | 5年 | 长期趋势 |
| max | 全部 | 历史全量 |

**interval 参数**: 1m, 5m, 15m, 1h, 1d, 1wk, 1mo

### Python 解析 — 获取当前价格和涨跌

```python
import json, urllib.request

ticker = "AAPL"
url = f"https://query1.finance.yahoo.com/v8/finance/chart/{ticker}?range=5d&interval=1d"
req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
data = json.loads(urllib.request.urlopen(req, timeout=10).read())

result = data["chart"]["result"][0]
meta = result["meta"]

current = meta["regularMarketPrice"]
prev_close = meta["chartPreviousClose"]
change_pct = (current - prev_close) / prev_close * 100

print(f"{ticker}: ${current:.2f} ({change_pct:+.2f}%)")

# 5日收盘价
closes = result["indicators"]["quote"][0]["close"]
print(f"Recent closes: {[round(c, 2) for c in closes if c]}")
```

### Python 解析 — 批量获取多只股票

```python
import json, urllib.request, time

tickers = ["IREN", "XPEV", "COHR"]
for t in tickers:
    url = f"https://query1.finance.yahoo.com/v8/finance/chart/{t}?range=1mo&interval=1d"
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
    data = json.loads(urllib.request.urlopen(req, timeout=10).read())
    result = data["chart"]["result"][0]
    meta = result["meta"]
    print(f"{t}: ${meta['regularMarketPrice']:.2f}")
    time.sleep(0.3)  # 避免频率限制
```

### Python 解析 — 获取技术分析所需的历史数据

```python
import json, urllib.request

ticker = "AAPL"
url = f"https://query1.finance.yahoo.com/v8/finance/chart/{ticker}?range=6mo&interval=1wk"
req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
data = json.loads(urllib.request.urlopen(req, timeout=10).read())

result = data["chart"]["result"][0]
quotes = result["indicators"]["quote"][0]
closes = [c for c in quotes["close"] if c]
highs = [h for h in quotes["high"] if h]
lows = [l for l in quotes["low"] if l]
volumes = [v for v in quotes["volume"] if v]

# 计算均线
ma20w = sum(closes[-20:]) / len(closes[-20:]) if len(closes) >= 20 else None
ma10w = sum(closes[-10:]) / len(closes[-10:]) if len(closes) >= 10 else None

print(f"6mo High: {max(highs):.2f}")
print(f"6mo Low: {min(lows):.2f}")
print(f"20wk MA: {ma20w:.2f}" if ma20w else "20wk MA: N/A")
print(f"10wk MA: {ma10w:.2f}" if ma10w else "10wk MA: N/A")
```

## 52周高低点获取

使用 `range=max&interval=1mo` 获取月线数据，取最近12个月的高低点：

```python
# 近似52周高低（月线精度足够用于基本面分析）
recent_12m_high = max(highs[-12:])
recent_12m_low = min(lows[-12:])
```

精确52周高低使用 `range=1y&interval=1d` 的日线数据。

## 关键指数

| 指数 | Ticker | 用途 |
|------|--------|------|
| S&P 500 | %5EGSPC | 大盘基准 |
| NASDAQ | %5EIXIC | 科技股基准 |
| Russell 2000 | %5ERUT | 小盘股基准 |
| VIX | %5EVIX | 波动率/恐慌指数 |

注意: URL 中 `^` 需编码为 `%5E`。

## 数据局限性

- **基本面数据受限**: v7/v10 API 需要认证，免费用户无法获取 PE/PB/ROE 等指标
  - 替代方案: 用 web_search 或 browser 工具从 Yahoo Finance/Finviz 页面获取
  - 或用 finviz screener URL 抓取: `https://finviz.com/quote.ashx?t={TICKER}`
- **实时数据延迟**: 免费数据可能有15分钟延迟
- **频率限制**: 批量请求需加 `time.sleep(0.3)` 间隔
- **User-Agent 必须设置**: 否则返回 403

## 常见 Pitfalls

1. **Yahoo v7/v10 不可用**: 不要尝试 `v7/finance/quote` 或 `v10/finance/quoteSummary`，会返回 "Unauthorized" 或 "Invalid Crumb"。只用 v8 chart API。
2. **macOS grep 不支持 -P**: 用 Python 替代 `grep -oP` 进行正则提取。
3. **URL 特殊字符编码**: `^` → `%5E`，其他特殊字符也需要 URL encode。
4. **JSON 嵌套结构**: `data["chart"]["result"][0]["indicators"]["quote"][0]` 是固定的四层嵌套。
