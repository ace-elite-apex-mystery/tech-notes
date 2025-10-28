# 現代 CPU 多核心排程與核心綁定技術全解析

## 一、為什麼電腦要在不同核心上輪巡，而不是讓一件事完全做完？

### 1. CPU 與 I/O 的速度差距

CPU 執行速度極快（奈秒級），但記憶體與 I/O 操作是微秒～毫秒級，若讓 CPU
等待 I/O，效率會極度浪費。
因此現代作業系統讓多個任務「輪流執行（time-slicing）」以充分利用 CPU
時間。

### 2. 作業系統的排程器 (Scheduler)

-   每個任務獲得短暫的「時間片」。
-   Scheduler 定期進行「context switch（上下文切換）」。
-   使用者體驗上會覺得多程式同時執行。

### 3. 多核心設計的考量

-   現代 CPU 擁有多核心，但任務通常遠多於核心數。
-   Scheduler 會動態分配任務到各核心。
-   同時執行 + 輪巡調度，確保整體流暢與公平。

### 4. Context Switching 的成本

雖然切換任務需保存暫存器與狀態，但其代價（奈秒級）遠小於 I/O
等待（毫秒級），因此整體效能仍提升。

### 5. 若不輪巡會發生什麼？

-   其他任務被凍結，系統失去即時反應。
-   滑鼠、鍵盤、音樂播放會卡頓。
-   系統體驗大幅下降。

------------------------------------------------------------------------

## 二、那麼，能不能讓重要任務在單一核心上跑完？

可以，這就是 **CPU Affinity（核心親和性）** 或 **CPU
Pinning（核心綁定）**。

### 優點

  優點           說明
  -------------- ------------------------
  Cache 效率高   避免快取重新載入
  穩定延遲       適合即時系統
  減少搶佔       減少 context switching
  可預測性高     行為穩定、延遲可控

### 適用場景

-   即時音訊處理（DAW、錄音）\
-   遊戲主執行緒 / 物理引擎\
-   高頻交易系統（HFT）\
-   工控、嵌入式系統

### Linux 設定

``` bash
taskset -c 2 ./my_program
```

或程式碼中設定：

``` c
cpu_set_t mask;
CPU_ZERO(&mask);
CPU_SET(2, &mask);
sched_setaffinity(0, sizeof(mask), &mask);
```

### Windows 設定

``` c
SetThreadAffinityMask(GetCurrentThread(), 1 << 2);
```

### Java 容器設定

``` yaml
resources:
  limits:
    cpu: "1"
  requests:
    cpu: "1"
```

------------------------------------------------------------------------

## 三、前 6 大語言在系統級 CPU 操作的比較

  語言         是否能做 CPU 綁定    難度       常見場景
  ------------ -------------------- ---------- ----------------------
  **C**        ✅ 完全可行          ⭐⭐⭐     核心模組、驅動、RTOS
  **C++**      ✅ 可行              ⭐⭐       遊戲引擎、HFT
  **Java**     ⚠️ JNI / 外部指令    ⭐⭐⭐⭐   金融、通訊
  **Python**   ❌ 不支援直接控制    ⭐⭐⭐⭐   腳本、自動化
  **Go**       ⚠️ 不可手動控制      ⭐⭐       伺服器、高併發
  **Rust**     ✅ 透過 crate 實現   ⭐⭐       系統程式、安全並行

### C 範例

``` c
#define _GNU_SOURCE
#include <pthread.h>
#include <sched.h>
#include <stdio.h>

void *worker(void *arg) { while (1) { /* do something */ } return NULL; }

int main() {
    pthread_t thread;
    pthread_create(&thread, NULL, worker, NULL);
    cpu_set_t mask;
    CPU_ZERO(&mask);
    CPU_SET(2, &mask);
    pthread_setaffinity_np(thread, sizeof(mask), &mask);
    pthread_join(thread, NULL);
}
```

### Rust 範例

``` rust
use core_affinity::set_for_current;

fn main() {
    let cores = core_affinity::get_core_ids().unwrap();
    set_for_current(cores[2]);
    loop { /* 任務邏輯 */ }
}
```

------------------------------------------------------------------------

## 四、Java 的特殊情況：可以，但不常見

### 限制

  限制                  原因
  --------------------- --------------------------------------------
  JVM 自行調度 thread   JVM 可能在 GC 或 safepoint 暫停全部 thread
  無官方 API            必須使用 JNI / JNA / native binding
  GC 中斷               即使綁定核心也可能被 GC 暫停
  跨平台抽象            Java 不暴露核心 ID / 排程策略

### 但仍有使用的產業

#### 1. 高頻交易 (HFT)

-   使用 LMAX Disruptor、Chronicle Queue
-   常透過 JNI 呼叫 `sched_setaffinity()` 或使用 `taskset`
-   tuning GC（如 ZGC / Epsilon GC）
-   減少 jitter、微秒級延遲

#### 2. 通訊系統

-   Telecom 核心系統（VoIP、Message routing）
-   業務層用 Java，底層透過 JNI 控制排程

#### 3. 即時資料處理

-   Kafka、Flume、Chronicle Threads
-   使用 Docker / K8s 限制 CPU core

#### 4. 遊戲伺服器 / 實時模擬

-   主邏輯固定 CPU0，I/O 處理分核心
-   搭配 `-XX:+UseNUMA` 優化

### 常見工具

  工具 / 套件         功能
  ------------------- ------------------------------
  OpenHFT Affinity    Java 設定 Thread 與 CPU 綁定
  JNI / JNA           呼叫底層 C 系統函數
  Chronicle Threads   提供 Thread pinning API
  taskset             系統層面綁定 JVM
  Docker CPU 限制     容器層面控制可用核心

### 範例：OpenHFT Affinity

``` java
AffinityLock al = AffinityLock.acquireLock();
try {
    // 任務固定於當前核心執行
    doWork();
} finally {
    al.release();
}
```

------------------------------------------------------------------------

## 五、總結

  指標               Java 狀況                       備註
  ------------------ ------------------------------- ------
  是否能綁核心       ✅ 可行，但需 JNI / 外部工具    
  是否常見           ❌ 少數高頻交易與通訊系統使用   
  難度               高，需理解 JVM 與 OS 關係       
  效果               微秒級優化明顯                  
  一般應用是否建議   不建議，可能破壞 JVM 調度效能   

------------------------------------------------------------------------

## 六、關鍵觀念回顧

  主題               核心觀念
  ------------------ -----------------------------------
  為何要輪巡         提高 CPU 利用率、即時反應、公平性
  為何要綁核心       提升延遲穩定性與 cache 效率
  哪些語言常用       C / C++ / Rust
  Java 是否能做      可以，但非主流
  哪些產業會這樣做   高頻交易、通訊核心、即時模擬

------------------------------------------------------------------------

**總結一句話：** \> 「一般應用靠排程器讓系統整體流暢，\
\> 高性能應用則靠核心綁定追求穩定與極限效能。」

## 若要精確分類成層級關係（學術結構）
Computer Science
 ├── Systems
 │    ├── Operating Systems
 │    │    ├── Process Management
 │    │    │    ├── CPU Scheduling
 │    │    │    ├── Context Switching
 │    │    │    └── CPU Affinity (Core Binding)
 │    ├── Concurrency & Parallelism
 │    └── Real-Time Systems
 ├── Performance Engineering
 │    ├── Low-Latency Optimization
 │    ├── Cache & Memory Tuning
 │    └── NUMA / Thread Pinning
 └── Software Engineering
      └── JVM / Runtime Performance (for Java)
