# 如何使用 GitHub 進行個人 Scrum 管理

這份文件定義了如何利用 GitHub 內建功能執行個人進修的 Scrum 流程，目標是建立具備大廠實戰感（如 Google, Airwallex）的開發軌跡。

## 一、 建立 Scrum 看板 (GitHub Projects)

1. **進入設定**：登入 GitHub，點擊右上角個人頭像 → **Your projects**。
2. **建立專案**：點擊 **New project** → 選擇 **Board** 模板。
3. **命名規範**：`senior-backend-advanced-roadmap` (全小寫並以破折號分隔)。
4. **自定義欄位 (Columns)**：
   - **Backlog**：存放所有長期想進修的技術點。
   - **Sprint Backlog**：本兩週 (Sprint) 承諾完成的任務。
   - **In Progress**：開發、Debug 或研究中。
   - **In Review**：已完成開發，正在撰寫技術筆記或準備讀書會 Demo。
   - **Done**：正式完成並合併。

## 二、 任務與進度連結 (Issues & Milestones)

Scrum 的核心在於將抽象的目標轉化為可執行的任務（Issues），並設定週期（Milestones）。

### 1. 建立 Sprint 週期 (Milestones)
- 至專案 Repository → **Issues** → **Milestones** → **New milestone**。
- **標題**：`Sprint 1: Java Foundations & Payment Logic` (2026/01/19 - 02/01)。
- **截止日期**：2026-02-01。

### 2. 建立開發任務 (Issues)
- 建立 Issue 時，務必在右側選單設定：
  - **Projects**：連結至你的 Scrum 看板。
  - **Milestone**：連結至當前的 Sprint 週期。
- **標題規範**：使用 `feat:`, `research:`, `docs:` 等前綴增加專業感。

## 三、 自動化工作流 (GitHub Automation)

透過自動化指令，讓你在寫程式的同時更新看板狀態，這是面試時展示「開發效率」的利器。

1. **自動移動卡片**：當你在 Commit Message 或 Pull Request (PR) 描述中寫入 `closes #Issue編號` 時，該卡片會在 PR 被 Merge 時自動從 **In Progress** 移至 **Done**。
2. **指令範例**：
   ```bash
   git commit -m "feat: implement redis distributed lock (closes #12)"