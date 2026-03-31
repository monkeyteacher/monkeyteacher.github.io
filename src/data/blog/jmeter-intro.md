---
title: JMeter 介紹及使用心得－壓測工具筆記
author: 小P
pubDatetime: 2025-02-11T00:00:00+08:00
slug: jmeter-intro
featured: false
draft: true
tags:
  - jmeter
  - devops
  - 測試
description: "JMeter 壓測工具的安裝與使用教學，包含 Thread Group、HTTP Request、監聽器等基本操作說明。"
---

最近有個系統需要進行壓測，剛好以前有稍微碰過 jmeter，但並沒有寫筆記記錄。藉此趁這個機會來複習一下，順便記錄一下使用心得。

壓測工具有很多，比較著名的像是 ApacheBench、JMeter、k6……，而當初選擇 JMeter 主要是因為一張梗圖 XD

## JMeter 介紹

JMeter 與 ApacheBench（AB）一樣都是 Apache 基金會所維護的開源專案，但與 AB 不同的是 AB 只能進行 HTTP 的壓測且只提供 command 的方式，JMeter 支援更多的協定像是 HTTP/S、FTP、JDBC，且他還提供 GUI 介面讓用戶可以更直觀的使用。

## JMeter 安裝

JMeter 的安裝很簡單，他是由 Java 所撰寫且支援多種平台（Windows、Linux、Mac），只要有裝 Java 8+ 就可以執行，安裝方式只要去官網下載 Binaries 檔，下載完後直接解壓縮打開即可。

執行檔位置在解壓縮後的 `bin` 目錄底下。

> 在 Mac 上可能會遇到被系統判定為惡意軟體而無法開啟。記得去「系統設定 → 隱私權及安全性」拉到最下面，點擊「強制開啟」即可。

## JMeter 操作說明

由於 JMeter 的功能很多，這邊只介紹平常在用的部分，如果要進行簡單的壓測 API，看以下介紹就夠了。

### 開啟 JMeter

打開 JMeter 後會看到有一個 **Test Plan**。Plan 為 JMeter 儲存的基本單位，你可以設定完後把 Plan 存下來。一個頁面只能存在一個 Plan，但如果你想要同時操作多個 Plan 可以開多個視窗。

常用參數：

- **Run Thread Groups consecutively**：設定在執行 Plan 時，每個 Thread Groups 是同時進行壓測，還是一個執行完後換下一個。

### 創建 Thread Group

Thread Group（線程組）：用來定義壓測時的用戶數、啟動時間和循環次數等。

常用參數說明：

| 參數 | 說明 |
|------|------|
| Number of Thread (users) | 使用者數量 |
| Ramp-up period (seconds) | 啟動所有 Thread 所需要的時間 |
| Loop Count | 每個 Thread 執行的次數（勾選 Infinite 則會一直跑） |

**範例**：模擬 100 位用戶同時登入網站：

- Number of Thread (users)：`100`
- Ramp-up period (seconds)：`5`（一般用戶不會真的同時進入）
- Loop Count：`1`

### 加入 HTTP Request

創建方式：右鍵點擊 Thread Group → Add → Sampler → **Http Request**

> 注意：如果是 POST 要上傳檔案，要點擊 Parameters 旁邊的 **Files Upload**。

### 加入 HTTP Header Manager

有時候 API 在呼叫時會需要在 Header 設定參數，例如 `Accept`、`Authorization` 等。

創建方式：右鍵點擊 Thread Group → Add → Config Element → **Http Header Manager**

### 加入監聽器

以下三個 Listener 為常用的壓測結果查看工具：

#### Summary Report

提供每個 sampler 的測試結果，相同 Label 的壓測結果會合併在一起。

創建方式：右鍵點擊 Thread Group → Add → Listener → **Summary Report**

常用欄位說明：

| 欄位 | 說明 |
|------|------|
| Label | 對應到 sampler 的 name |
| Samples | 請求的數量 |
| Average | 每個請求的平均回應時間（毫秒） |
| Min | 所有請求中最小的回應時間（毫秒） |
| Max | 所有請求中最大的回應時間（毫秒） |
| Error % | 請求錯誤的百分比 |
| Throughput | 每秒處理的請求量 |
| KB/sec | 每秒處理的數據量（KB） |

#### View Results Tree

可以查看每個請求的結果與回應訊息，如果 API 有錯誤訊息也可以從這邊看。

創建方式：右鍵點擊 Thread Group → Add → Listener → **View Results Tree**

#### View Results in Table

把每個請求的資料表格化，方便比較。

創建方式：右鍵點擊 Thread Group → Add → Listener → **View Results in Table**
