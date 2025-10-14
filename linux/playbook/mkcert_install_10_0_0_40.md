# mkcert 在 Linux 虛擬機 (VM) 上的安裝與使用教學

> 目標：在 *Linux VM* 上安裝 `mkcert`，建立本地受信任的 CA，並產生給 IP `10.0.0.40` 的 TLS 憑證（`cert` 與 `key`），最後示範放到 Nginx 使用與驗證步驟。

---

## 版本與前置需求

- 本教學針對常見 Linux 發行版（Ubuntu / Debian / CentOS / Rocky / Alma）提供步驟。
- 建議在 **虛擬機** 本機端（有瀏覽器或可存取系統信任庫）執行 `mkcert -install`，若 VM 是 headless，可參考「手動安裝 rootCA」章節。
- 需要具備 `sudo` 權限以安裝系統套件與設定檔位置（若無 sudo，請以 root 執行相應命令）。

---

## 1. 安裝 mkcert（通用流程）

### 1.1 安裝必要工具

**Ubuntu / Debian（示範）**

```bash
sudo apt update
sudo apt install -y libnss3-tools wget ca-certificates openssl
```

**CentOS / RHEL / Rocky / AlmaLinux（示範）**

```bash
sudo yum install -y nss-tools wget ca-certificates openssl
# 或（新版系統）
# sudo dnf install -y nss-tools wget ca-certificates openssl
```

> `libnss3-tools`（或 nss-tools）是讓 mkcert 能夠同時安裝到 Firefox 的信任庫。


### 1.2 下載 mkcert 二進位檔（穩健做法）

將 `mkcert` 放到 `/usr/local/bin`（或其他 PATH）並加入執行權限。

```bash
# 範例：下載特定版本（請以實際最新版替換 v1.4.4）
MKCERT_VER=v1.4.4
wget https://github.com/FiloSottile/mkcert/releases/download/${MKCERT_VER}/mkcert-${MKCERT_VER}-linux-amd64 -O mkcert
chmod +x mkcert
sudo mv mkcert /usr/local/bin/mkcert

# 確認安裝
mkcert -version
```

> 若你的系統是 ARM（例如 Raspberry Pi / 某些 VM），請下載對應架構的二進位檔。

---

## 2. 建立本地 CA 並安裝到系統（信任）

> 注意：`mkcert -install` 會在使用者主目錄建立一組 local CA（rootCA），並嘗試把它安裝到系統信任庫與瀏覽器（Windows / macOS / Linux）的信任存放區。

```bash
# 以目前使用者執行（不要用 sudo 直接以 root 建立，除非你知道自己在做什麼）
mkcert -install
```

- 成功後會輸出 root CA 的位置（例如 `~/.local/share/mkcert` 或 `~/.local/share/mkcert/rootCA.pem`）。
- 若 VM 是 headless（沒有 GUI / 沒有瀏覽器），`mkcert -install` 可能無法自動將 CA 加入到所有環境，此時請參閱「手動安裝 rootCA 到系統信任」章節。

---

## 3. 為 IP `10.0.0.40` 產生憑證

在你執行 `mkcert -install` 的同一個使用者下執行下列命令：

```bash
# 產生 cert 與 key（檔名可自訂）
mkcert -cert-file 10.0.0.40.pem -key-file 10.0.0.40-key.pem 10.0.0.40
```

執行後會在當前資料夾產生：

- `10.0.0.40.pem`（公用憑證）
- `10.0.0.40-key.pem`（私密金鑰）

> 備註：`mkcert` 會自動把 SAN（Subject Alternative Name）塞到憑證中，所以瀏覽器可以信任直接用 IP 存取的 HTTPS（前提是該瀏覽器信任 mkcert 的 rootCA）。

---

## 4. 將憑證移到伺服器標準路徑（Nginx 範例）

建議的檔案位置（僅範例，可依專案調整）：

