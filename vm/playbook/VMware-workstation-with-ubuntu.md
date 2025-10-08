# VMware Workstation 建立 Linux 虛擬機
# 建立相同規格的 Linux VM（VMware Workstation）

本教學將帶你在 VMware Workstation 中建立一台具有以下規格的 Linux 虛擬機：

* **2 核心 / 4 執行緒**
* **15 GiB 記憶體**
* **適合安裝 Docker 與全端應用（Nginx、MySQL、前後端等）**

---

## ✅ 準備環境

1. **下載 VMware Workstation**

   * [VMware Workstation Player](https://www.vmware.com/products/workstation-player.html) 下載免費版

2. **下載 Linux ISO**

   * Debian 12: [https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)
   * Ubuntu Server 22.04: [https://ubuntu.com/download/server](https://ubuntu.com/download/server)

---

## ✅ 步驟 1: 新建虛擬機

1. 打開 VMware Workstation
2. 點選 `Create a New Virtual Machine`
3. 選擇：`Installer disc image file (iso)`，載入 Linux ISO

---

## ✅ 步驟 2: 選擇 OS 類型

* Guest operating system: `Linux`
* Version: `Debian 10/11/12` 或 `Ubuntu 64-bit`

---

## ✅ 步驟 3: 設定虛擬機名稱與儲存位置

* VM Name: `linux-docker-server`
* VM Location: 選擇確保碼光大空間路徑

---

## ✅ 步驟 4: 設定磁碟容量

* 建議: `30~60GB`
* 選擇: `Split virtual disk into multiple files`

---

## ✅ 步驟 5: 自訂虛擬機硬體設定

點選 `Customize Hardware`，調整下列設定:

| 項目         | 設定值                     |
| ---------- | ----------------------- |
| Memory     | 15360 MB (15GiB)        |
| Processors | 1 processor, 2 cores    |
| Network    | NAT or Bridged (freely) |
| CD/DVD     | ISO image path          |

---

## ✅ 步驟 6: 啟動虛擬機

* 啟動 VM ，進入 Linux 安裝程式
* 按漸進導引完成安裝
* 安裝完成後重啟並移除 ISO 啟動光碟

---

## ✅ 步驟 7: 確認系統資源

```bash
lscpu      # 確認 CPU 2 core / 4 thread
free -h    # 確認 15Gi RAM
```

---
---
**撰寫紀錄：**
| 作者      | 職稱             | 日期       | 備註   |
|----------|-----------------|------------|-------|
| Ian Wang | 資深後端工程師    | 2025/07/10 | 初版   |