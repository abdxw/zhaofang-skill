# 找房助手（通用版）

> 适用于 AI Agent（Hermes Agent）一键找房：租房/购房场景，支持北京/上海/广州/深圳等城市。

---

## 功能概述

- 整合房源信息、计算通勤、比对优劣势、生成攻略
- 支持租房和购房两种场景
- 适用城市：北京、上海、广州、深圳、成都、杭州等

---

## 使用前提

### 环境变量

```bash
# 必须配置
TAVILY_API_KEY=你的Tavily搜索API密钥

# 推荐配置（如需通勤计算）
GAODE_KEY=你的高德Web服务Key
AMAP_KEY=同上（部分环境用此变量名）
```

### 获取方式

- **Tavily API**：https://tavily.com 注册后获取，Free plan 足够日常使用
- **高德 Web 服务 Key**：https://lbs.amap.com 控制台 → 添加Key → 应用类型选「Web服务」

---

## 快速开始

用户只需说一句话，例如：

> "帮我找北京海淀牡丹园附近一居室，预算6000左右，在中关村上班"

AI Agent 自动解析：城市+区域+户型+预算+工作地点+通勤方式，然后立即开始搜索，不再反复询问。

---

## 数据源优先级（重要）

购房数据必须用权威来源验证，禁止直接呈现未经核实的搜索结果。

| 优先级 | 来源 | 获取方式 |
|--------|------|----------|
| 1 | **链家/贝壳小区详情页**（xiaoqu/） | `小区名 链家 均价 二手房 2026` 搜索，精确可靠 ✅ |
| 2 | 贝壳研究院/克而瑞报告 | `贝壳研究院 {板块} 报告` 搜索，有滞后但权威 |
| 3 | 安居客小区详情页 | 交叉验证用，不作为主要来源 |

### 购房数据搜索的正确方式

```bash
# ✅ 正确：搜链家小区详情页，获取参考均价+在售套数+成交记录
# 搜索词格式：`小区名 链家 均价 二手房 2026`
# 示例：`马家堡西里 链家 均价 二手房 2026` → 返回：均价¥43,258/㎡，在售53套
# 示例：`角门东里 链家 均价 二手房 2026` → 返回：均价¥42,713/㎡，在售57套

# ✅ 正确：用 site:lianjia.com/xiaoqu/ 限定小区详情页
# 示例：`site:lianjia.com/xiaoqu/ 小区名`

# ❌ 错误：跨区域混合搜索（容易混入不同板块数据）
# 示例：`南六环 二手房 400万 两居室` → 可能返回多个不相关板块数据

# ⚠️ 原则：跨板块搜索必须分区逐个查，合并时再次验证
```

### 租房数据搜索方式

```bash
# 搜索安居客房源详情页（注意过期问题）
node search.mjs "城市 区域 户型 预算 site:anjuke.com/fangyuan/" --deep

# 搜索58同城
node search.mjs "城市 区域 户型 预算 site:58.com" --deep

# 小红书真实体验（可补充验证）
node search.mjs "城市 区域 租房 小红书 真实体验 避坑" --deep
```

---

## 工具环境限制

| 工具 | 状态 | 说明 |
|------|------|------|
| Tavily 搜索 | ✅ 可用 | 需配置 TAVILY_API_KEY |
| 高德 API v3 | ✅ 需配置Key | 通勤时间计算，驾车/公交均可 |
| Browser/Playwright | ❌ 未装 | 无法渲染 JS 动态页面 |
| 直接 curl 抓房源平台 | ❌ 返回空 | 反爬限制 |

---

## 工作流程

### Step 1：解析需求

用户发来需求 → 直接解析以下字段：
- 城市 + 区域
- 户型（1室/2室1厅等）
- 预算（月租 or 总价）
- 工作/上学地点
- 能接受的最长通勤时间

### Step 2：并行搜索

