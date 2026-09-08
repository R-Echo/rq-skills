---
name: "multi-search-engine"
version: "3.0.0"
description: "智能多引擎搜索策略器。17 搜索引擎 + 意图识别 + 场景模板 + 降级链 + 结果聚合。零 API key，开箱即用。"
---

# Multi Search Engine v3.0 — 智能搜索策略器

> 不只是 URL 列表。帮你选引擎、选策略、选时间范围，聚合结果，处理失败。

## 核心能力

1. **意图识别** — 根据用户提问自动识别搜索场景，选择最佳引擎组合
2. **场景模板** — 8 种内置场景（代码/学术/商品/新闻/人物/隐私/知识/综合），一键启动
3. **多引擎聚合** — 一次搜索查 2-3 个引擎，去重合并，交叉验证
4. **智能降级** — 首选引擎失败时自动切换备选，不中断搜索
5. **反封策略** — 请求间隔、URL 变体、失败重试，最大化成功率
6. **零 API key** — 全部引擎走公开搜索页面，无需注册

---

## 搜索引擎清单

### 国内（8 个）

| 引擎 | 擅长 | 搜索 URL |
|------|------|----------|
| **百度** | 中文综合 | `https://www.baidu.com/s?wd={keyword}` |
| **必应中国版** | 中文+英文 | `https://cn.bing.com/search?q={keyword}&ensearch=0` |
| **必应国际版** | 英文为主 | `https://cn.bing.com/search?q={keyword}&ensearch=1` |
| **360 搜索** | 中文备选 | `https://www.so.com/s?q={keyword}` |
| **搜狗** | 中文备选 | `https://sogou.com/web?query={keyword}` |
| **微信搜一搜** | 公众号文章 | `https://wx.sogou.com/weixin?type=2&query={keyword}` |
| **今日头条** | 中文资讯 | `https://so.toutiao.com/search?keyword={keyword}` |
| **集思录** | 投资理财 | `https://www.jisilu.cn/explore/?keyword={keyword}` |

### 国际（9 个）

| 引擎 | 擅长 | 搜索 URL |
|------|------|----------|
| **Google** | 全球综合最强 | `https://www.google.com/search?q={keyword}` |
| **Google HK** | 中文+国际 | `https://www.google.com.hk/search?q={keyword}` |
| **DuckDuckGo** | 隐私+Bangs 跳转 | `https://duckduckgo.com/html/?q={keyword}` |
| **Yahoo** | 综合备选 | `https://search.yahoo.com/search?p={keyword}` |
| **Startpage** | Google 结果+隐私 | `https://www.startpage.com/sp/search?query={keyword}` |
| **Brave** | 独立索引+新闻 | `https://search.brave.com/search?q={keyword}` |
| **Ecosia** | 环保搜索 | `https://www.ecosia.org/search?q={keyword}` |
| **Qwant** | 欧盟合规 | `https://www.qwant.com/?q={keyword}` |
| **WolframAlpha** | 知识计算 | `https://www.wolframalpha.com/input?i={keyword}` |

---

## 搜索流程（Agent 必读）

当用户要求搜索时，按以下流程执行：

### Step 1：识别意图

根据用户输入判断搜索场景：

| 用户意图关键词 | 场景 | 首选引擎 | 备选引擎 |
|---------------|------|---------|---------|
| 代码/函数/库/API/报错/GitHub | 🔧 代码搜索 | Google (`site:`限定) | DuckDuckGo (`!gh`/`!so` bangs) |
| 论文/研究/文献/PDF/学术 | 📚 学术搜索 | Google Scholar | Google (`filetype:pdf`) |
| 价格/便宜/哪里买/评测/比价 | 🛒 商品搜索 | 百度 | 今日头条 + Google |
| 新闻/最新/今天/最近/事件 | 📰 新闻搜索 | Google News | Brave News + 百度 |
| 某个人/公司/组织/背景 | 👤 人物搜索 | Google | 百度 + DuckDuckGo |
| 隐私/敏感/不想被追踪 | 🔒 隐私搜索 | DuckDuckGo | Startpage + Brave |
| 计算/换算/汇率/公式/数据 | 📊 计算 | WolframAlpha | Google |
| 其他/通用 | 🔍 综合搜索 | Google | 必应中国版 + Brave |

### Step 2：构造查询

根据场景自动添加：
- **高级操作符**（`site:`, `filetype:`, `""`, `-`）
- **时间过滤**（`tbs=qdr:w` 等）
- **URL 编码**（中文和特殊字符必须编码）

### Step 3：执行搜索

