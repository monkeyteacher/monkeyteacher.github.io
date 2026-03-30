---
title: 部落格之旅 - 使用 Astro 重建部落格
author: 小P
pubDatetime: 2025-03-25T00:00:00+08:00
slug: recreate-blog-use-astro
featured: true
draft: false
tags:
  - astro
  - hugo
  - devops
  - github-actions
description: ""
---

## 背景
去年年初的時候有試著使用 hugo 建置部落格，但後來工作繁忙就停更了。
最近剛好有時間可以來進行整理，卻發現原本的 hugo 專案好像不見了...
由於 github 上面只有保留靜態頁面的輸出內容，所以乾脆直接重建一個。


## 事前評估
在開始重建前有稍微評估是否要繼續使用 hugo，還是改用其他套部落格的套件。
在上一份工作有稍微玩過 Astro ，體驗其實還不錯，再加上他不像是 hugo 是原始碼和輸出分開管理。
只需要管理一個專案就好，所以不會遇到跟這次一樣原碼不見的問題XD

另外目前有再練習 devops 相關技能，在規畫時有加入 Github Action 的流程，來建立簡單的 CI/CD 機制。
原本 Astro 就由提供 Github Action 相關範例，所以很適合作練習。

## 遷移過程
在這個 AI 時代，部落格的建置及舊檔案移轉當然是由 AI 來進行實作，而我是負責內容的把關及 Github Action 的部分。

### 建立專案

