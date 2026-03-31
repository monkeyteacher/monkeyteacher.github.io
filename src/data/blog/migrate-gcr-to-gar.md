---
title: GCR 移轉至 GAR 筆記
author: 小P
pubDatetime: 2025-02-08T00:00:00+08:00
slug: migrate-gcr-to-gar
featured: false
draft: true
tags:
  - gcp
  - gcr
  - gar
  - devops
description: "Google Container Registry 停止服務前，使用官方自動移轉指令將 images 搬遷至 Google Artifact Registry 的實戰紀錄。"
---

由於 Google Container Registry 在 2025/03/18 就會停止服務，因此需要在這之前將 GCR 中的 images 轉移至 GAR（Google Artifact Registry）。

Google 有提供可以自動轉移的指令，參考網址：https://cloud.google.com/artifact-registry/docs/transition/auto-migrate-gcr-ar

## 轉移指令介紹

```bash
gcloud artifacts docker upgrade migrate \
    --projects=PROJECTS
```

可用參數：

- `--recent-images=DAYS`：只複製 30-180 天內有從 GCR 拉取（pull）的 images。
- `--uploaded-version=VERSIONS`：複製每個 image 特定數量的上傳版本。

> 注意：以上**兩個參數無法同時使用**。

## 本次移轉歷程

一開始先使用 Google 提供的指令查看目前 GCR 的使用狀況：

```bash
gcloud container images list-gcr-usage \
    --project=PROJECT
```

確認完後執行轉移指令，流程如下：

1. 在 GAR 上建立每個 `gcr.io` 專案的 repositories。
2. 為每一個 repository 設定 IAM 策略（採用 Google 建議策略，用戶將擁有在 Artifact Registry 上執行所有與 Container Registry 相同操作的權限）。
3. 將 `gcr.io` 的流量都轉到 GAR。
4. 將 `gcr.io` 的所有 images 複製到 GAR 上。

此專案最後成功移轉 **2 個 tags** 與 **103 個 manifests**，並成功將 `*.gcr.io` 的流量轉移至 GAR。

> 由於轉移是用**複製**的方式，結束後若測試沒問題記得還要去 GCS 把原本 GCR 上的 images 清除。參考：https://cloud.google.com/artifact-registry/docs/transition/clean-up-images-gcr

## 遇到的問題

### 1. 執行時遇到找不到指令

```
ERROR: (gcloud) Invalid choice: 'upgrade'
```

**解決方式**：將 gcloud 套件更新至最新版本。

```bash
gcloud components update
```

### 2. 帳號權限不足

錯誤訊息會建議需要給予的權限，依照提示在 IAM 頁面補上對應的 roles 即可。
