# 小P's Blog

個人技術部落格，記錄 DevOps / Backend 實戰經驗與學習心得。

🌐 **https://monkeyteacher.com**

## Tech Stack

- **Framework**: [Astro](https://astro.build/) + [AstroPaper](https://github.com/satnaing/astro-paper)
- **Deployment**: GitHub Pages
- **CI/CD**: GitHub Actions
- **Local Dev**: Docker Compose

## CI/CD Pipeline

```
push to main
    → Lint (markdownlint + astro check)
    → Build (astro build)
    → Lighthouse CI (performance/SEO ≥ 90, accessibility ≥ 85)
    → Deploy (GitHub Pages)
```

## Local Development

**使用 Docker（不需安裝 Node.js）**

```bash
docker-compose up
```

**使用 npm**

```bash
npm install
npm run dev
```

開啟 http://localhost:4321

## 新增文章

在 `src/data/blog/` 新增 `.md` 檔：

```yaml
---
title: 文章標題
author: 小P
pubDatetime: 2025-01-01T00:00:00+08:00
slug: article-slug
tags:
  - devops
description: "文章摘要"
draft: false
---
```

寫完後 `git push`，自動部署。
