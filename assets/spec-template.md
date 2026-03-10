# {PROGRAM_ID} ({SPOOL_ID}) — {PROGRAM_NAME}

> **程式類型**：{PROGRAM_TYPE}
> **開發日期**：{DEV_DATE}
> **原始碼**：{SOURCE_INFO}

---

## 一. 程式邏輯

{LOGIC_SECTION}

---

## 二. 副程式表格

| 程式代號 | 功能說明 | 呼叫段落 | 傳入參數 | 取回結果 |
|---------|---------|---------|---------|---------|
{CALL_TABLE}

---

## 三. Table 定義

{TABLE_SECTION}

---

{IF_INTERACTIVE}
## 四. 畫面規格

### 畫面排版

```
{SCREEN_LAYOUT}
```

### Record Format 欄位

{SCREEN_FIELDS}

### Function Key 對照表

| 按鍵 | 指標 | 功能說明 |
|------|------|---------|
{FK_TABLE}

### Indicator 對照表

| 指標 | 用途 | 控制欄位/動作 |
|------|------|-------------|
{INDICATOR_TABLE}

{END_IF_INTERACTIVE}

---

## 五. 參數介面

### LINKAGE SECTION

{LINKAGE_TABLE}

### LDA（Local Data Area）

{LDA_TABLE}
