# Excel 進階功能：動態陣列、LAMBDA 與更多

## 基本資訊
- **來源**: 布布 research（Microsoft Excel 官方部落格、微軟開發者文件）
- **年份**: 2024-2026

## 核心內容

### 1. 動態陣列 (Dynamic Arrays) - 已經成熟
動態陣列是 Excel 的重大革新，讓公式可以傳回多個結果到相鄰儲存格。

**主要函數**:
- **FILTER**: 根據條件篩選資料
  ```excel
  =FILTER(A2:C20, B2:B20>100, "無資料")
  ```
- **SORT / SORTBY**: 排序資料
  ```excel
  =SORT(A2:C20, 3, -1)  // 按第3欄降冪排序
  ```
- **UNIQUE**: 提取唯一值
  ```excel
  =UNIQUE(A2:A100)
  ```
- **XLOOKUP**: 取代 VLOOKUP/HLOOKUP 的新函數
  ```excel
  =XLOOKUP(查詢值, 查詢範圍, 傳回範圍, 未找到時顯示)
  ```
- **XMATCH**: 新一代 MATCH 函數
- **SEQUENCE**: 生成連續數字序列
  ```excel
  =SEQUENCE(10, 1, 1, 1)  // 產生 1x10 的 1-10 序列
  ```
- **RANDARRAY**: 生成隨機數字陣列

### 2. LAMBDA 函數 - 自訂函數的強大工具
LAMBDA 讓您可以在 Excel 中建立可重複使用的自訂函數，無需 VBA 或巨集。

**基本語法**:
```excel
=LAMBDA(參數, 運算式)
```

**實際範例 - 計算稅金**:
```excel
=LAMBDA(金額, 稅率, IF(金額>500000, 金額*0.2, 金額*稅率))
```

**LET 函數搭配 LAMBDA**:
```excel
=LAMBDA(x, y, LET(sx, x*2, sy, y*3, sx+sy))(A1, B1)
```

**名稱管理員建立 LAMBDA**:
1. 公式 → 名稱管理員 → 新增
2. 名稱: `我的函數`
3. 參照到: `=LAMBDA(x, x^2 + 2*x + 1)`

### 3. 2024-2025 新函數
- **TEXTSPLIT**: 文字分割（對應 CONCATENATE 的相反）
- **VSTACK / HSTACK**: 垂直/水平堆疊陣列
- **TOCOL / TOROW**: 將範圍轉換為單一列或欄
- **WRAPCOLS / WRAPROWS**: 將陣列包裝成多欄/多列
- **CHOOSECOLS / CHOOSEROWS**: 選擇特定欄/列
- **EXPAND**: 擴展陣列到指定尺寸
- **LAMBDA 增強**: 遞迴 LAMBDA 可用於複雜計算

### 4. Power Automate Excel 整合 (2024)
- **在 Excel 中直接建立自動化流程**: 按鈕式觸發
- **即時資料更新**: 自動化填入、格式化、資料驗證
- **PDF 轉 Excel 自動化**: 智慧型表格辨識
- **與 Microsoft Lists 整合**: SharePoint List 自動同步 Excel

### 5. Excel 中的 AI 功能 (2025-2026)
- **Ideas in Excel**: 自然語言分析資料、產生圖表建議
- **Copilot in Excel**: "分析此資料並找出趨勢"、"建立預測模型"
- **內建 Python 支援**: 直接在 Excel 中執行 Python 程式碼分析
- **自然語言公式建立**: 描述想要的計算，Excel 自動建議公式

### 6. 效能改進
- **加快的計算引擎**: 動態陣列計算效能提升
- **更大範圍支援**: 現在支援整個工作表的陣列公式
- **相容性改善**: 舊版 Excel 檔案的動態陣列向後相容

## 實際應用

### 資料分析範例
1. **快速篩選與彙總**:
   ```excel
   =SORT(FILTER(A:C, (B:B>100)*(C:C="已完成")), 1, -1)
   ```

2. **動態下拉選單**:
   ```excel
   =UNIQUE(資料庫!A:A)
   ```
   設定資料驗證 → 允許 → 清單 → 來源使用此公式

3. **自訂 LAMBDA 函數 - 計算 BBB**:
   - 在名稱管理員建立: `=LAMBDA(a, b, (a+b)/2 + SQRT(a*b))`
   - 使用: `=我的BBB(10, 20)`

### 報表自動化
- VSTACK + 查詢函數 = 合併多個工作表為單一視圖
- LET + LAMBDA = 複雜公式效能最佳化

## 標籤
#Excel #動態陣列 #LAMBDA #FILTER函數 #XLOOKUP #LET函數 #PowerAutomate #ExcelAI