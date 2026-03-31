---
title: Docker Logs Viewer Tool
author: 小P
pubDatetime: 2025-01-21T00:00:00+08:00
slug: docker-logs-viewer-tool
featured: false
draft: true
tags:
  - docker
  - devops
description: "介紹 docker logs 指令及好用的開源 log 查看工具：Portainer 與 Dozzle。"
---

在 debug 或是開發的時候，常常會需要查看 docker logs。若是使用 Docker Desktop 可以使用 GUI 來查看 logs，但如果是在 linux 的平台上目前都還是使用下指令的方式來查看。使用指令的方式來查看會遇到以下問題：

- container 太多需要一直切來切去。
- 且在終端機上比較難操作或是查看。

因此來研究一下有哪些好用的 docker log viewer。

## 基本 docker logs 介紹

先簡單的介紹一下 docker logs 這個指令。顧名思義 docker logs 就是來查看 docker 中 container 所產生的 log。詳細說明可以參考[官方網站](https://docs.docker.com/reference/cli/docker/container/logs/)。

### 指令說明

```bash
docker logs [OPTIONS] CONTAINER
```

- `CONTAINER`：可以指定容器的 ID 或是容器名稱。
- `[OPTIONS]`：可以調整 log 的顯示方式或是過濾條件。

### 常用的 OPTIONS

`--follow (-f)` 可以實時跟蹤容器的日誌輸出：

```bash
docker logs -f <container_id_or_name>
```

`--tail (-n)` 顯示最後 n 行：

```bash
docker logs --tail 100 <container_id_or_name>
```

`--since` 顯示某個時間點後的 log：

```bash
docker logs --since "2025-01-20T00:00:00" <container_id_or_name>
```

### 存放位置

docker log 通常是使用 json 的方式儲存：

```
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

以下介紹的工具皆為開源工具，且皆有提供 docker container 的部署方式。

---

## Portainer

- 官網：https://www.portainer.io/
- GitHub：https://github.com/portainer/portainer
- 星星數：31.7k（2025/01/20）

Portainer 是一個管理 docker 的開源工具。簡單來說就是一個網頁版的 Docker Desktop，可以管理 Docker 中的 container、images、network、volumes……

其中 Portainer 有分 Community Edition（CE）與 Business Edition（BE），以下討論皆以 CE 為主。

### 部署方式

```bash
# docker pull
sudo docker pull portainer/portainer-ce

# docker run
sudo docker run -d -p 9000:9000 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

第一次登入時會要求設定帳號密碼，畢竟此工具可以操作大部分的 docker 功能。

---

## Dozzle

- 官網：https://dozzle.dev/
- GitHub：https://github.com/amir20/dozzle
- 星星數：7.4k（2025/01/20）

此為專門監控 docker logs 的開源工具。Dozzle 不存儲任何的 log，就是單純的即時監控 docker container 的 log。

### 特點

- 可以使用模糊搜尋、正規表達等方式來查詢 log。
- 佔用記憶體很小。
- 支援分割畫面查看多個 logs。
- 即時查看記憶體與 CPU。

### 部署方式

```bash
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -p 8080:8080 \
  amir20/dozzle:latest
```