```
首选引擎 → 成功则提取结果
         → 失败（403/超时/空结果）→ 自动切备选引擎
         → 仍失败 → 再切下一个
         → 全失败 → 告知用户，建议换关键词或手动搜索
```

**关键规则：**
- 每次 `web_fetch` 设置 `maxChars: 15000`，避免截断不够或内容过长
- 两次请求之间**不刻意等待**（Agent 调用本身就是间隔的）
- 如果首选引擎返回 403/captcha/空白，**立即**切换备选，不要重试同一引擎
- 聚合搜索时最多同时查 **3 个引擎**，避免浪费 token

### Step 4：整理结果

- 去重（同一来源出现在不同引擎的，只保留一条）
- 按相关性排序
- 标注来源引擎
- 提取标题、摘要、链接

---

## 场景模板详解

### 🔧 代码搜索

**触发词：** 代码、函数、库、API、报错信息、GitHub、npm、PyPI

```javascript
// 首选：Google 限定技术站点
web_fetch({url: "https://www.google.com/search?q=site:github.com+{keyword}"})
web_fetch({url: "https://www.google.com/search?q=site:stackoverflow.com+{keyword}"})

// 备选：DuckDuckGo Bangs 直跳
web_fetch({url: "https://duckduckgo.com/html/?q=!gh+{keyword}"})    // GitHub
web_fetch({url: "https://duckduckgo.com/html/?q=!so+{keyword}"})    // Stack Overflow
web_fetch({url: "https://duckduckgo.com/html/?q=!npm+{keyword}"})   // npm
web_fetch({url: "https://duckduckgo.com/html/?q=!pypi+{keyword}"})  // PyPI
web_fetch({url: "https://duckduckgo.com/html/?q=!mdn+{keyword}"})   // MDN 文档
web_fetch({url: "https://duckduckgo.com/html/?q=!docker+{keyword}"}) // Docker Hub
```

**技巧：**
- 报错搜索：把完整报错信息用 `""` 包裹精确匹配
- 找高 star 项目：加 `stars:>1000`
- 找特定语言：加 `language:python`

### 📚 学术搜索

**触发词：** 论文、研究、文献、学术、PDF、引用

```javascript
// 首选：Google Scholar
web_fetch({url: "https://scholar.google.com/scholar?q={keyword}"})

// 备选：Google 限定 PDF
web_fetch({url: "https://www.google.com/search?q={keyword}+filetype:pdf"})

// 备选：arXiv
web_fetch({url: "https://duckduckgo.com/html/?q=site:arxiv.org+{keyword}"})
```

**技巧：**
- 限定年份：`&tbs=cdr:1,cd_min:1/1/2024,cd_max:12/31/2024`
- 限定作者：`author:"Geoffrey Hinton"`
- 找引用多的：Scholar 默认按引用数排序

### 🛒 商品搜索

**触发词：** 价格、便宜、哪里买、评测、推荐、比价、多少钱

```javascript
// 首选：百度（国内商品覆盖最好）
web_fetch({url: "https://www.baidu.com/s?wd={keyword}+价格+评测"})

// 备选：Google 购物
web_fetch({url: "https://www.google.com/search?q={keyword}&tbm=shop"})

// 价格范围搜索
web_fetch({url: "https://www.google.com/search?q={keyword}+$500..$1000"})
```

### 📰 新闻搜索

**触发词：** 新闻、最新、今天、最近、事件、动态

```javascript
// 首选：Google News（实时性最强）
web_fetch({url: "https://www.google.com/search?q={keyword}&tbm=nws&tbs=qdr:d"})

// 备选：Brave News（独立索引）
web_fetch({url: "https://search.brave.com/search?q={keyword}&source=news&tf=pw"})

// 国内新闻
web_fetch({url: "https://so.toutiao.com/search?keyword={keyword}"})
web_fetch({url: "https://www.baidu.com/s?wd={keyword}&rtt=4&bsst=1"})
```

**时间过滤：**
- 过去 1 小时：`tbs=qdr:h`
- 过去 24 小时：`tbs=qdr:d`
- 过去 1 周：`tbs=qdr:w`
- 过去 1 月：`tbs=qdr:m`

### 👤 人物/公司搜索

**触发词：** 某人是谁、某公司、背景、简历、经历

```javascript
// 首选：Google 精确搜索
web_fetch({url: "https://www.google.com/search?q=%22{name}%22"})

// 备选：百度（国内人物）
web_fetch({url: "https://www.baidu.com/s?wd=%22{name}%22"})

// 隐私视角：DuckDuckGo
web_fetch({url: "https://duckduckgo.com/html/?q=%22{name}%22"})
```

### 🔒 隐私搜索

