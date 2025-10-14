# ⚠️ 修改 /etc/passwd 為 heroic:x:0:0: 造成的問題與修復指南

## 📘 前言
在 Linux 系統中，`/etc/passwd` 檔案負責定義系統中所有使用者帳號的基本資訊。  
若誤將一般使用者（例如 `heroic`）的 UID 與 GID 都改為 `0:0`，  
會導致系統辨識混亂、權限異常、甚至無法登入或執行 `sudo`。  

本文將說明這樣的設定代表什麼、會造成哪些問題、以及如何完整修復。

---

## 🧩 一、/etc/passwd 的結構說明

`/etc/passwd` 中的每一行格式如下：
```
使用者名稱:密碼佔位符:UID:GID:描述:家目錄:登入 Shell
```

範例：
```
root:x:0:0:root:/root:/bin/bash
heroic:x:1000:1000:heroic:/home/heroic:/bin/bash
```

其中：
- `UID`：使用者 ID（User ID）
- `GID`：主要群組 ID（Group ID）
- `0` 表示 **root 等級權限**（系統最高權限）

---

## 🚨 二、錯誤設定：heroic:x:0:0

如果誤將 heroic 設成：
```
heroic:x:0:0:heroic:/home/heroic:/bin/bash
```

代表：
- 你讓 heroic 擁有與 root 相同的權限（UID 0 = root）。
- 系統中出現兩個 UID=0 的帳號（`root` 與 `heroic`）。
- 對於 PAM、NSS、sudo、login 等安全模組而言，**這是致命錯誤**。

---

## 🧨 三、會出現的錯誤與症狀

1. **sudo 無法使用**
   ```
   sudo: you do not exist in the passwd database
   ```
   原因：系統內部帳號驗證時找不到對應 UID 的安全紀錄。

2. **無法 su 進入 root**
   ```
   su: Authentication failure
   ```
   因為 root 沒有設定密碼、而 heroic 已取代 UID=0 導致 PAM 無法識別。

3. **無法登入（tty1 或 ssh）**
   ```
   Login incorrect
   ```
   因為 login 模組無法正確讀取使用者紀錄。

4. **部分系統服務啟動失敗**
   - Docker、systemd、journald 可能無法辨識執行者。
   - 使用者環境變數、家目錄權限錯亂。

---

## 🧭 四、為什麼會出錯

Linux 系統並非僅用 UID 判斷權限，還會透過 PAM（Pluggable Authentication Modules）  
確認使用者是否存在於安全資料庫中（`/etc/passwd`, `/etc/shadow` 等）。  
當系統內出現兩個 UID=0 時，會導致 PAM 驗證結果不一致。

---

## 🛠️ 五、正確的修復方式

### ✅ 方法一：透過 Recovery Mode 進入 root shell

1. **重啟系統**，在開機時連按 `Shift` 或 `Esc` 進入 GRUB。  
2. 選擇：
   ```
   Advanced options for Ubuntu
   ```
3. 接著選：
   ```
   Ubuntu, with Linux (recovery mode)
   ```
4. 進入 Recovery Menu 後，選：
   ```
   root - Drop to root shell prompt
   ```
5. 系統會進入 root shell。先讓檔案系統可寫：
   ```bash
   mount -o remount,rw /
   ```

6. 備份並修正 `/etc/passwd`：
   ```bash
   cp /etc/passwd /etc/passwd.bak_$(date +%s)
   sed -i 's/^heroic:x:0:0:/heroic:x:1000:1000:/' /etc/passwd
   chown -R heroic:heroic /home/heroic
   ```

7. 重設密碼：
   ```bash
   passwd heroic
   passwd root
   ```

8. 重開機：
   ```bash
   reboot
   ```

---

### ✅ 方法二：使用 Live USB 修復（若無法進入 Recovery Mode）

1. 用另一台機器製作 Ubuntu Live USB。
2. 從該 USB 啟動 → 選「Try Ubuntu」。  
3. 掛載系統磁碟：
   ```bash
   sudo mkdir /mnt/rootfs
   sudo mount /dev/sda1 /mnt/rootfs    # /dev/sda1 換成實際根分割區
   sudo chroot /mnt/rootfs
   ```
4. 修復 `/etc/passwd`：
   ```bash
   sed -i 's/^heroic:x:0:0:/heroic:x:1000:1000:/' /etc/passwd
   chown -R heroic:heroic /home/heroic
   passwd heroic
   exit
   sudo reboot
   ```

---

## 🧰 六、防止問題再發生

1. **永遠不要直接編輯 `/etc/passwd` 來改 UID/GID。**  
   改用：
   ```bash
   sudo usermod -u 1001 username
   sudo groupmod -g 1001 username
   ```

2. **讓使用者擁有 root 權限的正確做法：**
   ```bash
   sudo usermod -aG sudo heroic
   sudo visudo
   ```
   修改：
   ```
   %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
   ```
   ✅ 這樣 heroic 仍是一般使用者，但可直接執行所有 root 指令（安全又穩定）。

---

## 🧾 七、修復後檢查

登入後檢查：
```bash
id heroic
```

應該顯示：
```
uid=1000(heroic) gid=1000(heroic) groups=1000(heroic),sudo
```

再確認：
```bash
sudo docker ps
```
能成功執行表示系統已完全恢復。

---

## 🏁 八、結論

| 狀態 | 結果 |
|------|------|
| heroic:x:0:0 | ⚠️ 系統錯亂、無法登入 |
| heroic:x:1000:1000 | ✅ 正常一般使用者 |
| 加入 sudo 群組 + NOPASSWD | ✅ 擁有 root 權限、安全穩定 |

---

## 📚 參考資料
- [The Linux man-page for passwd(5)](https://man7.org/linux/man-pages/man5/passwd.5.html)
- [Ubuntu Wiki - RootSudo](https://help.ubuntu.com/community/RootSudo)
- [GNU Coreutils Documentation](https://www.gnu.org/software/coreutils/manual/coreutils.html)

---


