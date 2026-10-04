# erp-w.com 发文手册

> 本手册配套 skill `.claude/commands/add-article.md`。先读 `REQUIREMENTS.md`。
> 本站为**中英双语**：新增一篇文章 = 创建 EN + ZH 两个页面 + 同步 4 个列表页 + 2 个首页 + sitemap，共约 10 个文件。

## 1. 双语 URL 树

| | 英文（默认） | 中文 |
|---|---|---|
| 首页 | `/index.html` | `/zh/index.html` |
| 全部文章 | `/articles/index.html` | `/zh/articles/index.html` |
| 栏目页 | `/articles/<栏目>/index.html` | `/zh/articles/<栏目>/index.html` |
| 文章页 | `/articles/<slug>.html` | `/zh/articles/<slug>.html` |
| 关于 | `/about.html` | `/zh/about.html` |

中英文章**共用同一 slug**；hreflang 三链中 x-default 恒指英文版。

## 2. 六个栏目映射

| 目录 | EN 名 | ZH 名 |
|------|-------|-------|
| arch | Architecture | 架构与选型 |
| finance | Finance | 财务模块 |
| prod | Manufacturing & MRP | 生产与 MRP |
| workflow | Workflow | 工作流审批 |
| integration | Integration | 系统集成 |
| ui | Enterprise UI | 企业级界面 |

## 3. 页面骨架（以英文文章页为准，中文逐字对应）

1. `<head>`：title（≤60 字符）/ description（≤155）/ canonical（本语言 URL）/ hreflang 三链 / OG 全套 / Twitter summary
2. JSON-LD：`Article` + `BreadcrumbList`（首页→栏目页→本文章，三条 item）
3. header：logo、checkbox 汉堡、9 项导航、`.lang-switch`（EN 页"中文"→ `/zh/articles/<slug>.html`；ZH 页"English"→ `/articles/<slug>.html`）
4. 面包屑（与 BreadcrumbList 一致）
5. sidebar：栏目 7 链 / 热门文章 3 篇 / 关于本站
6. `<article>`：唯一 h1、`发布时间：YYYY-MM-DD | 分类：栏目名`（EN 用 `Published: ... | Category: ...`）
7. 正文：首段 100 字内点题；3–5 个 h2；1–2 处链**同语言**站内文章的内链
8. 文末 `.article-nav`：上一篇/下一篇（同栏目相邻文章，最新一篇"下一篇"用 `<span></span>` 占位）
9. footer：三栏 + 四站互链 + webnic@qq.com

## 4. 分页设计约定（硬规则）

1. 同一 HTML 内多个 `<div class="page-group" id="page-N">`，N 从 1 连续编号；页码用 `#page-N` 锚点
2. **所有分组同时展示**，页码只是滚动导航；禁用 `:target` 隐藏
3. 每组 ≤ 6 卡；新卡插组顶，满 7 张把该组最后一张（最旧）移入下一组，级联处理
4. **每个 page-group 紧跟一个 `.pager`**，页码覆盖全部组，当前组标 `<span class="current">K</span>`
5. 首页不分页，「最新文章」固定 6 张

## 5. 新增文章完整步骤

1. 定 slug（2–4 个英文语义词，小写连字符；确认 `/articles/` 与 `/zh/articles/` 均无同名文件）、定栏目、取当天日期
2. 以现有文章页（如 `articles/modular-monolith.html`）为模板创建 EN 页
3. 创建 ZH 页：同 slug、内容对译、lang="zh-CN"、中文导航/面包屑/sidebar/footer、meta 用中文
4. EN 栏目页顶部插卡，计数 +1，按第 4 节处理满组
5. ZH 栏目页同步
6. EN `/articles/index.html` 顶部插卡，计数 +1，满组处理
7. ZH `/zh/articles/index.html` 同步
8. EN 首页「最新文章」顶部插卡保持 6 张；ZH 首页同步
9. `sitemap.xml` 的 `</urlset>` 前插入中英两条：
   `<url><loc>https://erp-w.com/articles/<slug>.html</loc><lastmod>日期</lastmod><priority>0.7</priority></url>`
   `<url><loc>https://erp-w.com/zh/articles/<slug>.html</loc><lastmod>日期</lastmod><priority>0.6</priority></url>`
10. 校验 + commit + push

## 6. 校验（node -e，本环境 shell 变量展开异常勿用 bash 循环）

```bash
node -e '
const fs=require("fs"),path=require("path");
function walk(d){return fs.readdirSync(d,{withFileTypes:true}).flatMap(e=>e.isDirectory()?walk(path.join(d,e.name)):[path.join(d,e.name)])}
let missing=[],checked=0;
for(const f of walk(".").filter(f=>f.endsWith(".html"))){
  const html=fs.readFileSync(f,"utf8");
  for(const m of html.matchAll(/href="(\/[^"#]*)"/g)){
    let p=m[1]; if(!p||p.startsWith("//"))continue;
    if(p.endsWith("/"))p+="index.html";
    checked++; if(!fs.existsSync("."+p))missing.push(f+" -> "+m[1]);
  }
  for(const j of html.matchAll(/<script type="application\/ld\+json">([\s\S]*?)<\/script>/g)){try{JSON.parse(j[1])}catch(e){missing.push(f+" -> JSON-LD parse error")}}
}
console.log("checked",checked,"missing/errors",missing.length);missing.forEach(x=>console.log(x));
'
```

通过标准：0 缺失、0 JSON-LD 错误。另人工确认：中英两版卡片计数一致、hreflang 三链齐全、分页结构符合第 4 节。

## 7. 上线与推送

1. `git add -A && git commit -m "Add article: <slug> (EN+ZH)" && git push`
2. push 被拒先 `git pull --rebase origin main`（远程可能有网页端 CNAME 操作提交）
3. 新站上线向百度/Google/Bing 站长平台提交 `https://erp-w.com/sitemap.xml`
