---
title: 資料庫遷移：從 MySQL 到 MSSQL 的步驟指引
author: 小P
pubDatetime: 2025-02-21T00:00:00+08:00
slug: mysql-to-mssql-step-by-step
featured: false
draft: false
tags:
  - database
  - mysql
  - mssql
description: "使用 SSMA（SQL Server Migration Assistant）將 MySQL 資料庫遷移至 MSSQL 的完整步驟與常見問題解法。"
---

最近有遇到一個需求，就是要將 MySQL 上的資料庫轉移到 MSSQL。

雖然同樣是 SQL，但兩者語法還是有差，很多用法在 MySQL 上可以用，但在 MSSQL 上卻不行，因此無法直接將 MySQL 匯出的 sql 檔匯入 MSSQL。

後來受到高人指點才知道 Microsoft 有提供 MySQL 遷移至 MSSQL 的官方工具：SSMA（SQL Server Migration Assistant）。

官方文件：https://learn.microsoft.com/en-us/sql/sql-server/migrate/guides/mysql-to-sql-server

## 安裝 SSMA for MySQL

SSMA 全名為 **SQL Server Migration Assistant for MySQL**，顧名思義就是用來協助將 MySQL 資料庫遷移至 MSSQL 的工具。

下載點：https://www.microsoft.com/en-us/download/details.aspx?id=54257

## 安裝 MySQL ODBC Driver

由於 SSMA 與 MySQL 進行連線時需要用到 MySQL Connector/ODBC Driver，因此需要先安裝。

官方載點：https://dev.mysql.com/downloads/connector/odbc/

## 準備開始遷移

### 1. 在 SSMA 創建新 Project

點擊左上方 **File → New Project** 來創建專案，在創建時需要選擇要 Migrate 的 SQL Server 版本。

### 2. 連結 MySQL & MS SQL

創建完後點擊上方 **Connect to MySQL** 與 **Connect to SQL Server**，即可連接資料庫。

> 注意：在 MySQL 連結視窗中可以看到 Driver 選項，如果這邊是空的代表電腦裡沒有安裝 MySQL ODBC Driver。若剛裝完還是沒有出現，可以重啟電腦試試。

連線完後會讓你選擇要載入哪個資料庫，載入完後在各自的 Metadata 視窗就可以看到資料庫選項。

## 產生分析報告

在進行遷移之前，SSMA 提供一個分析報告功能，會顯示哪些結構在轉移後可能有問題。

在 MySQL Metadata 視窗，選擇要遷移的資料庫右鍵點擊 **Create Report**。

報告會以網站方式呈現，左方為 MySQL Schema，右方則為 MS SQL Schema，有 warning 與 error 訊息會出現在 MSSQL 視窗中。

建議先確認報告無誤後再進行轉移。

### 常見問題

**問題 1：Foreign keys with different types of columns and referenced columns can not be converted**

原因：在使用 Laravel 建置主 key 時是使用 UUID，而外 key 是 string(36)，導致主 key 為 `char(36)`、外 key 為 `varchar(36)`。在 MySQL 中可以關聯，但在 MSSQL 中會發生錯誤。

**解決方式**：統一主 key 與外 key 的資料型別。

---

**問題 2：ON DELETE CASCADE|SET NULL|SET DEFAULT action was changed to NO ACTION**

原因：某張資料表中有多個外 key 同時指向同一個父表，在執行 ON DELETE 行為時發生衝突。

**解決方式**：將 ON DELETE 行為改為 `NO ACTION`，避免產生衝突。

## 創建 Schema

確認報表沒問題後，點擊上方的 **Convert Schema**（此時不會實際在 MSSQL 建立資料表，而是先產生在 SQL Server Metadata）。

在 SQL Server Metadata 視窗，對目標資料庫右鍵點擊 **Synchronize with Database**，即可開始建立資料表與匯入 Schema。

## 轉移資料

建立完資料表後，在 MySQL Metadata 視窗對要移轉的資料庫右鍵點擊 **Migrate Data**。

點擊後會再次要求輸入 MySQL 與 MSSQL 的連線資訊，輸入後就會開始進行資料移轉。