**触发词：** 隐私、不想被追踪、匿名、敏感

```javascript
// 首选：DuckDuckGo（零追踪）
web_fetch({url: "https://duckduckgo.com/html/?q={keyword}"})

// 备选：Startpage（Google 结果 + 隐私保护）
web_fetch({url: "https://www.startpage.com/sp/search?query={keyword}"})

// 备选：Brave（独立索引 + 无追踪）
web_fetch({url: "https://search.brave.com/search?q={keyword}"})
```

### 📊 知识计算

**触发词：** 计算、换算、汇率、公式、数据、统计

```javascript
// 首选：WolframAlpha
web_fetch({url: "https://www.wolframalpha.com/input?i={query}"})

// 支持：数学、单位换算、汇率、股票、天气、营养、化学、物理
// 示例：100 USD to CNY / integrate x^2 dx / AAPL stock
```

### 🔍 综合搜索（默认）

**触发词：** 不匹配以上任何场景时

```javascript
// 首选：Google
web_fetch({url: "https://www.google.com/search?q={keyword}"})

// 备选：必应中国版
web_fetch({url: "https://cn.bing.com/search?q={keyword}&ensearch=0"})

// 备选：Brave
web_fetch({url: "https://search.brave.com/search?q={keyword}"})
```

---

## 高级操作符速查

| 操作符 | 功能 | 示例 |
|--------|------|------|
| `""` | 精确匹配 | `"machine learning"` |
| `-` | 排除关键词 | `python -snake` |
| `site:` | 限定站点 | `site:github.com` |
| `filetype:` | 文件类型 | `filetype:pdf` |
| `intitle:` | 标题包含 | `intitle:"index of"` |
| `inurl:` | URL 包含 | `inurl:login` |
| `OR` | 或运算 | `cat OR dog` |
| `*` | 通配符 | `machine * algorithms` |
| `$min..$max` | 价格范围 | `laptop $500..$1000` |
| `related:` | 相关网站 | `related:github.com` |

---

## 降级链配置

每个场景的降级顺序（首选 → 备选1 → 备选2）：

| 场景 | 首选 | 备选 1 | 备选 2 |
|------|------|--------|--------|
| 代码 | Google (site:) | DuckDuckGo (bangs) | Brave |
| 学术 | Google Scholar | Google (filetype:pdf) | 百度 |
| 商品 | 百度 | Google Shopping | 今日头条 |
| 新闻 | Google News | Brave News | 百度新闻 |
| 人物 | Google ("") | 百度 ("") | DuckDuckGo |
| 隐私 | DuckDuckGo | Startpage | Brave |
| 计算 | WolframAlpha | Google | 百度 |
| 综合 | Google | 必应中国版 | Brave |

**降级触发条件：**
- HTTP 403 / 429（被封/限流）
- 返回内容为空或只有 captcha
- `web_fetch` 超时
- 返回内容与搜索关键词完全无关

---

## DuckDuckGo Bangs 速查

| Bang | 跳转 | 用途 |
|------|------|------|
| `!g` | Google | 通用搜索 |
| `!gh` | GitHub | 项目搜索 |
| `!so` | Stack Overflow | 编程问答 |
| `!w` | Wikipedia | 百科 |
| `!yt` | YouTube | 视频 |
| `!a` | Amazon | 购物 |
| `!npm` | npm | Node 包 |
| `!pypi` | PyPI | Python 包 |
| `!mdn` | MDN | Web 文档 |
| `!docker` | Docker Hub | 容器镜像 |
| `!m` | Google Maps | 地图 |
| `!imdb` | IMDb | 电影 |

---

## 注意事项

1. **web_fetch 优先于 web_search** — 本 Skill 的引擎路由比内置 `web_search` 更精准
2. **中文搜索用中文关键词** — 不要把用户中文翻译成英文去搜百度
3. **URL 编码** — 中文关键词和特殊字符必须 `encodeURIComponent`
4. **maxChars 建议 15000** — 太小会截断关键结果，太大会浪费 token
5. **不要一次搜 10 个引擎** — 聚合搜索最多 3 个，否则 token 成本太高
6. **WolframAlpha 用英文提问** — 它不支持中文查询
7. **微信公众号搜索** — 用搜狗微信版（`wx.sogou.com`），不是通用搜索

---

## 版本历史

- v3.0.0 (2026-09-08) — 从 URL 列表升级为智能搜索策略器：意图识别 + 场景模板 + 降级链 + 聚合搜索
- v2.0.1 (2026-02-06) — 17 引擎 + 高级操作符 + 时间过滤
- v1.0.0 (2026-02-04) — 初始版本，8 个国内引擎
