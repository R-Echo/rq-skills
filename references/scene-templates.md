# 搜索场景模板详细参考

> 供 Agent 在执行搜索时查阅具体 URL 构造方式

---

## 1. 🔧 代码搜索

### 场景识别
用户提问包含：代码、函数、库、API、报错、GitHub、npm、PyPI、bug、error、怎么实现

### URL 构造

```
# Google 限定技术站点（首选）
https://www.google.com/search?q=site:github.com+{keyword}
https://www.google.com/search?q=site:stackoverflow.com+{keyword}
https://www.google.com/search?q=site:docs.python.org+{keyword}

# DuckDuckGo Bangs 直跳（备选）
https://duckduckgo.com/html/?q=!gh+{keyword}
https://duckduckgo.com/html/?q=!so+{keyword}
https://duckduckgo.com/html/?q=!npm+{keyword}
https://duckduckgo.com/html/?q=!pypi+{keyword}
https://duckduckgo.com/html/?q=!mdn+{keyword}
https://duckduckgo.com/html/?q=!docker+{keyword}

# Brave（第三备选）
https://search.brave.com/search?q={keyword}+github
```

### 操作符技巧
- 报错搜索：`"完整报错信息"` 精确匹配
- 高 star 项目：`{keyword} stars:>1000`
- 限定语言：`{keyword} language:python`
- 找教程：`{keyword} tutorial`
- 找示例：`{keyword} example`

### 降级链
Google → DuckDuckGo → Brave

---

## 2. 📚 学术搜索

### 场景识别
用户提问包含：论文、研究、文献、学术、PDF、引用、journal、paper、review

### URL 构造

```
# Google Scholar（首选）
https://scholar.google.com/scholar?q={keyword}

# Google 限定 PDF（备选）
https://www.google.com/search?q={keyword}+filetype:pdf

# arXiv（备选）
https://duckduckgo.com/html/?q=site:arxiv.org+{keyword}

# 百度学术（国内备选）
https://www.baidu.com/s?wd={keyword}+filetype:pdf
```

### 操作符技巧
- 限定年份：`&tbs=cdr:1,cd_min:2024/1/1,cd_max:2024/12/31`
- 限定作者：`author:"Geoffrey Hinton"`
- 找综述：`{keyword} review` 或 `{keyword} survey`
- 找被引多的：Scholar 默认按引用排序

### 降级链
Google Scholar → Google (filetype:pdf) → 百度

---

## 3. 🛒 商品搜索

### 场景识别
用户提问包含：价格、便宜、哪里买、评测、推荐、比价、多少钱、值得买、性价比

### URL 构造

```
# 百度（首选，国内商品覆盖最好）
https://www.baidu.com/s?wd={keyword}+价格+评测

# Google 购物（备选）
https://www.google.com/search?q={keyword}&tbm=shop

# 价格范围
https://www.google.com/search?q={keyword}+%24500..%241000

# 今日头条（国产评测/开箱）
https://so.toutiao.com/search?keyword={keyword}+评测

# 集思录（理财/投资类商品）
https://www.jisilu.cn/explore/?keyword={keyword}
```

### 操作符技巧
- 价格范围：`$500..$1000` 或 `¥3000..¥5000`
- 找评测：`{keyword} 评测 对比`
- 找优惠：`{keyword} 优惠券 折扣`

### 降级链
百度 → Google Shopping → 今日头条

---

## 4. 📰 新闻搜索

### 场景识别
用户提问包含：新闻、最新、今天、最近、事件、动态、breaking、刚发生

### URL 构造

```
# Google News（首选，实时性最强）
https://www.google.com/search?q={keyword}&tbm=nws&tbs=qdr:d

# Brave News（备选，独立索引）
https://search.brave.com/search?q={keyword}&source=news&tf=pw

# 百度新闻（国内新闻）
https://www.baidu.com/s?wd={keyword}&rtt=4&bsst=1&cl=2&tn=news

# 今日头条（国产资讯）
https://so.toutiao.com/search?keyword={keyword}

# 微信公众号（行业深度文章）
https://wx.sogou.com/weixin?type=2&query={keyword}
```