- 憑證：`/etc/nginx/cert/10.0.0.40.pem`
- 私鑰：`/etc/nginx/cert/10.0.0.40-key.pem`

```bash
sudo mkdir -p /etc/nginx/cert
sudo mv 10.0.0.40.pem /etc/nginx/cert/10.0.0.40.pem
sudo mv 10.0.0.40-key.pem /etc/nginx/cert/10.0.0.40-key.pem
sudo chown root:root /etc/nginx/cert/10.0.0.40.pem /etc/nginx/cert/10.0.0.40-key.pem
sudo chmod 644 /etc/nginx/cert/10.0.0.40.pem
sudo chmod 600 /etc/nginx/cert/10.0.0.40-key.pem
```

> 說明：公鑰檔設 `644` 讓系統服務（如 Nginx）可讀；私鑰檔設 `600` 以確保只有 root 可以讀取（或設定為執行 Nginx 的使用者可讀）。

---

## 5. Nginx 設定範例（最小 HTTPS server block）

假設你要綁定到 `10.0.0.40:443`：

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name 10.0.0.40;

    ssl_certificate     /etc/nginx/cert/10.0.0.40.pem;
    ssl_certificate_key /etc/nginx/cert/10.0.0.40-key.pem;

    # 安全性建議（可依需求調整）
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

儲存後重新載入 Nginx：

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 6. 驗證（本機與遠端瀏覽器）

### 6.1 在 VM 上檢查憑證資訊

```bash
openssl x509 -in /etc/nginx/cert/10.0.0.40.pem -noout -text | sed -n '1,120p'
```

### 6.2 使用 curl 驗證（若憑證在本機信任）

```bash
# 如果瀏覽器/系統已信任 mkcert 的 rootCA，直接：
curl -v https://10.0.0.40/

# 若未信任，可使用 rootCA 檔做測試（替換為實際 rootCA 路徑）
curl --cacert ~/.local/share/mkcert/rootCA.pem https://10.0.0.40/
```

### 6.3 在開發者電腦（或其他機器/瀏覽器）信任 rootCA

- 若要讓其他機器的瀏覽器信任這張憑證，需把 mkcert 所生成的 `rootCA.pem` 安裝到該機器系統或瀏覽器的信任根（CA）中。

常見作法：

```bash
# 找到 mkcert 的 rootCA（在執行 mkcert 的使用者下）
ls -l ~/.local/share/mkcert
# 或
ls -l /home/<user>/.local/share/mkcert

# 把 rootCA 複製到要信任的機器，然後安裝到系統（Debian/Ubuntu 範例）
sudo cp rootCA.pem /usr/local/share/ca-certificates/mkcert-rootCA.crt
sudo update-ca-certificates
```

> Firefox: 若使用 Firefox（套件庫獨立），可能需要透過 `certutil` 安裝到 Firefox 的 NSS store，或在 GUI 的憑證管理手動匯入。


---

## 7. Headless VM / 無法自動安裝 rootCA（手動安裝方法）

如果你是在沒有 GUI / 無法執行 `mkcert -install` 的環境，請在能執行 mkcert 的機器上（例如你本地的開發機）執行 `mkcert -install`，然後把該機器的 rootCA（`rootCA.pem` 與 `rootCA-key.pem`）複製到 VM，將 `rootCA.pem` 安裝到系統信任庫：

Ubuntu/Debian：

```bash
sudo cp rootCA.pem /usr/local/share/ca-certificates/mkcert-rootCA.crt
sudo update-ca-certificates
```

CentOS/RHEL：

```bash
sudo cp rootCA.pem /etc/pki/ca-trust/source/anchors/mkcert-rootCA.crt
sudo update-ca-trust extract
```

> 手動安裝後，瀏覽器（或 curl）就會信任由該 rootCA 簽發的憑證。

---

## 8. 常見問題與排錯