先建立基本的 astro 專案，版型的話我是使用 [astro-paper](https://github.com/satnaing/astro-paper)

``` 
npm create astro@latest  -- --template satnaing/astro-paper
```
### 目錄架構

第一步當然是先從目錄架構開始，要先知道基礎設定要在哪個檔案，文章的 markdown 要放在哪。
基本上我們撰寫的程式碼都是在 src 目錄裡面，而 astro-paper 樣板的目錄如下：

```
專案目錄
│
├── src/                        ← 所有原始碼在這裡
│   ├── config.ts               ← 部落格基本設定（名稱、作者、網址等）
│   │
│   ├── data/blog/              ← 文章放這裡（.md 檔）
│   │   ├── XXX.md
│   │   └── ...
│   │
│   ├── pages/                  ← 網站的每一個「頁面」
│   │   ├── index.astro         ← 首頁
│   │   ├── about.md            ← 關於頁面
│   │   ├── posts/              ← 文章列表 & 文章內頁
│   │   ├── tags/               ← 標籤頁面
│   │   ├── archives/           ← 封存頁面
│   │   ├── search.astro        ← 搜尋頁面
│   │   └── rss.xml.ts          ← RSS 訂閱
│   │
│   ├── layouts/                ← 頁面的外框模板（Header + Footer 等共用結構）
│   │   ├── Layout.astro        ← 最基礎的 HTML 結構
│   │   ├── Main.astro          ← 主要內容區塊
│   │   └── PostDetails.astro   ← 文章內頁的模板
│   │
│   ├── components/             ← 可重複使用的小元件
│   │   ├── Header.astro        ← 頁首
│   │   ├── Footer.astro        ← 頁尾
│   │   ├── Card.astro          ← 文章卡片
│   │   ├── Tag.astro           ← 標籤元件
│   │   └── Pagination.astro    ← 分頁元件
│   │
│   ├── assets/                 ← 靜態資源
│   │   ├── icons/              ← SVG 圖示（GitHub、LinkedIn 等）
│   │   └── images/             ← 圖片
│   │
│   ├── styles/                 ← CSS 樣式
│   └── utils/                  ← 工具函式（排序文章、產生 OG 圖等）
```

### 常會動到的檔案
| 動作 | 對應檔案/目錄 |
|---------|-------------|
| 寫新文章 | `src/data/blog/` 新增 `.md` |
| 改部落格名稱、作者 | `src/config.ts` |
| 改關於頁面 | `src/pages/about.md` |
| 改首頁內容 | `src/pages/index.astro` |
| 改頁首/頁尾 | `src/components/Header.astro` / `Footer.astro` |


## DevOps 實踐：CI/CD Pipeline

本次重建還有一個重點是實作 CI/CD 流程，這個專案是預計部屬在 github page 上面。
astro 官方有直接提供簡易的 [workflow](https://docs.astro.build/zh-tw/guides/deploy/github/) 可以將專案部屬到 GitHub Page。
但我這邊想要練習基本 github action，所以沒有直接使用官方的範例。

### CI/CD Pipeline 
這個專案的 Pipeline 會有四個階段：

```
lint → build → lighthouse ci → deploy
```

1. **lint**： `astro check` 型別檢查 + `markdownlint` 檢查文章格式
2. **build**：`npm run build` 產生靜態檔案，並打包到 `dist`
3. **lighthouse**：下載 `dist/`，測試效能/SEO/Accessibility 分數，結果僅供參考（警告模式，不擋部署）
4. **deploy**：把 `dist/` 部署到 GitHub Pages

觸發條件：`git push` 到 `main` 分支時自動執行，也可以在 GitHub 網頁上手動觸發。

### 撰寫流程
一開始是請 Claude Code 產生初版 deploy.yml，然後我再跳進去調整。

初版的 deploy.yml 雖然可以部屬成功，但細看的其實還是有些問題的。
1. 使用不必要的環境變數：`SITE`  
  `astro.config.ts` 的 `site` 是從 `src/config.ts` 讀固定值，並非從環境變數讀取，所以這個 `env` 完全沒有作用。
2. 使用不必要的步驟：`actions/configure-pages`  
  初版加了 `configure-pages` 步驟，但官方範例和實際部署流程中，`upload-pages-artifact` 和 `deploy-pages` 本身就能處理，不需要這個步驟。
3. 使用舊版的 GitHub Actions 版本
    | Action | 初版 | 最終版 |
    |--------|------|--------|
    | `actions/checkout` | v4 | v6 |
    | `actions/setup-node` | v4 | v6 |
    | `actions/upload-artifact` | v4 | v6 |
    | `actions/download-artifact` | v4 | v8 |
    | `actions/configure-pages` | v5 | 移除 |
    | `actions/upload-pages-artifact` | v3 | v4 |
    | `actions/deploy-pages` | v4 | v5 |
    | `node-version` | 20 | 24 |

修改完後，變得比較簡潔，且不會跳出版本過舊的警告訊息。

### 定時排程
由於還沒玩過 Githut action 定期排程，趁這個機會也來玩一下。  
在看其他人的部落格時常會發現舊文章會有一些失效連結，想說可以使用定期排程讓 github action 去檢查 md 檔裡面有沒有失效連結。

撰寫方法也很簡單：
```yaml
on:
  schedule:
    - cron: '0 0 * * 1'  # 每週一 00:00 UTC 執行
  workflow_dispatch:      # 也可以在 GitHub 網頁上手動觸發
```
cron 的格式就跟 Linux crontab  一樣
> 注意：GitHub Actions 的排程時間是 **UTC**。

本專案用途：`link-check.yml` 每週一自動掃描 `src/data/blog/**/*.md` 裡的無效連結，失敗時 GitHub 會寄 Email 通知。

## QA 
由於我比較熟悉的 CI 工具是 GitLab CI，因此在撰寫這份 GitHub Action 時有紀錄一些跟 AI 問答，整理如下：

### Q: GitHub Actions yml 裡 `runs-on` 和 `uses` 是什麼？

- **`runs-on`**：指定 Job 跑在哪種虛擬機上，`ubuntu-latest` 就是 GitHub 提供的 Ubuntu Linux，每個 Job 都是全新乾淨的 VM
- **`uses`**：引用別人寫好的 Action（格式：`{作者}/{名稱}@{版本}`）
`actions/` 開頭是 GitHub 官方，`treosh/` 是第三方開發者。

### Q: GitHub Actions 的 `actions/checkout` 在 GitLab 有對應的東西嗎？

`actions/checkout` 是 GitHub Actions 專屬的，GitLab CI/CD 不需要這個步驟。

- **GitHub Actions**：每個 Job 跑在全新空白的 VM，所有步驟都要明確宣告，彈性較高。
- **GitLab CI/CD**：預設 clone 程式碼，開箱即用，設定相對簡潔。

### Q: `npm ci` 和 `npm install` 有什麼差別？

兩者都是安裝套件，但用途不同：

| | `npm install` | `npm ci` |
|---|---|---|
| 用途 | 本地開發 | CI/CD 環境 |
| 依據 | `package.json` | `package-lock.json` |
| 版本 | 允許小版號更新 | 完全鎖定版本 |
| 速度 | 較慢 | 較快 |

**關鍵**：`npm ci` 嚴格照 `package-lock.json` 安裝，保證每次 CI 環境的套件版本跟本地開發完全一致，避免「本地可以跑，CI 跑不過」的問題。  
`npm ci` 在執行前會先刪除整個 node_modules 再重裝，這是它比 npm install 更適合 CI 環境。

