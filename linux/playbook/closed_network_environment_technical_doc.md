# 封閉網環境 Docker + React + Spring Boot + PostgreSQL 技術文件

## 🧩 環境說明

本文件說明在封閉網路環境下，如何建置並運行以下系統：

- **Docker Engine**（容器管理）  
- **React 前端**（使用 Node.js 編譯）  
- **Spring Boot 後端**（Java JAR 執行）  
- **PostgreSQL**（外部資料庫連線）

目的：確保系統在無法直接對外的情況下，透過精確白名單設定與離線安裝流程，仍可完成部署與運行。

---

## 🧱 主要步驟

### 1️⃣ 安裝 Docker Engine
- 需能存取官方 apt repository：`download.docker.com`
- 若環境完全封閉，可離線下載 `.deb` 套件與相依套件後安裝。

### 2️⃣ 安裝 Node.js 與 npm
- 用於 React 專案編譯與打包。
- 需能連線至 `deb.nodesource.com` 與 `registry.npmjs.org`。
- 若封閉環境，可使用離線的 `node-vXX.tar.xz` 套件。

### 3️⃣ 安裝 PostgreSQL client
- 連線至外部 PostgreSQL 資料庫（僅需安裝 `psql`）。
- 套件來源：`apt.postgresql.org`。

### 4️⃣ 啟動應用服務
- 以 Docker Compose 啟動：  
  - `nginx`：前端靜態頁面服務。  
  - `openjdk`：執行 Spring Boot JAR。  
  - `node`：若需在容器中編譯前端。

### 5️⃣ 連線外部資料庫
- Spring Boot 後端與 `psql` 工具皆需連線至外部 PostgreSQL。
- 資料庫 IP：`192.168.xxx.xx`
- 連線埠：`5432`

---

## 🌐 白名單設定

### ✅ 最終建議白名單清單

```
ALLOW OUT tcp 443 TO download.docker.com
ALLOW OUT tcp 443 TO registry-1.docker.io
ALLOW OUT tcp 443 TO auth.docker.io
ALLOW OUT tcp 443 TO deb.nodesource.com
ALLOW OUT tcp 443 TO registry.npmjs.org
ALLOW OUT tcp 443 TO apt.postgresql.org
ALLOW OUT tcp 80,443 TO archive.ubuntu.com
ALLOW OUT tcp 443 TO security.ubuntu.com
ALLOW OUT tcp 5432 TO 192.168.xxx.xx
```

---

### 📘 用途說明

| 類別 | 網域 / IP | Port | 用途 |
|------|-------------|------|------|
| Docker Engine | download.docker.com | 443 | 安裝 Docker CE / CLI |
| Docker Hub Registry | registry-1.docker.io / auth.docker.io | 443 | 拉取映像檔 (nginx / openjdk / node 等) |
| Node.js / npm | deb.nodesource.com / registry.npmjs.org | 443 | 安裝 Node.js / 前端依賴套件 |
| PostgreSQL client | apt.postgresql.org | 443 | 安裝 psql 工具 |
| Ubuntu Repository | archive.ubuntu.com / security.ubuntu.com | 80,443 | 系統更新與安全套件 |
| PostgreSQL DB | 192.168.xxx.xx | 5432 | Spring Boot / psql 連線資料庫 |

---

## 🧭 離線安裝替代方案

| 項目 | 作法 |
|------|------|
| Docker Engine | 外部下載 `.deb` 檔後用 `dpkg -i` 安裝 |
| Docker Images | `docker pull` → `docker save` → 搬入封閉區 → `docker load` |
| Node.js | 下載官方二進位包 `.tar.xz` 安裝 |
| npm 套件 | 外部 `npm ci --offline` 打包成 `node_modules` |
| PostgreSQL client | 外部 `apt download postgresql-client` 取得 `.deb` |
| React 前端 | 外部 `npm run build`，只搬 build/ 資料夾 |
| Spring Boot | 只需 JDK / Docker image 即可執行 JAR |

---

## ✅ 結論

開放上述白名單後，封閉環境將能：
- 正常安裝 Docker、Node.js、PostgreSQL client。  
- 成功拉取必要映像檔（nginx、openjdk、node）。  
- 編譯並執行 React + Spring Boot 專案。  
- 連線至內部資料庫 `192.168.xxx.xx:5432`。

此為**最小必要開放**的白名單設計，兼顧系統可用性與資訊安全。