| 任务 | 数据来源 | 搜索词示例 |
|------|----------|-----------|
| 小区数据 | 链家xiaoqu/ | `XX小区 链家 均价 二手房 2026` |
| 房源 | 安居客+58 | `城市 区域 户型 预算 site:anjuke.com/fangyuan/` |
| 通勤 | 高德API v3 | 通过 `restapi.amap.com/v3/direction/transit` 查询 |
| 避坑 | 小红书 | `城市 区域 小区 避坑 真实体验` |

### Step 3：生成比对报告

```markdown
【{城市}{区域} {户型} 租房比对】
数据获取时间：{日期} | 来源：安居客 + 链家

小区 / 租金 / 面积 / 距地铁 / 楼层 / 通勤 / 综合评分

📍 总结：通勤最优 / 性价比最优 / 平衡之选
💡 建议：优先看哪个，为什么
```

### Step 4：用户截图验证

推荐用户截图发来（App筛选后的实时结果），AI Agent 做深度分析。

---

## 通勤计算说明

### 高德 API 调用（需 Web服务Key）

```python
import subprocess, json

key = os.environ.get("AMAP_KEY") or os.environ.get("GAODE_KEY")

def get_driving(lon1, lat1, lon2, lat2):
    url = f"https://restapi.amap.com/v3/direction/driving?key={key}&origin={lon1},{lat1}&destination={lon2},{lat2}"
    r = subprocess.run(['curl', '-s', '--max-time', '8', url], capture_output=True, text=True)
    try:
        d = json.loads(r.stdout)
        if d.get('route', {}).get('paths'):
            p = d['route']['paths'][0]
            return int(p['duration']) // 60, int(p['distance']) // 1000  # 分钟, km
    except:
        pass
    return None, None

def get_transit(lon1, lat1, lon2, lat2):
    url = f"https://restapi.amap.com/v3/direction/transit/integrated?key={key}&origin={lon1},{lat1}&destination={lon2},{lat2}&city=北京&strategy=0"
    r = subprocess.run(['curl', '-s', '--max-time', '10', url], capture_output=True, text=True)
    try:
        d = json.loads(r.stdout)
        if d.get('route', {}).get('transits'):
            t = d['route']['transits'][0]
            return int(t['duration']) // 60, int(t['distance']) // 1000
    except:
        pass
    return None, None
```

### 关键注意事项

1. **必须用 v3 版本**（`/v3/direction/driving`），v5 不返回 duration 字段
2. **高德 Key 类型**：移动端Key不能用于HTTP请求，必须申请Web服务Key
3. **Python urllib 问题**：WSL环境直接用urllib会报UnicodeEncodeError，**用subprocess调用curl**
4. **坐标获取**：用 `restapi.amap.com/v3/place/text` 查询地点坐标
5. **不能用 haversine 直线距离代替实际通勤时间**

---

## 参考文件

| 文件 | 内容 |
|------|------|
| references/data-source-status.md | 各房产平台数据质量实测汇总 |
| references/block-analysis-workflow.md | 板块分析报告工作流 |
| references/checklist.md | 看房/签约检查清单 |
| references/evaluation.md | 房源多维评分体系 |
| references/purchase.md | 购房全流程与税费计算 |
| references/fang-url-pattern.md | 各平台URL规律 |
| references/commute-guide.md | 通勤计算通用指南 |

---

## 已知限制

1. **实时房源数据没有免费方案**：贝壳/链家/自如API封闭，安居客/58数据不实时
2. **安居客房源过期问题**：Tavily搜索返回的是历史索引，不是实时数据；缓解方案：用户截图验证
3. **高德API Key必须为Web服务类型**：移动端Key（Android/iOS SDK）无法用于HTTP接口

---

## 上传说明

本 skill 依赖 Hermes Agent 框架运行。

如只需内容参考，不需要完整框架：各 reference 文件可独立使用，提取其中方法论即可。