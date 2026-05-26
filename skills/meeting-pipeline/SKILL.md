---
name: meeting-pipeline
description: 從中文會議音訊（m4a/mp3/wav）產生結構化會議紀錄與 issues 的整合工作流。當用戶說「整理會議錄音」「轉逐字稿」「會議紀錄」「meeting notes」「整理這場會議」「拆 action items」，或丟一個音訊檔說要處理時觸發。涵蓋三段：(1) 呼叫 transcribe-zh 轉錄、(2) 依 references/notes-template.md 或 references/training-notes-template.md 產出會議紀錄、(3) 依 docs/agents/issue-tracker.md（若存在）拆 issues。
---

# Meeting Pipeline

整合「中文會議音訊 → 逐字稿 → 結構化紀錄 → 個人 issues」的工作流。

> 若 repo 沒有 mattpocock skills setup（無 `docs/agents/issue-tracker.md`），第三段「拆 issues」會跳過。

---

## 前置需求

| 需求 | 安裝 |
|------|------|
| macOS / Apple Silicon | — |
| `mlx-whisper` | `uv tool install mlx-whisper` |
| `transcribe-zh` | plugin 已附在 `${CLAUDE_PLUGIN_ROOT}/bin/transcribe-zh`，建議 `cp` 或 symlink 到 PATH（`~/.local/bin/` 或 `~/bin/`） |
| `ffmpeg`（mlx-whisper 依賴） | `brew install ffmpeg` |

驗證：

```bash
transcribe-zh --dry-run /path/to/sample.m4a
```

---

## 使用情境

- 用戶說「整理這場會議錄音」「轉成逐字稿」「會議紀錄」並提供音訊檔路徑
- 用戶丟一個 `.m4a` / `.mp3` / `.wav` 並要你處理
- 已有逐字稿，想重產會議紀錄（從第 2 段開始）
- 已有會議紀錄，想拆 issues（從第 3 段開始）

---

## 三段式流程

### 第 1 段：轉錄（deterministic，呼叫 shell script）

```bash
transcribe-zh <audio-file> [output-dir]
```

**預設輸出位置**：與音訊同目錄。建議結構：

```
meetings/<YYYY-MM-DD-slug>/
├── <audio-file>.m4a              # 原音訊（建議走 git-lfs）
├── <audio-file>.txt              # 純逐字稿
├── <audio-file>.srt              # 含時間戳
├── <audio-file>.vtt              # 網頁字幕
├── <audio-file>.tsv              # 表格
└── <audio-file>.json             # 完整 metadata
```

**驗證轉錄品質**（不可省）：

```bash
sort <output>.txt | uniq -c | sort -rn | head -5
```

如果最高重複行 > 50 次（連續同字 hallucination），代表 anti-hallucination flags 沒擋住，回頭手動加更嚴格參數重跑。常見原因：音訊有長段 BGM、回音、麥克風雜訊。

### 第 2 段：寫會議紀錄（需要 LLM 判斷）

**先選 template**：

| 場次性質 | Template |
|---------|----------|
| 講者單向傳播 + Demo + Q&A（教學 / 分享 / training） | [`references/training-notes-template.md`](./references/training-notes-template.md) |
| 多人對等討論、達成決議、分配任務（sync / planning / retro） | [`references/notes-template.md`](./references/notes-template.md) |
| 混合場（部分教學部分決議） | **training template**，把決議併入「關鍵原則」段 |

依選定 template 產出 `meeting-notes.md`，放在逐字稿同目錄。

**前置（不可省）：先讀 glossary.md 與 CONTEXT.md，逐名對照**：
- **跨專案偏移**：MUST 用 `Read` 讀本 skill 的 [`references/glossary.md`](./references/glossary.md)（Claude Code / 開發生態通用偏移；依自己場域持續擴充）。
- **本專案專屬名詞**：若 repo 根目錄有 `CONTEXT.md`，**也 MUST `Read` 全文**（不是只 `test -f` 確認存在）——人名、產品、團隊內慣用語。
- **判讀規則**：逐字稿每個專有名詞先比對上述兩份，**命中即用標準寫法、不要標 (?)**；只有兩份都查不到的新名才標 `_(?)_` 待人工校正。校正完一場後，把新確認的偏移累積回對應檔（跨專案→glossary.md，專屬→CONTEXT.md）。
- 本場紀錄末尾若列「對照表」，只列本場出現過的子集，不要照抄整份 glossary。

