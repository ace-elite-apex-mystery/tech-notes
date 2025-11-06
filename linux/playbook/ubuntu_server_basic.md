# 🐳 連線linux系統
## 📍 1. 查詢 Ubuntu 伺服器 IP 位址

安裝完成後登入 Ubuntu 終端機，執行以下指令：

```bash
ip addr
```

你會看到類似以下輸出：

```
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    inet 172.17.53.138/20 brd 172.17.63.255 scope global dynamic ens33
```

- 介面名稱：`ens33`  
- **IP 位址**：`172.17.53.138` ← ✅ 這就是你要連線的 IP  
- 若看不到 `inet`，表示尚未連上網路，請檢查 VMware 是否設定為「Bridged 模式」。

---

## 🔐 2. 從 Windows 使用 SSH 連線進入 Ubuntu

### ✅ 使用 PowerShell 或 CMD
在 Windows 上開啟 **PowerShell**，輸入以下指令（將帳號與 IP 改成你的）：

```bash
ssh heroic@172.17.53.138
```

第一次登入會提示：

```
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

輸入：
```
yes
```

然後再輸入你在安裝 Ubuntu 時設定的密碼。  
登入成功後你會看到類似：

```
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-31-generic x86_64)
heroic@matsu-language-exam:~$
```

恭喜 🎉 你已經從 Windows 成功遠端登入 Ubuntu！

---

## ⚙️ 3. 讓使用者執行 `sudo` 時不再輸入密碼

### 🔧 方法：一行命令永久生效，無需手動編輯

輸入以下指令即可完成設定：

```bash
echo "$(whoami) ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$(whoami)
```

這行指令會：
- 自動偵測目前使用者名稱；
- 在 `/etc/sudoers.d/` 下建立對應設定；
- 讓該使用者能執行任何 `sudo` 指令而不需輸入密碼。

---

### 🧪 測試設定是否成功

執行：
```bash
sudo ls /root
```

如果沒有要求你輸入密碼 ✅  
代表設定成功。

---

### ♻️ 若要恢復原狀

若想重新要求密碼，只要刪除設定檔即可：
```bash
sudo rm /etc/sudoers.d/$(whoami)
```

---

## 🧩 4. 常見問題（FAQ）

| 問題 | 解法 |
|------|------|
| SSH 連不上 | 確認 Ubuntu 的防火牆已開放 SSH：<br>`sudo ufw allow 22 && sudo ufw enable` |
| IP 每次重開都變 | 設定固定 IP（修改 `/etc/netplan/00-installer-config.yaml`） |
| WinSCP 無法登入 | 確認帳號、密碼正確，並使用 Port 22 |
| 貼上指令無法使用 Ctrl+V | 在 VMware Console 中用「右鍵 → Paste」或改用 SSH 工具登入 |

---

## ✅ 總結

| 操作 | 指令 |
|------|------|
| 查 IP | `ip addr` |
| SSH 登入 | `ssh user@IP` |
| 免 sudo 密碼 | `echo "$(whoami) ALL=(ALL) NOPASSWD:ALL" \| sudo tee /etc/sudoers.d/$(whoami)` |

---

## 💡 延伸閱讀
- [Ubuntu 官方文件：OpenSSH Server](https://ubuntu.com/server/docs/service-openssh)
- [VMware 官方文件：設定 Bridged Network](https://docs.vmware.com/)
- [WinSCP 官網下載](https://winscp.net/eng/download.php)

---

> 🧩 小提示：  
> 若你是為開發或內網測試使用（如考試系統伺服器），  
> 可以安全地設定免密碼 sudo。  
> 若是正式上線環境，建議仍保留 sudo 密碼或改用 SSH Key 認證。
