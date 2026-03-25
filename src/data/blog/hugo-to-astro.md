---
title: 部落格之旅 - 從 Hugo 到 Astro 的遷移紀錄
author: 小P
pubDatetime: 2025-03-25T00:00:00+08:00
slug: hugo-to-astro
featured: true
draft: false
tags:
  - astro
  - hugo
  - devops
  - github-actions
description: "記錄將部落格從 Hugo + Blowfish 遷移至 Astro + AstroPaper 的過程，並建立完整的 CI/CD 自動部署流程。"
---

去年開始用 Hugo + Blowfish 主題建立了這個技術部落格，但有一天打開電腦發現 Hugo 專案不見了，只剩下 GitHub Pages 上的靜態 HTML……

既然都要重建，乾脆就趁這個機會換成 **Astro**，並把整個部署流程做得更有 DevOps 的樣子。

## 為什麼從 Hugo 換到 Astro

| | Hugo | Astro |
|---|---|---|
| 語言 | Go template | HTML / JSX / MDX |
| 擴充性 | 有限 | 支援 React、Vue 等框架 |
| 生態系 | 成熟穩定 | 活躍快速成長 |
| 學習曲線 | 中等（Go template 較特殊） | 低（接近原生 HTML） |

主要原因是 Astro 對前端工程師更直覺，且 AstroPaper 主題夠簡潔，非常接近我偏好的風格。

## 遷移過程

### 1. 初始化 Astro 專案

```bash
npm create astro@latest . -- --template satnaing/astro-paper
```

### 2. 調整基本設定

修改 `src/config.ts`：

```typescript
export const SITE = {
  website: "https://monkeyteacher.com",
  author: "小P",
  desc: "DevOps / Backend 技術筆記",
  title: "小P's Blog",
  lang: "zh-TW",
  timezone: "Asia/Taipei",
  // ...
};
```

### 3. 遷移舊文章

從靜態 HTML 逆向還原成 Markdown，放入 `src/data/blog/`。AstroPaper 的 frontmatter 格式：

```yaml
---
title: 文章標題
author: 小P
pubDatetime: 2025-01-21T00:00:00+08:00
slug: article-slug
tags:
  - docker
  - devops
description: "文章摘要"
---
```

## DevOps 實踐：CI/CD Pipeline

這次重建的重點不只是換框架，而是建立完整的自動化部署流程。

### GitHub Actions Pipeline

整個 Pipeline 分為四個階段：

```
lint → build → lighthouse → deploy
```

1. **lint**：markdownlint 檢查文章格式 + `astro check` 型別檢查
2. **build**：`npm run build` 產生靜態檔案
3. **lighthouse**：跑 Lighthouse CI，效能/SEO 分數低於門檻就擋住部署
4. **deploy**：部署至 GitHub Pages

### Lighthouse CI 品質門檻

設定 `.lighthouserc.json`：

```json
{
  "ci": {
    "assert": {
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "categories:seo": ["error", {"minScore": 0.9}],
        "categories:accessibility": ["error", {"minScore": 0.85}]
      }
    }
  }
}
```

每次 push 都會自動驗證分數，確保部落格品質。

### Docker 本地開發環境

AstroPaper 本身已內建 `docker-compose.yml`，直接使用：

```bash
docker-compose up
# 開啟 http://localhost:4321
```

這樣不需要在本機安裝 Node.js，任何有 Docker 的環境都能直接開發。

## 成果

- 從 Hugo 原始碼遺失，到 Astro 完整重建，整個部落格現在有了完整的版本控制與自動化部署。
- 每次寫完文章只需要 `git push`，GitHub Actions 就會自動跑完所有檢查並部署。
- Lighthouse 分數維持在 90+ 以上。
