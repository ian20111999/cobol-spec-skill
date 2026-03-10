# COBOL Spec Generator

一個 Claude Code Skill，能自動解析 AS/400 COBOL 程式並產出標準化中文規格書。

## 功能

- 支援所有程式類型：線上互動（INTERACTIVE）、批次（BATCH）、副程式（SUBPROGRAM）、報表（REPORT）
- 自動拆解 AS/400 Spool File（SEU SOURCE LISTING / COPY FILE 格式）
- 解析 DDS 原始碼（PF / LF / DSPF）
- 逐段翻譯 COBOL PROCEDURE DIVISION 為中文邏輯說明
- 產出完整規格書（Markdown + HTML）
- 內建驗證機制確保規格書完整性

## 安裝

### 前置需求

- [Claude Code CLI](https://docs.anthropic.com/en/docs/claude-code)
- Python 3.9+（無需額外套件）

### 安裝步驟

```bash
# 1. Clone 到 Claude Code skills 目錄
git clone git@github.com:<your-username>/cobol-spec-skill.git ~/.claude/skills/cobol-spec

# 2. 確認 skill 已被載入
claude  # 啟動 Claude Code，輸入 /cobol-spec 應可觸發
```

> **注意**：若你的 `~/.claude/skills/` 目錄尚不存在，先建立它：`mkdir -p ~/.claude/skills`

## 使用方式

### 快速開始

1. 將 AS/400 匯出的 Spool File（`.txt`）放到工作目錄下
2. 在 Claude Code 中輸入：

```
/cobol-spec
```

3. Skill 會互動式引導你完成所有步驟

### 指定檔案

```
/cobol-spec my_spool.txt              # 自動選擇最大的 COBOL 程式
/cobol-spec my_spool.txt MY_PROGRAM   # 指定程式名
```

### 完整工作流程

```
Step 1: Spool 拆解
  └─ 自動辨識 spool file 中的所有 DDS/COBOL/CL 元件
  └─ 列出找到的 COBOL 程式，請你選擇要分析哪一支

Step 2: 骨架解析
  └─ 提取程式結構（files, paragraphs, calls, linkage）
  └─ 報告摘要，檢查所需的 DDS 原始碼是否齊全
  └─ 若有缺少的 DDS .txt 檔案，會請你補充

Step 3: 平行分析
  ├─ 邏輯翻譯：逐段翻譯 PROCEDURE DIVISION（分批處理）
  ├─ 副程式分析：分析所有 CALL 目標的功能
  ├─ Table 定義：解析 DDS 產出欄位定義表格
  └─ 畫面解析：解析 DSPF 產出畫面規格（僅 INTERACTIVE）

Step 4: 組裝規格書
  └─ 合併所有分析結果為完整 Markdown + HTML

Step 5: 驗證
  └─ 交叉比對規格書與程式骨架，確保無遺漏
```

## 輸入檔案準備

### Spool File（必要）

從 AS/400 匯出的編譯清單，包含 COBOL 原始碼。常見匯出方式：

```
CPYSPLF FILE(QPJOBLOG) TOFILE(QGPL/QPRINT) SPLNBR(*LAST)
```

或用 Client Access 直接另存為 `.txt`。

### DDS 原始碼（建議提供）

每個程式用到的 Physical File / Logical File 的 DDS 原始碼，同樣匯出為 `.txt`：

```
WRKMBRPDM FILE(YOURLIB/QDDSSRC) MBR(FFDFALD0) → F16 列印 → 另存 .txt
```

> Skill 在分析時會自動檢查哪些 DDS 檔案缺少，並列出清單請你補充。
> 若確實無法提供某些檔案，Skill 會從 COBOL WORKING-STORAGE 推斷，並標註來源。

### Display File（INTERACTIVE 程式需要）

線上互動程式的 DSPF 原始碼：

```
WRKMBRPDM FILE(YOURLIB/QDDSSRC) MBR(MFD0062) → F16 列印 → 另存 .txt
```

## 輸出

```
output/{program_id}/
├── {spool}_inventory.json     # Spool 元件清單
├── {program}_skeleton.json    # 程式骨架
├── {program_id}_spec.md       # 規格書（Markdown）
└── {program_id}_spec.html     # 規格書（HTML，可直接開啟）
```

### 規格書章節

| 章節 | 內容 | 適用類型 |
|------|------|---------|
| 一. 程式邏輯 | 逐段中文邏輯說明 | 全部 |
| 二. 副程式表格 | CALL 目標功能說明 | 有 CALL 時 |
| 三. Table 定義 | 檔案欄位定義 | 有檔案操作時 |
| 四. 畫面規格 | 欄位 + FK + Indicator + ASCII 排版 | 僅 INTERACTIVE |
| 五. 參數介面 | LINKAGE + LDA | 全部 |

## Python 工具腳本

Skill 內建 5 個 Python 腳本，也可獨立使用：

```bash
# 拆解 spool file
python3 scripts/spool_splitter.py <spool.txt>

# 解析 COBOL 骨架
python3 scripts/cobol_skeleton.py <spool.txt> --program <PROGRAM_NAME>

# 解析 DDS（Physical/Logical File）
python3 scripts/dds_parser.py <dds_file.txt>

# 解析 DSPF（Display File）
python3 scripts/dds_parser.py <dspf_file.txt> --dspf

# 驗證規格書完整性
python3 scripts/spec_validator.py <spec.md> <skeleton.json>

# Markdown → HTML
python3 scripts/md2html.py <spec.md>
```

## 檔案結構

```
cobol-spec/
├── SKILL.md                    # 主 Skill 定義（Claude Code 讀取此檔）
├── README.md                   # 本說明文件
├── scripts/
│   ├── spool_splitter.py       # Spool File → inventory.json
│   ├── cobol_skeleton.py       # COBOL → skeleton.json
│   ├── dds_parser.py           # DDS → field list JSON
│   ├── spec_validator.py       # 驗證 spec 完整性
│   └── md2html.py              # Markdown → HTML
├── references/
│   ├── logic-translator.md     # 邏輯翻譯 prompt 模板
│   ├── screen-analyzer.md      # 畫面解析 prompt 模板
│   └── callsite-analyzer.md    # 副程式分析 prompt 模板
└── assets/
    ├── cobol-dictionary.json   # COBOL 術語中文對照表
    └── spec-template.md        # 規格書格式模板
```

## 自訂與擴充

### 術語對照表

編輯 `assets/cobol-dictionary.json` 可自訂 COBOL 動詞、File Status 碼、資料型態的中文翻譯。

### 規格書模板

編輯 `assets/spec-template.md` 可調整規格書的章節結構和格式。

### 翻譯 Prompt

`references/` 目錄下的三個 `.md` 檔定義了 AI 分析的 prompt 模板，可根據需求調整翻譯深度和用詞風格。

## 限制

- 需要 Python 3.9+，但不需額外安裝任何第三方套件
- 極大程式（>10,000 行）會自動增加批次數，處理時間較長
- DDS REF() 參照需要被參照的 PF 也在工作目錄下才能完整解析
- Big5 編碼的中文在 spool file 中可能顯示為亂碼，Skill 會根據欄位命名推斷含義

## License

Private — 僅限受邀成員存取。
