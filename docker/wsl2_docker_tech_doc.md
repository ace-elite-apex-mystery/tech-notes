# WSL2 + Docker Desktop 技術文件（完整版）

## 1. VT-x 是什麼？
VT-x（Intel Virtualization Technology）是 CPU 的硬體虛擬化功能，讓系統能啟動虛擬機（WSL2、Docker Desktop、Hyper-V 等）。沒有啟用 VT-x，Docker Desktop 與 WSL2 無法運作。

## 2. Hyper-V 是否內建？
- Windows 10/11 Pro / Enterprise：✔ 有 Hyper-V
- Windows 10/11 Home：❌ 沒有 Hyper-V  
因此多數家用/學校電腦無 Hyper-V → Docker Desktop 只能使用 WSL2。

## 3. WSL2 是否內建？
WSL2 屬於 Windows 內建功能，但預設未啟用，需要手動：
1. 開啟 WSL 功能  
2. 開啟 Virtual Machine Platform  
3. 安裝 WSL2 Kernel（MSI）  
4. 設定 `wsl --set-default-version 2`

## 4. 為什麼要安裝 Linux Kernel？
WSL2 本質是一台真正的 Linux VM，需要 Linux Kernel 才能運作。啟用 WSL ≠ 安裝 Linux Kernel。

## 5. 檔案總管為何會出現「Linux」？
只有在：
- 啟用 WSL  
- 裝好 WSL2 Kernel  
- 安裝 **至少一個 Linux distro（例：Ubuntu）**  
才會出現「Linux」。代表你成功有 WSL2 環境。

## 6. VERSION=2 才是 WSL2？
是。`wsl -l -v` 必須看到：
```
Ubuntu          VERSION 2
docker-desktop  VERSION 2
```
才表示成功啟動 WSL2。

## 7. `*` 代表什麼？
代表預設啟動的 WSL 發行版（default distro）。Ubuntu 有 * 是正常的，不影響 Docker。

## 8. WSL2 會占 CPU 嗎？
不會。開機時全為 `Stopped`，完全不吃 CPU / RAM。  
只有：
- 開 Ubuntu  
- 執行 `wsl`  
- Docker Desktop 啟動  
時才會啟動。

## 9. Docker Desktop 會占資源嗎？
會。有 container 時才會吃 CPU。Idle 時 CPU 0%、RAM 約 200~500MB。  
執行 `wsl --shutdown` 可完全釋放資源。

## 10. 重開機後要手動啟動 WSL 嗎？
❌ 不需要。  
WSL2 是 **按需啟動** 的，只會在需要時自動啟動。

## 11. Docker 是否需要 WSL2？
- Windows Home → ✔ 必須使用 WSL2  
- Windows Pro → 可選 Hyper-V 或 WSL2，但官方推薦 WSL2（更快、更穩、不衝突）。

## 12. 封閉環境完整部署流程
1. 啟用 WSL  
2. 啟用 VM Platform  
3. 重開機  
4. 安裝 WSL2 Kernel  
5. `wsl --set-default-version 2`  
6. 安裝 Ubuntu（可離線）  
7. 安裝 Docker Desktop  
8. 匯入 images（docker load）  
9. `docker compose up -d`

---

以上為完整技術文件，用於 WSL2 與 Docker Desktop 的理解與部署。
