---
description: 向 erp-w.com 站点新增一篇文章（中英双语）：创建 EN+ZH 文章页并同步更新栏目页、总列表、首页和 sitemap
---

在 erp-w.com 站点（当前仓库）新增一篇文章。**本站为中英双语站**：一篇文章 = 英文页 + 中文页 + 4 个列表页 + 2 个首页 + sitemap，约 10 个文件。完整规范见 `docs/adding-articles.md`。

## 输入

用户输入：$ARGUMENTS（格式：`文章标题 [栏目]`，栏目为 arch / finance / prod / workflow / integration / ui 之一）。

标题或栏目缺失、栏目名不合法时，先询问用户确认，不要猜栏目归属。

## 分页设计约定（执行前必读）

本站列表页采用**锚点分页**，硬规则：

1. 多个 `<div class="page-group" id="page-N">`，N 从 1 连续编号，页码链接 `#page-N`
2. **所有分组同时展示**，不用 `:target` 隐藏
3. 每组 ≤ 6 卡；新卡插组顶，满 7 张把该组最后一张移入下一组，级联处理后续组
4. **每个 page-group 紧跟一个 `.pager`**，页码覆盖全部组，当前组 `<span class="current">K</span>`
5. 首页不分页，「最新文章」固定 6 张

## 栏目映射（slug 中英共用）

| 目录 | EN 名 | ZH 名 |
|------|-------|-------|
| arch | Architecture | 架构与选型 |
| finance | Finance | 财务模块 |
| prod | Manufacturing & MRP | 生产与 MRP |
| workflow | Workflow | 工作流审批 |
| integration | Integration | 系统集成 |
| ui | Enterprise UI | 企业级界面 |

## 执行步骤

### 第 1 步：元信息
- slug：2–4 个英文语义词、小写连字符；确认 `/articles/` 与 `/zh/articles/` 均不存在同名文件
- 日期：今天（YYYY-MM-DD）
- 双语标题：给出 EN 与 ZH 标题（ZH 标题供中文页使用）

### 第 2 步：创建英文文章页 `/articles/<slug>.html`
以 `articles/modular-monolith.html` 为模板：
1. 正文 ≥ 600 词，内容真实具体（结合 Web 版 ERP 场景），禁止空话
2. SEO：title `标题 - erp-w.com`（≤60 字符）、description ≤155 字符、canonical 与 og:url 指本页、og:type=article、Twitter summary
3. hreflang 三链：en→本页、zh-CN→`https://erp-w.com/zh/articles/<slug>.html`、x-default→本页
4. JSON-LD：`Article` + `BreadcrumbList`
5. header 导航含 `<a class="lang-switch" href="/zh/articles/<slug>.html" hreflang="zh-CN">中文</a>`
6. sidebar、footer 与现有页面逐字一致；正文 1–2 处链同语言站内文章；文末 article-nav 接本栏目相邻文章

### 第 3 步：创建中文文章页 `/zh/articles/<slug>.html`
同 slug 对译版本：
1. 正文 ≥ 800 字；lang="zh-CN"；中文导航/面包屑/sidebar/footer
2. meta description 用中文；og:locale=zh_CN；JSON-LD inLanguage=zh-CN
3. hreflang 镜像（en 链英文页，x-default 链英文页）
4. lang-switch 改为 `<a class="lang-switch" href="/articles/<slug>.html" hreflang="en">English</a>`
5. 正文内链链中文站同语言文章；article-meta 用 `发布时间：日期 | 分类：栏目中文名`

### 第 4 步：更新英文栏目页 `/articles/<栏目>/index.html`
顶部插卡、计数 +1、满组按分页约定级联；每 group 后 pager 齐全。

### 第 5 步：更新中文栏目页 `/zh/articles/<栏目>/index.html`
同步第 4 步（卡片用中文标题/摘要/meta）。

### 第 6 步：更新英文总列表 `/articles/index.html`
顶部插卡、计数 +1、满组级联。

### 第 7 步：更新中文总列表 `/zh/articles/index.html`
同步第 6 步。

### 第 8 步：更新首页
- `/index.html`「最新文章」顶部插卡保持 6 张（EN 卡片）
- `/zh/index.html` 同步（ZH 卡片）

### 第 9 步：更新 `sitemap.xml`
`</urlset>` 前插入：
```xml
<url><loc>https://erp-w.com/articles/<slug>.html</loc><lastmod>日期</lastmod><priority>0.7</priority></url>
<url><loc>https://erp-w.com/zh/articles/<slug>.html</loc><lastmod>日期</lastmod><priority>0.6</priority></url>
```

### 第 10 步：校验（全过才算完成）
node -e 校验（勿用 bash 变量循环）：
1. 新 EN/ZH 页 title/description/canonical/h1 各恰好 1 个；所有 JSON-LD 可 JSON.parse
2. 全站站内链接 0 缺失（目录链接补 index.html 后查存在）
3. 分页结构：每 group ≤6 卡且后紧跟 pager；编号连续
4. 中英互查：两版卡片计数一致、lang-switch 互指正确、hreflang 三链齐全

### 第 11 步：汇报
报告：两个新页面 URL、改动文件清单、校验结果，并给出建议的 commit 命令（不自动 git 提交，除非用户要求）。
