# POR041C 程式規格說明文件

## 一、基本資訊

- **程式ID**：POR041C
- **程式名稱**：採購廠商交易明細報表主控程式
- **系統歸屬**：西祺企業管理資訊系統 - 採購管理模組
- **開發者**：A1139 JANE
- **建立日期**：1981.01.20
- **最後更新**：1998.11.10 (Y2K修改)

## 二、程式概述

POR041C是西祺企業管理資訊系統中採購管理模組的報表主控程式，負責協調採購廠商交易明細報表的產生流程。該程式透過使用者介面收集報表參數，然後呼叫相應的子程式執行報表產生任務。程式支援批次處理模式和互動式處理模式，可根據系統環境自動選擇適當的執行方式。

## 三、系統架構

POR041C程式在系統中的位置處於採購管理模組的報表處理環節，是連接使用者介面和實際報表處理的中間控制層。程式架構如下：

1. **主控程式**：POR041C - 負責流程控制和子程式呼叫
2. **介面程式**：POR041 - 負責使用者互動和參數收集
3. **處理程式**：POR041C1 - 負責實際的報表處理邏輯
4. **列印程式**：POR0411 - 負責報表格式和輸出

## 四、程式流程

1. **初始化階段**：
   - 從LDA(本地資料區)獲取使用者資訊和系統日期
   - 設定初始參數和環境變數

2. **使用者互動階段**：
   - 呼叫POR041顯示使用者介面
   - 收集使用者輸入的報表參數（包括廠商類別、廠商代碼範圍、日期等）
   - 處理使用者功能鍵（F3退出、F4說明）

3. **處理控制階段**：
   - 根據系統環境變數($EVR)判斷執行模式
   - 互動式模式：直接呼叫POR041C1程式
   - 批次處理模式：提交後台作業(SBMJOB)執行POR041C1

4. **循環控制**：
   - 程式執行完成後返回使用者互動階段，等待下一次操作

## 五、介面設計

POR041D顯示檔案是程式的主要使用者介面，包含以下元素：

1. **標題區**：
   - 程式標識和系統名稱
   - 當前使用者和系統日期/時間

2. **參數輸入區**：
   - 廠商類別選擇（F-廠商、L-內部）
   - 廠商代碼範圍（起始代碼、結束代碼）
   - 日期範圍
   - 報表明細選項

3. **功能鍵**：
   - F3：退出程式
   - F4：說明資訊

## 六、資料處理

POR041C1子程式負責實際的資料處理，主要流程包括：

1. **資料準備**：
   - 從LDA獲取使用者選擇的參數
   - 設定檔案覆蓋和列印參數

2. **資料查詢**：
   - 使用OPNQRYF指令根據條件查詢廠商交易資料
   - 根據廠商類別、代碼範圍和日期範圍篩選資料

3. **報表產生**：
   - 呼叫POR0411程式產生報表
   - 設定列印參數和輸出格式

## 七、檔案使用

程式使用的主要檔案包括：

1. **顯示檔案**：
   - POR041D - 使用者介面顯示檔案

2. **資料檔案**：
   - POPAPF - 廠商交易檔案
   - MTMBLF - 代碼檔案
   - MTMAPF - 輔助代碼檔案
   - PA#APF - 廠商基本資料檔案

3. **列印檔案**：
   - POR041P - 廠商交易明細報表格式

## 八、安全與權限

1. **使用者驗證**：
   - 程式透過LDA獲取使用者資訊
   - 使用者權限控制透過系統模組管理

2. **資料安全**：
   - 報表僅顯示使用者有權限存取的資料
   - 檔案共享設定確保資料一致性

## 九、效能考慮

1. **批次處理最佳化**：
   - 透過環境變數判斷是否使用批次處理模式
   - 大量資料處理時自動切換到後台作業處理

2. **查詢最佳化**：
   - 使用OPNQRYF指令提高查詢效率
   - 適當設定鍵值欄位提高排序效能

## 十、維護建議

1. **程式碼最佳化**：
   - 統一錯誤處理機制
   - 增強參數驗證邏輯

2. **功能增強**：
   - 增加更靈活的查詢條件
   - 提供報表預覽功能
   - 支援電子報表輸出

3. **相容性改進**：
   - 確保與新版系統的相容性
   - 最佳化Y2K相關程式碼

## 十一、整合點

1. **上游系統**：
   - 採購管理模組主選單(POM001)
   - 廠商基本資料維護程式(POA001)

2. **下游系統**：
   - 採購分析報表
   - 決策支援系統

## 十二、報表格式

POR041P列印檔案定義了報表的格式，主要包括：

1. **報表標題**：
   - 系統名稱和報表標題
   - 日期、時間和頁碼

2. **參數資訊**：
   - 廠商類別
   - 廠商代碼範圍
   - 報表內容類型

3. **明細資訊**：
   - 廠商代碼和名稱
   - 交易日期和類型
   - 交易金額和餘額
   - 其他相關資訊

## 十三、總結

POR041C程式是西祺企業管理資訊系統採購管理模組中的重要報表工具，負責產生廠商交易明細報表。程式設計合理，支援批次處理和互動式處理，能夠滿足企業日常採購分析的需求。透過與其他採購相關程式的協作，提供了完整的廠商交易資訊，協助企業進行採購決策和管理。