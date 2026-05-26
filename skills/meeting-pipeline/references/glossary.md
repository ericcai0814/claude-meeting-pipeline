# Glossary: mlx-whisper 中文逐字稿偏移詞彙修正表

> 本檔是空殼架構。請依自己接觸的領域擴充——例如電商、醫療、金融、AI/開發者
> 工具，各自會有完全不同的「whisper 中文偏移詞彙」。
> 完整範例見原作者個人版本（連結見 README）。

跨專案的 whisper 偏移修正表，給 meeting-pipeline skill 第 2 段「寫會議紀錄」時參考。
專案專屬名詞（人名、產品代號、團隊內部慣用語）請改放在該 repo 的 `CONTEXT.md`。

> 使用方式：寫紀錄時，先讀本檔對逐字稿做心理對照；末尾「Whisper 偏移詞彙對照表」可挑與本場相關的子集列出。

---

## Claude Code / Agent 生態（AI 工程 / 開發者會議常見）

> 跨專案通用：凡是討論 Claude Code、agent、開發工具鏈的中文會議都易出現這些偏移。

| 逐字稿（任一種寫法） | 實際 |
|----------------------|------|
| Cloud Code / Cloud（指工具時） | **Claude Code** |
| Honest / Honus / HARMES | **Harness** |
| HOOP / HOODs / HOOCH | **Hook** |
| Skell / Scale（指 Skill 時）/ Scull | **Skill** |
| Site Agent / SY Agent / Savage | **Subagent** |
| Prong / Pron / Chrome（指 Prompt 時） | **Prompt** |
| Sation / Bond（指 Session 時） | **Session** |
| Pretour Use / Post Tour Use | **Pre-tool-use / Post-tool-use** |
| NCP | **MCP** |
| Codecs / CodeC | **Codex** |
| Folk / Foulk | **Fork** |
| Nodes（指語言時） | **Node.js** |
| Nobu LN / Notebook LN | **NotebookLM** |
| Adrian / Adrian Skell | **agent skill**（非人名 / repo 名） |

## 領域 B（換成你自己的領域：電商 / 醫療 / 金融 / …）

| 逐字稿 | 實際 |
|--------|------|
| _(待填)_ | _(待填)_ |

## 人名 / Owner 偏移

| 逐字稿 | 實際 |
|--------|------|
| _(待填)_ | _(待填)_ |

## 路徑 / 副檔名

| 逐字稿 | 實際 |
|--------|------|
| `.nd` / ND | **`.md`** |
| EMV / EMB | **`.env`** |
| JSUM | **JSON** |

## 常見技術片語

| 逐字稿 | 實際 |
|--------|------|
| 漸減（式） | **漸進（式）** |

## YouTube / 套語 hallucination

mlx-whisper 對音訊**尾端靜音段**或**長段 BGM** 容易產生這類重複 hallucination，**務必刪除**：

- 中國 YouTube 訂閱套語（例：「请不吝点赞 订阅 转发 打赏支持...」）
- 連續 50+ 次的「好好好好…」「我现在要做什么…」— 中文長靜音段陷阱
- 重複的「謝謝大家」「拜拜」— 收尾若超過 3 次也是 hallucination

> 驗證方法：`sort <output>.txt | uniq -c | sort -rn | head -10`，若某行重複 > 30 次，基本確定為 hallucination。

---

## 使用注意

1. **不要盲目 replace**：本表是「**候選對照**」，有些詞在不同 context 意思不同
   - 例：`Cloud` 在 AI 領域常被誤轉成「Claude」，但若會議真的在講 cloud computing 就不要改
2. **末尾對照表 < 20 行**：寫紀錄末尾的「Whisper 偏移詞彙對照表」只列**本場實際出現過**的子集，不要照抄整份本檔
3. **人名 → CONTEXT.md**：本檔是跨專案，專案專屬人名請放該專案 `CONTEXT.md`

---

## 怎麼擴充本檔

每次寫完會議紀錄後，把「本場校正過的偏移」累積進本檔。3-5 場後就會收斂出一份貼合你領域的對照表。
