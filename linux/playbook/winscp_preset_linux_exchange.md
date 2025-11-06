# 能交換檔案之前, 會需要在虛擬機先安裝SSH Server
# 安裝 SSH Server 教學 (適用 Ubuntu / Debian 系統)

## ☑️ 前言

如果在 Linux VM 中執行 `sudo systemctl status ssh` 出現:

```
Unit ssh.service could not be found.
```

表示 **SSH Server (openssh-server)** 還未安裝。

---

## ☑️ 步驟 1: 更新 apt 資料庫

```bash
sudo apt update
```

---

## ☑️ 步驟 2: 安裝 openssh-server

```bash
sudo apt install openssh-server
```

載入安裝完成後，SSH 會自動啟動。

---

## ☑️ 步驟 3: 啟動 SSH 服務

```bash
sudo systemctl enable ssh   # 上次服務啟動時自動啟動
sudo systemctl start ssh    # 立即啟動 ssh
```

---

## ☑️ 步驟 4: 確認 SSH 狀態

```bash
sudo systemctl status ssh
```

如看到:

```
Active: active (running)
```

就是啟動成功！

---

## ☑️ 步驟 5: 確認 VM IP

查看 VM IP ：

```bash
ip a
```

或

```bash
hostname -I
```

例如：

```
192.168.72.128
```

---

## ☑️ WinSCP 連線設定

- **協定**: SFTP
- **主機名稱**: VM IP (e.g. 192.168.72.128)
- **使用者名稱**: VM 帳號
- **密碼**: VM 帳號密碼

如果密碼打錯, 會出現下圖
![image](uploads/2ebd66eccf8d6d97dba83d9175760f87/image.png)

連上後就可以操作 Linux VM 相關資料。

---

## ☑️ 附錄: 關閉 SSH 服務

如果需要關閉 SSH：

```bash
sudo systemctl stop ssh
```

可停用多餘的 SSH 服務。

---
**撰寫紀錄：**
| 作者      | 職稱             | 日期       | 備註   |
|----------|-----------------|------------|-------|
| Ian Wang | 資深後端工程師    | 2025/07/11 | 初版   |