# PAM000C 程式規格書

## 1. 基本資訊

- **程式名稱**：PAM000C
- **程式類型**：CL程式（控制語言程式）
- **程式庫**：REPLIB
- **功能描述**：採購系統主選單控制程式
- **相關檔案**：PAM000D（顯示檔案）
- **系統名稱**：西祺企業管理資訊系統
- **子系統**：採購系統
- **作者**：ROGER CHIEN（從PAM000D推斷）
- **建立日期**：80/11/15（從PAM000D推斷）

## 2. 程式概述

PAM000C是西祺企業管理資訊系統中採購系統的主選單控制程式。此程式負責顯示採購系統的主選單介面，處理使用者的選擇，檢查使用者權限，並呼叫相應的採購功能模組程式。它是採購系統的中央控制點，提供了一個統一的入口點來訪問所有採購相關功能，包括採購訂單的輸入、查詢、修改、刪除等操作。

## 3. 程式流程

```
開始
  |
  ↓
宣告變數和檔案
  |
  ↓
覆蓋訊息檔
  |
  ↓
從LDA獲取使用者資訊和日期
  |
  ↓
設定使用者ID變數
  |
  ↓
START: 顯示採購系統主選單畫面
  |
  ↓
接收使用者選擇
  |
  ↓
根據選擇設定對應的程式名稱
  |
  ↓
檢查使用者權限 (CHKAUT)
  |
  ↓
呼叫選擇的程式或返回主選單
  |
  ↓
結束
```

## 4. 程式碼分析

### 4.1 變數宣告

```
DCL VAR(&WDATE) TYPE(*CHAR) LEN(6)
DCL VAR(&$USER) TYPE(*CHAR) LEN(10)
DCL VAR(&S0101I) TYPE(*CHAR) LEN(10)
DCL VAR(&S0102I) TYPE(*CHAR) LEN(10)
DCL VAR(&S0101O) TYPE(*CHAR) LEN(1)
DCLF FILE(*LIBL/PAM000D) RCDFMT(DSPC1)
```
- 宣告日期變數&WDATE，長度為6字元，用於儲存系統日期
- 宣告使用者ID變數&$USER，長度為10字元，用於儲存當前使用者ID
- 宣告程式參數變數&S0101I、&S0102I和&S0101O，用於權限檢查和程式呼叫
- 宣告PAM000D顯示檔案，使用DSPC1記錄格式顯示主選單

### 4.2 系統初始化

```
OVRMSGF MSGF(QCPFMSG) TOMSGF(PTFLIB/PTMF)
RTVDTAARA DTAARA(*LDA (101 10)) RTNVAR(&$USER)
RTVDTAARA DTAARA(*LDA (119 6)) RTNVAR(&WDATE)
CHGVAR VAR(&$EGMDY) VALUE(&WDATE)
CHGVAR VAR(&S0101I) VALUE(&$USER)
```
- 覆蓋系統訊息檔QCPFMSG為自定義訊息檔PTFLIB/PTMF，提供本地化訊息
- 從本地資料區(LDA)位置101-110獲取使用者ID，儲存到&$USER
- 從本地資料區(LDA)位置119-124獲取系統日期，儲存到&WDATE
- 將日期值設定到顯示變數$EGMDY，用於在主選單上顯示
- 將使用者ID設定到參數變數S0101I，用於後續權限檢查

### 4.3 主選單顯示與處理

```
START:
SNDRCVF DEV(*FILE) RCDFMT(DSPC1)
CHGVAR VAR(&IN98) VALUE('0')
```
- 顯示採購系統主選單畫面並接收使用者輸入
- 使用DSPC1記錄格式，這是在PAM000D中定義的主選單格式
- 重設錯誤指示器IN98為'0'，清除之前可能的錯誤狀態

### 4.4 選項處理

```
IF COND(&DOPID *EQ '01') THEN(DO)
   CHGVAR VAR(&S0102I) VALUE('PAA001C')
ENDDO
...
IF COND(&DOPID *EQ '90') THEN(RETURN)
IF COND(&DOPID *EQ '  ') THEN(GOTO START)
```
- 根據使用者選擇的選項(DOPID)設定對應的程式名稱到&S0102I
- 系統包含50個採購相關功能，從PAA001C到PAA050C
- 如果選擇'90'，則返回上層選單（RETURN命令）
- 如果未選擇任何選項（空白），則重新顯示主選單（GOTO START）

### 4.5 權限檢查與程式呼叫

