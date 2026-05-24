# 通勤计算通用指南

> 适用于使用高德 Web 服务 API 计算真实通勤时间。支持驾车/公交/步行等多种方式。

---

## 高德 API 基础知识

### Key 类型区分（重要）

高德开放平台**同一账号下有两类Key**：

| Key类型 | 用途 | 用于 HTTP 接口 |
|--------|------|-------------|
| 移动端Key | Android/iOS SDK | ❌ 返回 `USERKEY_PLAT_NOMATCH (10009)` |
| **Web服务Key** | Web API | ✅ 正确 |

**申请方式**：控制台 → 添加Key → 应用类型选「Web服务」→ 勾选「Web服务API」

---

### API 版本区分

| 版本 | 端点 | 返回字段 | 适用场景 |
|------|------|---------|---------|
| **v3** | `/v3/direction/driving` | duration + distance | ✅ 推荐 |
| v5 | `/v5/direction/driving` | 仅 distance | ❌ 不推荐 |

---

## 基础 API 调用

### 1. 地址坐标查询（place/text）

```python
import subprocess, json, urllib.parse

key = os.environ.get("AMAP_KEY") or os.environ.get("GAODE_KEY")

def get_coordinates(location_name):
    """查询地点坐标"""
    encoded = urllib.parse.quote(location_name)
    url = f"https://restapi.amap.com/v3/place/text?key={key}&keywords={encoded}&city=北京"
    r = subprocess.run(['curl', '-s', '--max-time', '8', url], capture_output=True, text=True)
    try:
        d = json.loads(r.stdout)
        if d.get('pois') and len(d['pois']) > 0:
            loc = d['pois'][0]['location']
            return loc  # "经度,纬度" 格式
    except:
        pass
    return None
```

### 2. 驾车路线（driving）

```python
def get_driving_time(lon1, lat1, lon2, lat2):
    """驾车时间计算（返回分钟数）"""
    url = f"https://restapi.amap.com/v3/direction/driving?key={key}&origin={lon1},{lat1}&destination={lon2},{lat2}&strategy=4"
    r = subprocess.run(['curl', '-s', '--max-time', '8', url], capture_output=True, text=True)
    try:
        d = json.loads(r.stdout)
        if d.get('route', {}).get('paths'):
            p = d['route']['paths'][0]
            return int(p['duration']) // 60, int(p['distance']) // 1000  # 分钟, km
    except:
        pass
    return None, None

# 示例
minutes, km = get_driving_time("116.3708,39.8390", "116.3535,39.9563")  # 公益西桥→学院桥
print(f"{minutes}分钟({km}km)")
```

### 3. 公共交通路线（transit）

```python
def get_transit_time(lon1, lat1, lon2, lat2, city="北京"):
    """公共交通时间计算（返回分钟数）"""
    url = f"https://restapi.amap.com/v3/direction/transit/integrated?key={key}&origin={lon1},{lat1}&destination={lon2},{lat2}&city={city}&strategy=0"
    r = subprocess.run(['curl', '-s', '--max-time', '10', url], capture_output=True, text=True)
    try:
        d = json.loads(r.stdout)
        if d.get('route', {}).get('transits'):
            t = d['route']['transits'][0]
            return int(t['duration']) // 60, int(t['distance']) // 1000
    except:
        pass
    return None, None

# 示例
minutes, km = get_transit_time("116.3708,39.8390", "116.3535,39.9563")
print(f"{minutes}分钟({km}km)")
```

---

## 常见问题排查

| 错误现象 | 原因 | 解决方案 |
|---------|------|---------|
| `USERKEY_PLAT_NOMATCH (10009)` | 用了移动端Key | 重新申请Web服务Key |
| `SERVICE_NOT_AVAILABLE (10002)` | 接口不可用 | 改用 `place/text` 搜索 |
| `KeyError: 'duration'` | 用了v5 API | 改用 v3 API（`/v3/direction/driving`） |
| `UnicodeEncodeError` | Python urllib直接发中文URL | 用subprocess调用curl |
| `json.loads` 返回空 | curl超时或网络问题 | 加 `--max-time 8` 和超时处理 |

---

## 注意事项

1. **不能用 haversine 直线距离代替实际通勤时间**：北京实际通勤通常是直线距离的 1.5-2.5 倍
2. **不能用 OSRM/free-flow 数据答复用户**：OSRM 返回的是理论时间，不代表真实路况
3. **公交策略选择**：`strategy=0` 为最省时间，适合通勤；`strategy=1` 为最少换乘
4. **API 配额**：免费版有日配额限制，高频使用需关注配额

---

## 调用示例：两地通勤评估

```python
import subprocess, json, os

AMAP_KEY = os.environ.get("AMAP_KEY") or os.environ.get("GAODE_KEY")

# 假设用户说：在国贸上班，想租角门西/公益西桥的房子
# 先查坐标
origin = "国贸"  # 经度纬度需通过 place/text 查询
destination = "角门西"  # 同上

# 然后计算驾车和公交时间
# ...

# 输出结果
print("从 国贸 到 角门西：")
print(f"  驾车：{driving_min}分钟 ({driving_km}km)")
print(f"  公交：{transit_min}分钟 ({transit_km}km)")
```

---

## 环境变量配置

```bash
# .env 或 shell profile
export AMAP_KEY="你的高德Web服务Key"
export GAODE_KEY="同上"
export TAVILY_API_KEY="你的Tavily Key"
```

AI Agent 会自动读取这些环境变量。