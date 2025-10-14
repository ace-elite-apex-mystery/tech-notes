# 🧰 Linux 將使用者 `heroic` 提升為超級使用者（root）

---

## 🧩 方法一：將 `heroic` 加入 `sudo` 群組（建議方式）

這是最安全、最標準的方式。  
加入後即可使用 `sudo` 執行任何 root 權限命令。

### 1️⃣ 以 root 身分登入（如果目前帳號沒有 sudo 權限）
```bash
su -
```

### 2️⃣ 將 heroic 加入 sudo 群組  
- **Debian / Ubuntu 系列：**
```bash
usermod -aG sudo heroic
```
- **CentOS / RHEL 系列：**
```bash
usermod -aG wheel heroic
```

### 3️⃣ 登出並重新登入
以下是常見的登出 / 登入指令（視場景而定）：

- 如果你在 root shell 下，先退出 root：
```bash
exit
```

- 如果你在 SSH 連線的 session 中，登出：
```bash
logout
```

- 重新以 `heroic` 使用者登入（例如使用 SSH）：
```bash
ssh heroic@<你的主機 IP>
```

> 或者直接在本機終端（local desktop / tty）登出後再重新登入 GUI 或使用者帳號。

### 4️⃣ 驗證是否具有 sudo 權限
```bash
sudo whoami
```
若回傳：
```
root
```
代表設定成功 ✅

---

## 🧩 方法二：直接編輯 `/etc/sudoers` 授權

若系統沒有 `sudo` 群組，或想單獨給某人權限：

### 1️⃣ 切換為 root
```bash
su -
```

### 2️⃣ 使用 visudo 開啟設定檔
> 請務必使用 `visudo`，避免語法錯誤造成 sudo 無法使用。
```bash
visudo
```

### 3️⃣ 在檔案末端新增以下一行：
```
heroic  ALL=(ALL:ALL) ALL
```

### 4️⃣ 儲存離開後登出再登入
```bash
exit
logout
ssh heroic@<主機 IP>
```

### 5️⃣ 驗證：
```bash
sudo whoami
```

---

## ⚠️ 方法三（危險）：將 heroic 變成真正的 root（UID=0）

僅供系統損壞、無法 sudo、也無法登入 root 時使用。  
這會讓 `heroic` 變成 **UID=0**，與 root 完全相同。

### 1️⃣ 編輯 `/etc/passwd`
```bash
sudo nano /etc/passwd
```

找到類似的那一行（此處以常見 UID 1000 為例）：
```
heroic:x:1000:1000:heroic:/home/heroic:/bin/bash
```

改成：
```
heroic:x:0:0:heroic:/home/heroic:/bin/bash
```

### 2️⃣ 登出再登入
```bash
logout
ssh heroic@<主機 IP>
```

此時 heroic 就是 root，無需 sudo。

⚠️ **風險警告：**
- 系統安全性完全喪失。
- 所有 root 檔案會被 heroic 修改。
- 建議修復 sudo 權限後立刻把 `/etc/passwd` 改回原本的 UID（例如 1000）。

---

## 🧭 常見補充說明（為何需要登出再登入？）

- Linux 使用者的群組資訊（groups）通常是在登入時由登入管理程式（PAM / login / sshd）讀入。如果你在已登入的 session 裡修改了使用者所屬的群組，系統不會自動在現有 session 更新這些資訊。因此必須登出並重新登入，讓新的群組權限生效。
- 如果你不方便完整登出，可使用 `newgrp` 臨時切換到新群組（僅對當前 shell 有效）：
```bash
newgrp sudo
```
但通常建議完整登出再登入以避免遺漏。

---

## 🧾 總結對照表

| 目標 | 方法 | 登出必要性 | 安全性 | 備註 |
|------|-------|-------------|--------|------|
| 正常升級權限 | ✅ 方法一：加入 sudo 群組 | ✅ | ★★★★★ | 最常見、建議方式 |
| 單一使用者授權 | 方法二：編輯 sudoers | ✅ | ★★★★☆ | 適合單用戶且需精細控制 |
| 強制取得 root | 方法三：改 UID=0 | ✅ | ★☆☆☆☆ | 僅限救援用，完成後務必回復 |

---

如果你要我把這份文件存成 `grant-root-access-linux.md` 並提供下載，或要我改成英文雙語版本，我可以馬上幫你處理。
