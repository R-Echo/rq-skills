# Changelog

## v3.0.0 (2026-09-08)
- **重大升级：从 URL 列表升级为智能搜索策略器**
- 新增意图识别：根据用户提问自动匹配 8 种搜索场景
- 新增场景模板：代码 / 学术 / 商品 / 新闻 / 人物 / 隐私 / 计算 / 综合
- 新增降级链：每个场景配置首选→备选引擎，失败自动切换
- 新增聚合搜索策略：最多 3 引擎交叉验证，去重合并
- 新增反封策略指导：403/captcha/空结果的识别与处理
- 重写 SKILL.md 为面向 Agent 的执行手册（不再是人类阅读文档）
- 更新 config.json 增加场景路由配置

## v2.0.1 (2026-02-06)
- Simplified documentation
- Removed gov-related content
- Optimized for ClawHub publishing

## v2.0.0 (2026-02-06)
- Added 9 international search engines
- Enhanced advanced search capabilities
- Added DuckDuckGo Bangs support
- Added WolframAlpha knowledge queries

## v1.0.0 (2026-02-04)
- Initial release with 8 domestic search engines
