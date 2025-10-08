
# 為什麼架設 GitLab 並實作 CI/CD 建議使用 VMware 而非 Hyper-V

本文件將詳細說明在企業內部架設 GitLab 並搭配 CI/CD 時，為何推薦使用 **VMware（橋接模式）** 而不是 Hyper-V（Default Switch NAT 模式）。

---

## 一、背景與目的

在企業內部，GitLab 通常部署於內網伺服器，用於管理程式碼與觸發 CI/CD Job。當我們使用虛擬機作為 GitLab Runner 或測試環境時，網路連線品質直接影響 Job 效率與穩定性。

此時選擇正確的虛擬化架構與網路模式極為關鍵。

---

## 二、VMware vs Hyper-V 封包路徑比較

### 🟥 Hyper-V（Default Switch / NAT 模式）

```text
[VM (172.17.x.x)]
  ↓
[Default Switch vSwitch]
  ↓
[主機 WinNAT 模組]
  ↓
[主機實體網卡 (Wi-Fi/Ethernet)]
  ↓
[區網 GitLab Server]
```

**特性與問題：**

- VM 使用 NAT 模式連出
- 所有封包都經由主機轉發（尤其回傳封包經 WinNAT 負責）
- 容易造成 clone 卡住在 `Receiving objects`
- 無法從主機直接訪問 VM（除非額外設 Port Forwarding）
- 大量 TCP 回傳會導致緩衝堆積、吞吐瓶頸

---

### 🟩 VMware（橋接模式 Bridged）

```text
[VM (192.168.x.x)]
  ↓
[區網交換器 / 路由器]
  ↓
[區網 GitLab Server (192.168.x.x)]
```

**特性與優點：**

- VM 獲得與主機相同網段 IP（192.168.x.x）
- 直接參與區網，封包無需轉送或 SNAT
- GitLab webhook 可直接觸發 VM Job
- Git clone 速度可達 5~20MB/s
- VM 可雙向與其他機器溝通（適合微服務、測試 API）

---

## 三、CI/CD 實際應用場景需求

| 功能 | 對網路需求 |
|------|------------|
| GitLab webhook 觸發 Job | VM 需可被主機/伺服器訪問 |
| Git clone + pull + push | 雙向高速連線 |
| 執行自動化測試 | 穩定連接 API / 資料庫 |
| Docker pull/push | 封包大、需高頻率穩定傳輸 |
| 上傳 artifact / deploy | 雙向快速存取，不能 NAT 掉封包 |

---

## 四、Hyper-V 的限制

- Default Switch 是黑盒 NAT 架構，無法自訂網段
- 封包全部經過主機處理，負擔重
- clone Git repo 容易卡住在 5% / receiving
- 無法模擬 LAN 架構，無法測試微服務/內部連線
- 無法提供服務給 GitLab（Webhook 打不到）

---

## 五、VMware 的優勢（橋接模式）

| 優點 | 說明 |
|------|------|
| LAN 實體等級封包傳輸 | 無 NAT，封包不經主機處理 |
| VM = LAN 一員 | 可做 webhook receiver, API provider |
| GitLab 整合最佳 | clone 快速、CI/CD 穩定 |
| Docker registry 相容 | push/pull 都不會卡 |
| 支援多 VM 架構 | dev/staging/prod 可各自部署模擬 |

---

## 六、共存問題：VMware 與 Hyper-V 可同時使用嗎？

預設情況下 ❌ 不行。VMware 需要獨佔 VT-x 虛擬化功能，而 Hyper-V 啟用後會鎖住這些資源。

### 解法：切換 Hyper-V 開關

#### 關閉 Hyper-V（讓 VMware 正常）

```powershell
bcdedit /set hypervisorlaunchtype off
```
重開機後，Hyper-V 關閉，VMware 可正常使用。

#### 開啟 Hyper-V（回復原本狀態）

```powershell
bcdedit /set hypervisorlaunchtype auto
```

---

## 七、建議 CI/CD 架構

```text
[GitLab Server] (192.168.50.10)
      ↑
┌────────────┐
│ VM1 - Runner (192.168.50.101) │ ← VMware 橋接模式
│ VM2 - Test API (192.168.50.102)
│ VM3 - Web UI   (192.168.50.103)
└────────────┘
```

- 所有 VM 與 GitLab Server 同區網，穩定、低延遲
- 適合多容器模擬、微服務整合測試
- 單機即可模擬完整 DevOps 環境

---

## 八、總結

| 面向 | Hyper-V Default Switch | VMware 橋接模式 |
|------|------------------------|------------------|
| 網路模式 | NAT                  | LAN 實體直連     |
| VM IP 類型 | 172.x.x.x           | 192.168.x.x      |
| Git clone 效能 | 慢，常卡住        | 快，穩定        |
| 可否接收 webhook | 否               | 是              |
| 封包轉送負擔 | 高（經主機）       | 低（直連）      |
| 適合 CI/CD | ❌                | ✅              |

---

## ✅ 建議使用方式

- 若你目標是：**打造內部 CI/CD 測試環境（GitLab + 多台測試 VM）**
- 請使用 **VMware + 橋接模式**
- 若仍要用 Hyper-V，則務必改為 External Switch + 有線網卡
