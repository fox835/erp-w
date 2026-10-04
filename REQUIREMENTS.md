# erp-w.com 站点建设要求（v1 · 定稿）

> 姊妹站：jxc-w.com（同为双语技术站，模板同源；参考站群 jxc.js.cn / psi.js.cn / ims.js.cn）
> 联系邮箱：webnic@qq.com

## 1. 定位与部署

- **主题**：ERP Web 版技术站——浏览器端 ERP 系统的工程实战（与 jxc-w.com 错位：那边偏进销存 SMB 场景，本站偏企业级 ERP 全栈）
- **语言**：**中英双语，默认英文**。英文树在根路径 `/`，中文树在 `/zh/` 子树；每篇文章两版并存
- **托管**：GitHub Pages（仓库根即站点根，`CNAME` 为 `erp-w.com`，强制 HTTPS，国外服务器，无 ICP 备案）
- **性质**：纯静态站，无后端、无数据库、全站零 JavaScript
- **联系邮箱**：`webnic@qq.com`（footer 与 about 页）

## 2. 技术约束

| 项目 | 要求 |
|------|------|
| 形态 | 纯静态 HTML + 单一样式表 `/css/style.css`，零 JS（checkbox 汉堡菜单、锚点分页） |
| 主题色 | teal：`--primary: #0d9488; --primary-dark: #0f766e`；hero 渐变 `#0d9488 → #0369a1` |
| 性能 | 单页 < 50KB（不含 CSS），无外链字体/脚本/图片 |
| 响应式 | 桌面/平板/手机三端，860px 以下汉堡菜单 |
| 双语 | hreflang 三链（en / zh-CN / x-default→英文）+ header `.lang-switch` 语言切换链接 |
| SEO | 见第 5 章（核心章节） |

## 3. 页面结构（双语两棵树）

```
/                      英文首页
/articles/             英文全部文章列表
/articles/<栏目>/      英文栏目页（目录式 URL）
/articles/<slug>.html  英文文章页
/about.html            英文关于
/zh/                   中文首页
/zh/articles/          中文全部文章列表
/zh/articles/<栏目>/   中文栏目页
/zh/articles/<slug>.html 中文文章页
/zh/about.html         中文关于
/404.html              自定义 404（双语单页）
/robots.txt /sitemap.xml /css/style.css
```

## 4. 全站模板骨架

1. **header**：logo + checkbox 汉堡 + 主导航（9 项）+ `.lang-switch`（EN 页显示"中文"链向 /zh/ 对应页，ZH 页显示"English"链向 / 对应页）
2. **layout**：`.sidebar`（栏目列表 / 热门文章 3 篇 / 关于本站）+ `.content`
3. **footer**：三栏（站点简介 / 栏目导航 / 相关站点互链 jxc.js.cn · psi.js.cn · ims.js.cn · jxc-w.com）+ 联系邮箱 + 版权
4. EN 与 ZH 同一页面的骨架**逐字对应**（仅语言不同），改一处必须改另一处

## 5. SEO 要求（重点）

### 5.1 页面级标签（两版各自唯一）
- `<title>` ≤ 60 字符：`标题 - erp-w.com`
- `meta description` ≤ 155 字符
- `canonical` 指向**本语言版本**自身 URL
- hreflang 三链：en 链英文页、zh-CN 链中文页、x-default 恒链英文页
- Open Graph 全套 + Twitter `summary`

### 5.2 结构化数据（JSON-LD）
- 首页：`WebSite`（inLanguage 各语言一条）；文章页：`Article` + `BreadcrumbList`；栏目页/列表页：`BreadcrumbList`
- 文章 datePublished/dateModified 用发布当天

### 5.3 内容结构
- 语义化 HTML5，唯一 H1；正文 H2 分节
- 英文正文 ≥ 600 词，中文正文 ≥ 800 字；首段 100 字内点题

### 5.4 URL 与内链
- 面包屑可见且与 BreadcrumbList 一致
- 内链三向：侧栏热门、正文互引、footer 站群互链
- 正文内链默认链**同语言**版本

### 5.5 爬虫与收录
- `robots.txt`：`Allow: /` + `Sitemap: https://erp-w.com/sitemap.xml`
- `sitemap.xml`：两棵树全部 URL 含 `<lastmod>`；发文时中英两条都加

## 6. 内容栏目（6 个）

| 栏目目录 | EN 名 | ZH 名 | 种子文章 slug |
|---------|-------|-------|--------------|
| arch/ | Architecture | 架构与选型 | modular-monolith.html |
| finance/ | Finance | 财务模块 | gl-document-integration.html |
| prod/ | Manufacturing & MRP | 生产与 MRP | bom-mrp-web.html |
| workflow/ | Workflow | 工作流审批 | visual-workflow-engine.html |
| integration/ | Integration | 系统集成 | ecommerce-integration.html |
| ui/ | Enterprise UI | 企业级界面 | enterprise-ui-patterns.html |

## 7. 分页与卡片约定

- 卡片：`<a class="card">` = `<h3>` + 摘要 `<p>` + `栏目 · 日期` meta
- 锚点分页：每组 `page-group` ≤ 6 卡，编号 `id="page-N"` 连续；每组紧跟一个 `.pager`；所有组同页展示，不用 `:target` 隐藏
- 首页「最新文章」固定 6 张不分页

## 8. 发文流程（新增一篇文章 = 约 10 个文件）

1. 创建英文文章页 `/articles/<slug>.html`
2. 创建中文文章页 `/zh/articles/<slug>.html`（同 slug，内容对译）
3. 英文栏目页 `/articles/<栏目>/index.html` 顶部插卡、计数 +1、按分页约定满组级联
4. 中文栏目页 `/zh/articles/<栏目>/index.html` 同步
5. 英文总列表 `/articles/index.html` 顶部插卡、计数 +1、满组级联
6. 中文总列表 `/zh/articles/index.html` 同步
7. 英文首页 `/index.html` 最新文章区顶部插卡，保持 6 张挤出最旧
8. 中文首页 `/zh/index.html` 同步
9. `sitemap.xml` 加中英两条 URL
10. node 校验（链接 0 缺失、JSON-LD 可解析、分页结构正确、hreflang 齐全）后 commit + push

详细步骤与校验脚本见 `docs/adding-articles.md`；skill 见 `.claude/commands/add-article.md`。
