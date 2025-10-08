# VMware linux安裝docker
# 移除舊版（如有）
sudo apt remove docker docker-engine docker.io containerd runc
<br>
# 安裝必要工具 安裝前會先update目錄
sudo apt update 
<br>
sudo apt install ca-certificates curl gnupg lsb-release
<br>
# 加入 Docker GPG 金鑰 需要安裝docker金鑰才可以安裝docker
為什麼要加入 Docker GPG 金鑰？
<br>
簡單說：
<br>
為了讓你的系統相信「從 Docker 官方套件庫下載的檔案是安全且沒被竄改的」。
<br>
sudo mkdir -p /etc/apt/keyrings
<br>
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
<br>
# 查看是否有金鑰
ls -l /etc/apt/keyrings/docker.gpg
<br>
若是出現類似以下則安裝成功
<br>
-rw-r--r-- 1 root root 2800 Jul 11 16:26 /etc/apt/keyrings/docker.gpg
<br>
# 設定 Docker 套件來源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 安裝 Docker
sudo apt update
<br>
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
<br>
# 測試是否成功
docker --version
<br>
docker compose version
<br>
---
**撰寫紀錄：**
| 作者      | 職稱             | 日期       | 備註   |
|----------|-----------------|------------|-------|
| Ian Wang | 資深後端工程師    | 2025/07/10 | 初版   |