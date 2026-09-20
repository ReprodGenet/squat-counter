# 医刊速递 MedFeeds

生物医学期刊 RSS 订阅聚合站。挑选期刊订阅，以 RSS 方式集中接收各刊最新发表的文章。

## 快速开始

```bash
npm install
npm start          # 默认 http://localhost:3000
```

监听 `PORT` 环境变量，绑定 `0.0.0.0`。

## 功能

| 模块 | 说明 |
| --- | --- |
| 发现期刊 | 52 本主流生物医学期刊，10 个学科分类，支持关键词搜索与文章预览 |
| 最新文章 | 多源聚合时间流，未读标记、收藏、按刊筛选、全文搜索、摘要展开 |
| 我的订阅 | 订阅管理、偏好设置、OPML 导入导出（兼容 Feedly / Inoreader） |
| 自定义订阅 | 任意 RSS/Atom 链接，或按 PubMed 期刊名添加（覆盖 3 万+ 期刊） |
| 自动刷新 | 15 分钟 ~ 3 小时可调，新文章可选桌面通知 |

数据存于浏览器 localStorage，无需注册登录。

## 技术方案

- **前端**：原生 JS 单页应用，无框架依赖，localStorage 持久化
- **后端**：Node.js + Express，`rss-parser` 解析
- **期刊源**：52 本中 39 本为官方 RSS 直连，13 本经 PubMed E-utilities 兜底

### 接口

| 端点 | 说明 |
| --- | --- |
| `GET /api/catalog` | 期刊目录（分类 + 期刊元数据） |
| `GET /api/feed?url=&limit=` | RSS/Atom 代理抓取，支持 RDF(RSS 1.0) / RSS 2.0 / Atom |
| `GET /api/pubmed?journal=&limit=` | 按 PubMed 期刊名检索最新文献 |
| `GET /api/health` | 健康检查 |

### 关键实现

- **三源兜底**：官方 RSS 直连 → PubMed E-utilities → 正则兜底解析器。反爬拦截（Cloudflare 403）与失效源由 PubMed 覆盖，PubMed 中未收录的（如部分预印本）走官方 RSS。
- **PubMed 噪音过滤**：剔除勘误、目录页、栏目标目等非文献条目，并对"原文 + Authors' reply"配对做标题归一化去重。
- **缓存与限流**：10 分钟内存缓存；外发抓取并发上限 8；NCBI E-utilities 串行节流（≤3 req/s），失败自动重试一次。
- **安全**：RSS 代理限制为公网 http/https，拦截内网地址与 `.local`/`.internal` 域名，防 SSRF。

## 目录结构

```
medfeeds/
├── server.js              # Express 服务：静态托管 + API
├── data/journals.json     # 期刊目录（52 本，10 分类）
├── public/
│   ├── index.html         # 页面骨架
│   ├── style.css          # 样式（响应式）
│   └── app.js             # 前端逻辑
└── package.json
```
