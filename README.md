# rq-skills

R-Echo / Rico 的公开技能集合。

## multi-search-engine v3.0

`multi-search-engine` 是一个智能搜索策略器：它会先识别用户意图，再选择适合的搜索场景模板，并通过多引擎降级链和结果聚合，输出可追溯的搜索结果。

### 能力

- 意图识别
- 8 类场景模板：代码、学术、商品、新闻、人物、隐私、计算、综合
- 多搜索引擎降级链
- 结果聚合与统一输出
- 零 API key：默认使用可用的公开搜索入口，不要求用户配置密钥

### 使用

将 `multi-search-engine` 目录安装到兼容 OpenClaw/SosoAgent 的 `skills/` 目录，保留目录结构，然后直接提出搜索需求。例如：

> 查一下 Windows Electron net.request 的 request close 行为，给出官方资料和可复现实验方法。

技能会根据问题类型选择对应策略，并保留来源链接，便于核验。

### 目录结构

```text
multi-search-engine/
├── SKILL.md
├── config.json
├── metadata.json
├── CHANGELOG.md
├── CHANNELLOG.md
├── references/
└── ...
```

### 隐私与限制

本 Skill 不要求 API key，但搜索是否联网取决于运行环境和所选搜索入口。使用云端搜索时，查询内容可能离开本机；敏感内容请先脱敏。零 API key 不等于零联网。

## License

See the repository contents and upstream project terms for applicable licensing.
