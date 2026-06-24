# A股实时数据源

## 东方财富 Push API（免费、无需认证）

获取A股实时行情数据（股价、PE、PB、市值等）。

### 基础行情接口

```
GET https://push2.eastmoney.com/api/qt/ulist.np/get
```

**参数**：
- `fltt=2` — 浮点数格式
- `fields=f2,f3,f4,f5,f6,f7,f8,f9,f10,f12,f13,f14,f15,f16,f17,f18,f20,f21,f23,f24,f25,f100` — 返回字段
- `secids=市场.代码` — 证券代码列表

**市场代码**：
- `1.xxxxxx` = 上海证券交易所
- `0.xxxxxx` = 深圳证券交易所

**字段映射**：

| 字段 | 含义 |
|------|------|
| f2 | 最新价 |
| f3 | 涨跌幅(%) |
| f4 | 涨跌额 |
| f5 | 成交量(手) |
| f6 | 成交额(元) |
| f7 | 振幅(%) |
| f8 | 换手率(%) |
| f9 | PE(TTM) |
| f10 | 量比 |
| f12 | 证券代码 |
| f13 | 市场代码 |
| f14 | 证券名称 |
| f15 | 最高价 |
| f16 | 最低价 |
| f17 | 开盘价 |
| f18 | 昨收价 |
| f20 | 总市值(元) |
| f21 | 流通市值(元) |
| f23 | PB(市净率) |
| f24 | 60日涨跌幅 |
| f25 | 年初至今涨跌幅 |
| f100 | 所属行业 |

### 示例请求

```
https://push2.eastmoney.com/api/qt/ulist.np/get?fltt=2&fields=f2,f3,f9,f12,f14,f20,f23&secids=1.600011,1.600111,0.000400
```

### Python 解析

```python
import json, urllib.request

url = "https://push2.eastmoney.com/api/qt/ulist.np/get?fltt=2&fields=f2,f3,f9,f12,f14,f20,f23&secids=1.600011"
with urllib.request.urlopen(url) as resp:
    data = json.loads(resp.read())
    for s in data['data']['diff']:
        name = s['f14']
        price = s['f2']
        pe = s['f9']
        pb = s['f23']
        mcap = round(s['f20'] / 1e8, 1)  # 转换为亿元
        print(f"{name}: 股价={price}, PE={pe}, PB={pb}, 市值={mcap}亿")
```

## 注意事项

- 该接口免费无需认证，但可能有频率限制
- 数据为实时行情，非历史数据
- PE/PB 为 TTM（滚动12个月）口径
- 市值单位为元，需除以1亿转换为"亿元"
- 行业分类使用东方财富自有分类体系