### 时间过滤参数
| 参数 | 含义 | 适用引擎 |
|------|------|---------|
| `tbs=qdr:h` | 过去 1 小时 | Google |
| `tbs=qdr:d` | 过去 24 小时 | Google |
| `tbs=qdr:w` | 过去 1 周 | Google |
| `tbs=qdr:m` | 过去 1 月 | Google |
| `tf=pw` | 本周 | Brave |
| `tf=pm` | 本月 | Brave |
| `time=day` | 过去 24 小时 | Startpage |
| `time=week` | 过去 1 周 | Startpage |
| `rtt=4&bsst=1` | 按时间排序 | 百度 |

### 降级链
Google News → Brave News → 百度新闻

---

## 5. 👤 人物/公司搜索

### 场景识别
用户提问包含：某人是谁、某公司、背景、简历、经历、创始人、CEO

### URL 构造

```
# Google 精确搜索（首选）
https://www.google.com/search?q=%22{name}%22

# 百度（国内人物）
https://www.baidu.com/s?wd=%22{name}%22

# DuckDuckGo（隐私视角）
https://duckduckgo.com/html/?q=%22{name}%22

# LinkedIn 搜索
https://www.google.com/search?q=site:linkedin.com+%22{name}%22

# 公司背景
https://www.google.com/search?q=%22{company}%22+融资+创始人
```

### 操作符技巧
- 精确匹配姓名：`"张三"`
- 限定平台：`site:linkedin.com`、`site:zhihu.com`
- 组合搜索：`"张三" AND "清华"`
- 排除干扰：`"张三" -歌手 -演员`

### 降级链
Google → 百度 → DuckDuckGo

---

## 6. 🔒 隐私搜索

### 场景识别
用户提问包含：隐私、不想被追踪、匿名、敏感、安全搜索

### URL 构造

```
# DuckDuckGo（首选，零追踪）
https://duckduckgo.com/html/?q={keyword}

# Startpage（备选，Google 结果 + 隐私）
https://www.startpage.com/sp/search?query={keyword}

# Brave（备选，独立索引）
https://search.brave.com/search?q={keyword}

# Qwant（备选，欧盟合规）
https://www.qwant.com/?q={keyword}
```

### 隐私级别对比
| 引擎 | 追踪 | 数据保留 | 索引来源 |
|------|------|---------|---------|
| DuckDuckGo | 无 | 无 | Bing |
| Startpage | 无 | 无 | Google |
| Brave | 无 | 无 | 自有 |
| Qwant | 无 | 无 | 自有+Bing |

### 降级链
DuckDuckGo → Startpage → Brave

---

## 7. 📊 知识计算

### 场景识别
用户提问包含：计算、换算、汇率、公式、数据、统计、积分、导数

### URL 构造

```
# WolframAlpha（首选，必须用英文提问）
https://www.wolframalpha.com/input?i={english_query}

# Google（备选，简单计算可直接）
https://www.google.com/search?q={keyword}

# 百度（备选，中文计算）
https://www.baidu.com/s?wd={keyword}
```

### WolframAlpha 查询示例
- 数学：`integrate x^2 dx`
- 汇率：`100 USD to CNY`
- 股票：`AAPL stock`
- 天气：`weather in Beijing`
- 营养：`calories in banana`
- 化学：`molar mass of H2SO4`
- 物理：`speed of light`
- 日期：`days between Jan 1 2020 and Dec 31 2024`

**注意：** WolframAlpha 不支持中文查询，必须翻译为英文

### 降级链
WolframAlpha → Google → 百度

---

## 8. 🔍 综合搜索（默认）

### 场景识别
不匹配以上任何场景时使用

### URL 构造

```
# Google（首选）
https://www.google.com/search?q={keyword}

# 必应中国版（备选）
https://cn.bing.com/search?q={keyword}&ensearch=0

# Brave（备选）
https://search.brave.com/search?q={keyword}

# 百度（中文综合备选）
https://www.baidu.com/s?wd={keyword}
```

### 降级链
Google → 必应中国版 → Brave