```
CHKAUT: CALL PGM(S#S01) PARM(&S0101I &S0102I &S0101O)
IF COND(&S0101O = 'Y') THEN(CALL PGM(&S0102I))
```
- 呼叫S#S01程式檢查使用者權限，傳入使用者ID(&S0101I)和目標程式(&S0102I)
- S#S01返回權限檢查結果到&S0101O，'Y'表示有權限
- 如果有權限(&S0101O = 'Y')，則呼叫選擇的程式(&S0102I)
- 如果無權限，程式會設定錯誤指示器並返回主選單（程式碼未完全顯示）

## 5. 功能模組對應表

根據PAM000D顯示檔案的內容，以下是主要功能模組的對應表：

| 選項 | 程式名稱 | 功能描述 |
|------|----------|----------|
| 01 | PAA001C | 採購訂單輸入 |
| 02 | PAA002C | 採購訂單查詢 |
| 03 | PAA003C | 採購訂單處理 |
| 04 | PAA004C | 採購訂單處理 |
| 05 | PAA005C | 採購訂單處理 |
| ... | ... | ... |
| 21 | PAA021C | 採購訂單修改 |
| 22 | PAA022C | 採購訂單刪除 |
| ... | ... | ... |
| 50 | PAA050C | 採購訂單處理 |
| 90 | (RETURN) | 返回上層選單 |

## 6. 相關檔案分析

### 6.1 PAM000D（採購系統主選單顯示檔案）

- **功能**：定義採購系統主選單的顯示格式
- **主要記錄格式**：DSPC1
- **主要欄位**：
  - $EGMDY：顯示日期，格式為YY/MM/DD
  - $USER：顯示使用者ID
  - DOPID：使用者選擇的選項，長度為2字元
  - 錯誤訊息欄位（由指示器98控制）

- **畫面布局**：
  - 顯示程式ID、系統標題、日期、時間和使用者資訊
  - 列出所有可用的採購功能選項，包括：
    - 採購訂單輸入、查詢、修改、刪除等基本功能
    - 各種採購訂單處理功能
  - 提供功能選擇輸入欄位
  - 底部顯示錯誤訊息區域

### 6.2 其他相關程式

- **S#S01**：使用者權限檢查程式，檢查使用者是否有權限執行特定功能
- **PAA001C-PAA050C**：各種採購功能模組程式，處理具體的採購業務邏輯
- **REM000C**：系統主選單程式，呼叫PAM000C進入採購系統

## 7. 程式特點

1. **模組化設計**：將不同採購功能分離為獨立的模組程式（PAA001C-PAA050C）
2. **權限控制**：通過S#S01程式檢查使用者權限，確保安全性
3. **使用者友好**：提供清晰的選單介面，方便使用者導航
4. **標準化結構**：遵循系統中其他選單程式的標準結構和命名規範
5. **完整功能**：提供豐富的採購相關功能，滿足企業採購管理需求

## 8. 系統架構關係

PAM000C在整個系統架構中的位置：

```
REM000C (系統主選單)
   |
   ↓
PAM000C (採購系統主選單) ←→ PAM000D (顯示檔案)
   |
   ↓
採購功能模組 (PAA001C-PAA050C)
```

## 9. 資料流分析

1. **輸入資料**：
   - 使用者選擇的選項(DOPID)
   - 本地資料區(LDA)中的使用者ID和日期

2. **處理邏輯**：
   - 根據選項確定要呼叫的程式
   - 檢查使用者權限
   - 呼叫相應的採購功能模組程式

3. **輸出資料**：
   - 採購系統主選單顯示
   - 錯誤訊息（如果有）
   - 採購功能模組程式的呼叫

## 10. 注意事項

1. 程式是採購系統的核心控制程式，修改時需謹慎
2. 程式依賴多個外部程式和檔案，如PAM000D、S#S01等
3. 系統使用LDA（本地資料區）儲存使用者資訊和日期
4. 程式支援大量的採購功能選項（50個），需要確保所有對應的程式都存在

## 11. 建議與改進

1. 可考慮使用子程式結構或WHEN-CASE結構來簡化選項處理邏輯
2. 可增加更詳細的錯誤處理和日誌記錄
3. 可考慮將功能選項分類或分頁，提高使用者體驗
4. 可考慮使用參數檔案來管理功能模組對應，使系統更具彈性

---

此規格書基於PAM000C.txt和PAM000D.txt的分析。PAM000C作為西祺企業管理資訊系統中採購系統的主選單控制程式，提供了一個統一的入口點來訪問所有採購相關功能。它與系統的其他模組（如銷售、庫存等）共同構成了一個完整的企業資源規劃(ERP)系統。