- **問題：瀏覽器仍顯示不受信任的憑證**
  - 原因：瀏覽器沒有安裝 mkcert 的 root CA，或安裝到錯誤的 store（例如只安裝到單一使用者但使用者不同）。
  - 解法：確認 `mkcert -CAROOT` 找到 rootCA 的實際位置，並把 `rootCA.pem` 安裝到客戶端系統/瀏覽器信任根。示範：`mkcert -CAROOT`。

- **問題：mkcert 在 headless server 上 `-install` 失敗**
  - 解法：在有 GUI 的機器執行 `mkcert -install`，把 rootCA 複製到 server，參照上面手動安裝指令。

- **問題：Nginx 無法讀取私鑰權限錯誤**
  - 確認私鑰權限與擁有者（例如 Nginx 以 `www-data` 或 `nginx` 使用者執行時，該使用者必須可以讀取私鑰；或將私鑰屬主改為 root 並讓 Nginx 以 root 讀後再降權運行）。

- **問題：要在多台機器使用同一張憑證？**
  - 注意私鑰暴露風險。若多台主機需要 HTTPS，建議每台主機各自使用 mkcert 產生自己的憑證或在內部 CA 下發。若必須複製，請確保私鑰傳輸安全（scp + ssh），並設嚴格檔案權限。

---

## 9. 範例完整命令（從零到尾）

```bash
# 1. 在 Ubuntu VM 安裝套件
sudo apt update
sudo apt install -y libnss3-tools wget ca-certificates openssl

# 2. 下載 mkcert（示範版本，請替換為最新）
MKCERT_VER=v1.4.4
wget https://github.com/FiloSottile/mkcert/releases/download/${MKCERT_VER}/mkcert-${MKCERT_VER}-linux-amd64 -O mkcert
chmod +x mkcert
sudo mv mkcert /usr/local/bin/mkcert

# 3. 建立並安裝本地 CA
mkcert -install

# 4. 產生 10.0.0.40 憑證
mkcert -cert-file 10.0.0.40.pem -key-file 10.0.0.40-key.pem 10.0.0.40

# 5. 把檔案移到 nginx 使用的路徑並設定權限
sudo mkdir -p /etc/nginx/cert
sudo mv 10.0.0.40.pem /etc/nginx/cert/10.0.0.40.pem
sudo mv 10.0.0.40-key.pem /etc/nginx/cert/10.0.0.40-key.pem
sudo chown root:root /etc/nginx/cert/10.0.0.40.*
sudo chmod 644 /etc/nginx/cert/10.0.0.40.pem
sudo chmod 600 /etc/nginx/cert/10.0.0.40-key.pem

# 6. 測試 nginx 設定並重新載入
sudo nginx -t && sudo systemctl reload nginx

# 7. 驗證（可在本機或客戶端使用）
openssl x509 -in /etc/nginx/cert/10.0.0.40.pem -noout -text | sed -n '1,80p'
curl --cacert $(mkcert -CAROOT)/rootCA.pem https://10.0.0.40/
```

---

## 10. 安全與清理（備註）

- 若你不再需要 local CA，可用下面指令移除 mkcert 安裝的 CA（僅會移除 mkcert 安裝到系統/瀏覽器的信任，不會刪除憑證檔案）：

```bash
mkcert -uninstall
```

- 若要完全刪除 mkcert 產生的 CA 檔，刪除 `mkcert -CAROOT` 回傳的資料夾（小心操作）

```bash
# 先查 CAROOT
mkcert -CAROOT
# 假設回傳 ~/.local/share/mkcert，刪除
rm -rf ~/.local/share/mkcert
```

---

## 附錄：常用 mkcert 小技巧

- `mkcert -CAROOT`：顯示 root CA 的儲存位置。
- `mkcert -install`：建立並安裝 root CA。
- `mkcert example.com 127.0.0.1 ::1`：同時為多個 hostname/IP 產生憑證。
- 若你希望憑證檔名自帶 `-key`（mkcert 預設會輸出 `-key.pem`），可使用 `-cert-file` / `-key-file` 自訂檔名。

---
