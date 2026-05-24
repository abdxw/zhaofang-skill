# 各房产平台 URL 规律

## 链家（lianjia.com）

| 页面类型 | URL格式 | 说明 |
|---------|---------|------|
| 小区详情页 | `https://bj.lianjia.com/xiaoqu/小区ID/` | **购房首选**，有参考均价、在售套数、成交记录 |
| 房源列表页 | `https://bj.lianjia.com/ershoufang/` | 会被反爬拦截，但搜索能获取摘要 |
| 成交记录 | `https://bj.lianjia.com/chengjiao/` | 可搜索近期的成交案例 |

**搜索方式**：
```bash
# 小区详情页（推荐）
site:lianjia.com/xiaoqu/ 小区名
小区名 链家 均价 二手房 2026
```

---

## 安居客（anjuke.com）

| 页面类型 | URL格式 | 说明 |
|---------|---------|------|
| 房源详情页 | `https://bj.zu.anjuke.com/fangyuan/{数字ID}/` | ✅ 可直接访问 |
| 租房列表页 | `https://bj.zu.anjuke.com/` | 筛选条件 |
| 二手房列表 | `https://bj.anjuke.com/sale/` | |

**搜索方式**：
```bash
# 房源详情页（推荐）
site:anjuke.com/fangyuan/ 城市 区域 户型 预算

# ⚠️ 注意：安居客房源大量过期，搜索结果需交叉验证
```

---

## 58同城（58.com）

| 页面类型 | URL格式 | 说明 |
|---------|---------|------|
| 租房列表 | `https://bj.58.com/zufang/` | 筛选条件 |
| 二手房列表 | `https://bj.58.com/ershoufang/` | |

**搜索方式**：
```bash
# 房源详情页
site:58.com/zufang/ 城市 区域
```

---

## 房天下（fang.com）

| 页面类型 | URL格式 | 说明 |
|---------|---------|------|
| 城市站 | `https://bj.fang.com/` | |
| 房源列表 | `https://bj.zu.fang.com/` | |

**注意**：WSL环境下 `fang.com` DNS 失败，浏览器也无法访问。需换其他平台。

---

## 贝壳找房（beike.com）

| 页面类型 | URL格式 | 说明 |
|---------|---------|------|
| 小区详情 | `https://www.beike.com/xiaoqu/` | 与链家数据打通 |
| 成交记录 | `https://www.beike.com/chengjiao/` | |

**注意**：主站 `beike.com` SSL证书有问题，链家 `lianjia.com` 为主。

---

## 搜索词模板

### 购房搜索
```bash
# 小区详情（推荐）
site:lianjia.com/xiaoqu/ 小区名
小区名 链家 均价 二手房 2026

# 板块分析
site:lianjia.com/xiaoqu/ 板块名
板块名 链家 均价 二手房 2026

# 成交案例
小区名 成交 2025 链家
```

### 租房搜索
```bash
# 安居客房源
site:anjuke.com/fangyuan/ 城市 区域 户型 预算

# 58同城
site:58.com/zufang/ 城市 区域

# 小红书真实体验（辅助）
城市 区域 小区 租房 真实体验 小红书
```