**讀取策略**：
- 逐字稿可能很長（一場 1 小時的會議常見 3000-5000 行）。**先讀 200 行樣本**看品質與內容類型，再決定要不要分批讀完整。
- 若超過 5000 行，考慮先抽樣判斷是否有 hallucination 殘留。

**輸出規則精要**（詳見 references）：
- TL;DR 5-7 條編號
- 與會者表格（角色 / 姓名 / 主要關注）
- 決策清單依領域分節，用 `- [x]`
- **Action Items 依「人」分組**（不是依「領域」分組）
- 詳細議題用 H3
- 風險顧慮獨立區段
- 連結回原始 `.srt` 並提示時間戳定位

**保留不確定性**：
- 英文人名/技術名詞偏移處用 `(?)` 標註
- 不要強裝確定（Whisper 對英文人名/技術名詞中文化偏移率高）
- 範例：`Nebula → Naple/Nebel _(Nebula?)_`、`Cloud Design _(?)_`

### 第 3 段：拆 issues（可選；需要 issue tracker setup）

**前置檢查**：

```bash
test -f docs/agents/issue-tracker.md && echo "OK" || echo "skip"
```

如果不存在，跳過第 3 段並告訴用戶：「這個 repo 沒設定 issue tracker（缺 `docs/agents/issue-tracker.md`），跳過拆 issues 階段。」

**若存在**：依 `docs/agents/issue-tracker.md` 描述的格式拆。常見模式（local markdown）：

```
.scratch/<feature-slug>/issues/<NN>-<slug>.md
```

**拆哪些 / 不拆哪些**（重要判斷）：

| 拆 | 不拆 |
|----|------|
| 跨人協作、自己是 owner 的 | 別人 owner 的（不在這個 repo 範圍）|
| 阻塞自己的 blocker（status: needs-info）| 5 分鐘內可完成的小事 |
| 規格清楚到可丟 agent | 純資訊性（「閱讀某文件」）|
| 多步驟、會有 sub-tasks | 已完成項 |

每個 issue 檔頂部 frontmatter：

```markdown
---
Status: ready-for-human | needs-info | ready-for-agent | wontfix
Owner: <name>
Waiting-on: <name>      # 若 status 是 needs-info
Source: meetings/<date>-<slug>/meeting-notes.md
Created: YYYY-MM-DD
---
```

底部加 `## Comments` 段落，記錄 issue 拆出時的評論。

---

## 整合執行流程

當用戶說「整理 xxx.m4a 這場會議」時：

1. **先確認** issue tracker 是否設定 → 決定要不要跑第 3 段
2. **決定輸出位置** → 通常是 `meetings/<YYYY-MM-DD-slug>/`，問用戶會議的 slug 名稱
3. **第 1 段：背景跑 `transcribe-zh`**（用 `run_in_background: true`，2-10 分鐘）
4. **等通知完成後**，驗證 hallucination
5. **第 2 段：讀逐字稿並產出會議紀錄**，先給 user preview 並提示「英文人名可能誤判，建議校正」
6. **等用戶校正完成**（人工 owner 認定不能省略，否則 issue 拆錯人）
7. **第 3 段：依用戶確認後的 owner，挑值得追的 1-3 個 issue 拆出**
8. **commit 分階段**：轉錄輸出一個 commit、會議紀錄一個、issues 一個

---

## 反指標（什麼時候不要用）

- **非中文音訊** → 改用 `mlx_whisper --language en` 或包別的 script
- **非會議內容**（podcast、訪談、課程） → 結構需求不同，不該套同一份 template
- **使用者只要「轉錄」不要「整理」** → 直接呼叫 `transcribe-zh`，不必喚起本 skill

---

## 為什麼這樣設計

- **三段式分層**：轉錄是純 shell（不需要 LLM 在 loop 裡），分離出來給其他工具/cron 也能用
- **第 3 段可選**：不是每個 repo 都有 issue tracker，硬綁定會在沒設定的 repo 失敗
- **不自動跑完三段**：第 2 段需要人校正才能進第 3 段——若 owner 認錯，issue 會拆給錯誤的人
