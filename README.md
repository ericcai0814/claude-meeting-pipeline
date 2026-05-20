# claude-meeting-pipeline

中文會議錄音 → 結構化紀錄 → 個人 issues 的整合工作流。Claude Code skill plugin。

> **平台限制**：macOS / Apple Silicon only（mlx-whisper 是 Apple MLX framework）。

## What it does

```
你: /meeting-pipeline 「整理 meetings/2026-05-20-sync.m4a」

Claude:
  [第 1 段] 背景跑 transcribe-zh → 產出 .txt / .srt / .vtt / .json
            → 驗證 hallucination（sort | uniq -c）

  [第 2 段] 選 template（教學場 vs 決議場）→ 讀逐字稿前 200 行樣本
            → 產出 meeting-notes.md（with `(?)` 標記不確定）
            → 給你校正人名與 owner

  [第 3 段] 若 repo 有 docs/agents/issue-tracker.md
            → 從校正後的 action items 挑 1-3 個拆 issue
```

整個流程是「**人機協作**」而非「**全自動**」——第 2 段必須由你確認 owner 才進第 3 段，避免 issue 拆給錯誤的人。

## Why

中文會議錄音用 LLM 整理常遇到三個問題：
1. **Whisper 對中英夾雜的轉錄品質不穩**——人名/技術名詞偏移率高（Cloud→Claude、Nebula→Naple）
2. **不同會議類型需求差很多**——training 場要抓「關鍵原則」、sync 場要抓「決議 + action items」，硬套同一份 template 紀錄會走味
3. **拆 issue 是 owner-sensitive 動作**——whisper 不告訴你誰說了什麼，全自動容易拆給錯誤的人

這個 skill 對應的設計：
- **anti-hallucination flags 內建在 transcribe-zh**（不需要每次自己記 mlx-whisper 參數）
- **兩種 template 分流**（`training-notes-template.md` / `notes-template.md`，附判斷表）
- **拆 issue 前強制 user 校正**（流程設計的強制 checkpoint）

## Install

```bash
# Claude Code plugin marketplace（待提交）
/plugin install claude-meeting-pipeline
```

手動安裝（直到上 marketplace）：

```bash
git clone https://github.com/EricCaiCai/claude-meeting-pipeline.git ~/.claude/plugins/meeting-pipeline
```

## Setup（一次性）

```bash
# 1. 安裝 mlx-whisper（首次跑會 download ~3GB large-v3 模型）
uv tool install mlx-whisper

# 2. 安裝 ffmpeg
brew install ffmpeg

# 3. 把 transcribe-zh 加到 PATH（任選一種）
cp bin/transcribe-zh ~/.local/bin/
# 或
ln -s "$PWD/bin/transcribe-zh" ~/.local/bin/transcribe-zh

# 4. 驗證
transcribe-zh --dry-run /path/to/any.m4a
```

## Usage

### 三段式完整流程

```bash
# 在你的 repo 內，丟一個會議錄音給 Claude
你: 整理 meetings/2026-05-20-sync.m4a 這場會議

Claude 會：
  1. 問你會議 slug（推薦 meetings/<date>-<slug>/）
  2. 背景跑 transcribe-zh
  3. 驗證 hallucination
  4. 讀逐字稿產出 meeting-notes.md（先 preview 給你看）
  5. 等你校正人名 / owner
  6. （若有 docs/agents/issue-tracker.md）拆 1-3 個 issue
```

### 從中段開始

```bash
# 已有逐字稿，只想重產紀錄
你: 用這份 .txt 重產一份 meeting notes，這是分享場

# 已有紀錄，只想拆 issue
你: 從 meeting-notes.md 拆 issue
```

## Templates

| Template | 適用 |
|---|---|
| [`skills/meeting-pipeline/references/notes-template.md`](./skills/meeting-pipeline/references/notes-template.md) | 多人對等討論、達成決議、分配任務（sync / planning / retro） |
| [`skills/meeting-pipeline/references/training-notes-template.md`](./skills/meeting-pipeline/references/training-notes-template.md) | 講者單向傳播 + Demo + Q&A（教學 / 分享 / training） |

判斷依據：講者花大半時間講課/投影片/寫白板 → training；多人對等討論決議 → notes。混合場用 training。

## Glossary

[`skills/meeting-pipeline/references/glossary.md`](./skills/meeting-pipeline/references/glossary.md) 是空殼。

每個人接觸的領域不同（電商、醫療、金融、AI/開發工具...），whisper 中文偏移的詞彙也不同。本 plugin 只提供分類架構，**請依自己場域擴充**——3-5 場會議後就會收斂出貼合自己領域的對照表。

> 原作者在 Anthropic / Claude Code / harness engineering 領域累積的完整 glossary 已從本 plugin 移除（個人色彩過強）；若想參考可看 [原始 dotfiles repo](https://github.com/ericcai0814/dotfiles)。

## Issue Tracker（第 3 段）

第 3 段「拆 issues」依賴 `docs/agents/issue-tracker.md`（mattpocock skills 慣例）。
若 repo 沒有這個檔案，第 3 段會自動跳過——只到「會議紀錄」就停。

要啟用第 3 段，在你的 repo 新增 `docs/agents/issue-tracker.md` 描述 issue 格式（檔名 pattern、frontmatter schema、放在哪）。

## Requirements

| 項目 | 必要性 | 安裝 |
|---|---|---|
| macOS / Apple Silicon | 必要 | — |
| mlx-whisper | 必要 | `uv tool install mlx-whisper` |
| ffmpeg | 必要 | `brew install ffmpeg` |
| Claude Code CLI | 必要 | https://claude.com/claude-code |
| mattpocock skills setup | 可選（第 3 段） | https://github.com/mattpocock/skills |

## Non-macOS users?

mlx-whisper 用 Apple MLX framework，**不支援 Linux/Windows**。短期內：
- 自己改 `bin/transcribe-zh`，替換成 `faster-whisper` 或 `openai-whisper`（CPU/GPU）
- 保留 anti-hallucination flags 的精神

長期：v0.2 可能 abstract 出 transcription backend，pluggable 不同 whisper 實作。PR welcome。

## License

MIT
