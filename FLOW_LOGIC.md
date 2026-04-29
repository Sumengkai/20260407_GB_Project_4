# 塊材申請作業 (GB_Project_2) — 流程邏輯文件

> 技術棧：Vue 3 + Vite | 版本：0.0.0 | 類型：SPA 前端應用

---

## 目錄

1. [專案結構](#1-專案結構)
2. [整體流程圖](#2-整體流程圖)
3. [分步驟流程說明](#3-分步驟流程說明)
4. [核心資料結構](#4-核心資料結構)
5. [元件責任表](#5-元件責任表)
6. [關鍵函式清單](#6-關鍵函式清單)
7. [商業規則](#7-商業規則)
8. [資料庫對應表（DB Schema）](#8-資料庫對應表db-schema)
9. [常數與設定參數](#9-常數與設定參數)
10. [系統日誌格式](#10-系統日誌格式)
11. [待整合事項](#11-待整合事項)

---

## 1. 專案結構

```
20260407_GB_Project_2/
├── index.html                  # 應用程式進入點，掛載 #app
├── package.json                # Vue 3.5.32 + Vite 6.0.0
├── vite.config.js              # Vite 建置設定
├── DB_Schema_D5申請單.md       # 資料庫 Schema 文件（7 張表）
├── src/
│   ├── main.js                 # 建立並掛載 Vue 應用
│   ├── App.vue                 # 主元件（所有狀態集中管理）
│   ├── style.css               # 全域樣式（微軟正黑體）
│   └── components/
│       ├── ConsumptionTab.vue      # 耗用 V1（機加工/鍍膜/純化）
│       ├── ConsumptionTabV2.vue    # 耗用 V2（機加工扁平模式）
│       ├── OutputTab.vue           # 產出 V1 容器（含自產/委外）
│       ├── OutputSection.vue       # 產出通用區塊（可重用）
│       ├── OutputTabV2.vue         # 產出 V2（含執行方式欄位）
│       ├── QualityTab.vue          # 品檢資訊輸入
│       └── HelloWorld.vue          # 未使用（模板殘留）
└── dist/                       # 建置輸出目錄
```

---

## 2. 整體流程圖

```
┌─────────────────────────────────────────────────────────────────┐
│                         使用者介面入口                            │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  查詢列 (Query Bar)  │  輸入詢價單或申請單號查詢
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Tab 0：詢價單     │  ← 發起起點
                    │  ─────────────────  │
                    │  [新增主檔]          │  建立 IQ 單號
                    │  [挑選庫存（鎖料）]  │  選庫存加入鎖料批號
                    │  [確認轉D5申請單]   │  產生 D5 單號並跳轉
                    └──────────┬──────────┘
                               │ 確認轉換
                    ┌──────────▼──────────┐
                    │  Tab 1：D5申請單    │  唯讀，資料連動詢價單
                    │  ─────────────────  │
                    │  所有欄位不可編輯   │
                    │  顯示 D5 單號       │
                    │  工單清單           │
                    │  產出批號總覽       │
                    └──────────┬──────────┘
                               │ 依詢價單勾選類型動態顯示後續 Tab
          ┌────────────────────┼────────────────────┐
          │                    │                    │
┌─────────▼────────┐ ┌─────────▼────────┐ ┌────────▼─────────┐
│   機加工  V1/V2  │ │      鍍膜        │ │      純化        │
│  Tab 2/3 (4/5)  │ │   Tab 6 / 7      │ │   Tab 8 / 9      │
│  耗用 → 產出     │ │   耗用 → 產出    │ │   耗用 → 產出    │
└─────────┬────────┘ └─────────┬────────┘ └────────┬─────────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               │ 有產出批號時
                    ┌──────────▼──────────┐
                    │  Tab 10：品檢資訊   │  依批號/產品輸入品檢
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  工單自動產生/刪除   │
                    │  系統日誌記錄       │
                    └─────────────────────┘
```

---

## 3. 分步驟流程說明

### Step 1 — 初始化

1. `main.js` 建立 Vue App，掛載至 `#app`
2. `App.vue` 初始化所有 reactive 狀態（含 mock 庫存、鎖定批號）
3. 預設交貨日 = 當月 + 1 個月
4. 預設起點為 Tab 0（詢價單）

---

### Step 2 — 查詢單號

1. 使用者輸入詢價單號（`IQ-YYYYMM-NNNN`）或申請單號（`D5-YYYYMM-NNNN`）
2. 點擊「查詢」→ 呼叫 `doQuery()`
3. 優先比對 `iq.id`，再比對 `d5.id`，顯示成功/失敗訊息並切換至對應 Tab

---

### Step 3 — 建立詢價單（Tab 0）

```
填寫欄位
  ├─ 交貨日（必填，預設下個月）
  ├─ 上傳附件（選填）
  ├─ 委託類型（可複選）
  │    ☐ 機加工  ☐ 鍍膜  ☐ 純化
  ├─ 爐號、費用、客戶、備註、客戶材料
  └─ 點擊 [新增主檔] → iqCreate()

系統自動產生
  └─ 詢價單號：IQ-{YYYYMM}-{NNNN}
```

---

### Step 4 — 鎖料（Tab 0）

```
點擊 [打開視窗挑選庫存（鎖料）]
    ↓
庫存 Modal 開啟（顯示全部倉庫，不限庫別）
    ↓
關鍵字搜尋 + 多選核取方塊
    ↓
[確認鎖料] → 選取批號加入 iq.lockedLots
    ↓
系統日誌：「鎖料 N 筆批號：...」

注意：
  - 重複 lotNo 不會重複加入
  - 可多次開啟 Modal 繼續追加鎖料批號
```

---

### Step 5 — 確認轉D5申請單（Tab 0 → Tab 1）

```
點擊 [確認轉D5申請單]
    ↓
驗證：需已建立詢價單且有填交期
    ↓
產生 D5 單號：D5-{YYYYMM}-{NNNN}
    ↓
系統日誌：「由詢價單 IQ-xxx 轉換，建立 D5 申請單：D5-xxx」
    ↓
自動切換至 Tab 1（D5申請單）
    ↓
按鈕變更為「已轉換：D5-xxx」（灰色，可點擊跳回 D5 tab）

注意：一張詢價單只能轉換一次，轉換後不可重複建立新的 D5 單號
```

---

### Step 6 — D5申請單操作（Tab 1）

```
工具列功能：
  ├─ [儲存]：呼叫 d5Save()，記錄操作日誌並顯示成功訊息
  └─ 若尚未轉換，工具列顯示「請先在詢價單按下確認轉D5申請單」提示

D5 專屬可編輯欄位（獨立於詢價單）：
  ├─ 受理與否（必填）：是 / 否（Radio 選擇）
  │    ✓ 選「是」→ 後續製程頁籤根據委託類型顯示
  │    ✗ 選「否」→ Tab 2~10 全部隱藏
  ├─ 備註：獨立 textarea（v-model d5.remark，不連動詢價單備註）
  └─ 附件：獨立 file input（@change d5FileChange，不連動詢價單附件）

唯讀連動詢價單（iq.*）的欄位：
  ├─ 交期、委託類型、爐號、費用、客戶、客供料
  ├─ 鎖料批號（同詢價單）
  ├─ 產出批號總覽（由各製程產出批號彙整）
  ├─ 工單清單（由耗用/產出資料自動驅動）
  └─ 系統備註（與詢價單共用同一份 iq.sysLog）
```

---

### Step 7 — 動態 Tab 顯示邏輯

**前提條件：Tab 2~10 僅在 D5 申請單「按下儲存」（d5.saved === true）且「受理與否」為「是」（d5.accepted === 'Y'）時才可能顯示。**

```
設定「受理與否」→ 按下「儲存」→ saved = true → 後續頁籤依委託類型顯示
                              ↓
            若再次修改「受理與否」→ saved 自動重設為 false
                              → 頁籤立即隱藏，需重新儲存
```

| 顯示條件 | 顯示 Tab |
|----------|----------|
| !d5.saved 或 d5.accepted !== 'Y' | 僅顯示 Tab 0, 1 |
| d5.saved + d5.accepted === 'Y' + typeMachining + V1 | Tab 2（耗用）、Tab 3（產出） |
| d5.saved + d5.accepted === 'Y' + typeMachining + V2 | Tab 4（耗用 V2）、Tab 5（產出 V2） |
| d5.saved + d5.accepted === 'Y' + typeCoating | Tab 6（耗用）、Tab 7（產出） |
| d5.saved + d5.accepted === 'Y' + typePurification | Tab 8（耗用）、Tab 9（產出） |
| d5.saved + d5.accepted === 'Y' + 任一製程已選 | Tab 10（品檢資訊）|

> Tab 0（詢價單）與 Tab 1（D5申請單）永遠顯示。  
> 計算屬性 `visibleTabIndices` 驅動顯示邏輯；若目前 Tab 被隱藏則自動重設為 Tab 0。  
> 受理與否修改後，`watch(() => d5.accepted)` 會自動將 `d5.saved` 重設為 `false`，需重新儲存。

---

### Step 8 — 耗用流程（Tab 2, 4, 6, 8）

#### V1 模式（ConsumptionTab.vue）

```
[打開視窗挑選庫存]
    ↓
庫存 Modal 開啟（依自產/委外過濾倉庫）
    ├─ 自產（Self）：BB 倉庫
    └─ 委外（Outsource）：排除 BBPN
    ↓
[確認耗用] → 新增至耗用表
    ↓
[取消耗用] → 刪除選取行
    ↓
系統日誌：「輸入【製程名(耗用)】」
```

#### V2 模式（ConsumptionTabV2.vue）

```
扁平單表 + 執行方式欄位（01=自產/02=委外）
其餘流程與 V1 相同
```

---

### Step 9 — 產出流程（Tab 3, 5, 7, 9）

```
底部輸入列 → 填寫欄位 → [新增]（呼叫 handleOutputAdd）
    ↓
現有行可 [修改] / [刪除]
    ↓
系統日誌：「輸入【製程名(產出)】」
```

---

### Step 10 — 工單自動產生 / 刪除（Watch 驅動）

```
有耗用或產出資料 → ensureWorkOrder(key, proc, execType)
    ↓ 第一筆資料加入時
產生工單號：WO-{YYYYMM}-{NNNN}
日誌：「生成【製程(方式)】工單：WO-YYYYMM-NNNN」

所有耗用/產出資料刪除 → removeWorkOrder(key)
    ↓
移除工單，日誌同步移除

唯一鍵 = 製程 × 執行方式（每組合一張工單）
```

---

### Step 11 — 品檢資訊（Tab 10）

```
依批號分組顯示產出品項
    ↓
依產品代碼顯示不同品檢參數（6-15 個欄位）
    ↓
使用者填入實測值 → handleQualityChange({lotNo, paramKey, value})
    ↓
系統日誌：「輸入【品檢資訊】批號 {lotNo}」
```

---

## 4. 核心資料結構

### 主狀態（App.vue reactive）

```javascript
// 詢價單資料（主要資料來源，所有欄位均在此）
iq: {
  id,              // IQ-YYYYMM-NNNN（建立後產生）
  deliveryDate,    // YYYY-MM-DD（必填）
  attachmentName,  // 上傳附件名稱
  typeMachining, typeCoating, typePurification, // boolean（驅動後續 Tab 顯示）
  furnaceNo, cost, customer, remark, clientMaterial,
  lockedLots: [],  // [{ lotNo, itemNo, itemName, qty, unit }]（鎖料結果）
  sysLog: []       // [{ time, msg, level, key }]（詢價單與D5共用）
}

// D5申請單（單號 + D5專屬欄位，備註/附件與詢價單分開管理）
d5: {
  id,             // D5-YYYYMM-NNNN（由詢價單轉換時產生）
  accepted,       // '' | 'Y' | 'N'（受理與否；修改後自動重設 saved = false）
  saved,          // boolean（按下儲存後為 true；控制後續 Tab 是否顯示的關鍵旗標）
  remark,         // D5申請單備註（獨立，不連動 iq.remark）
  attachmentName  // D5申請單附件名稱（獨立，不連動 iq.attachmentName）
}

// 各製程資料（不變）
machining:  { consumption: { self: [], outsource: [] }, output: { self: [], outsource: [] } }
machining2: { consumption: [], output: [] }        // 含 execMode 欄位
coating:    { consumption: { self: [], outsource: [] }, output: { self: [], outsource: [] } }
purification: 同 coating 結構

workOrders: { [key]: { no, proc, execType } }

qualityData: { [lotNo]: { [paramKey]: value } }

// 庫存 Modal（新增 mode 欄位）
picker: {
  show, keyword,
  selIds: Set,    // 已選庫存 id
  proc, section, execMode,
  mode: 'consumption' | 'lock'  // 區分耗用與鎖料模式
}
```

---

## 5. 元件責任表

| 元件 | 責任 | Props | Emits |
|------|------|-------|-------|
| `App.vue` | 全局狀態、路由 Tab、Modal 管理 | — | — |
| `ConsumptionTab.vue` | 耗用 V1 顯示與互動 | `consumption`, `procName` | `openPicker`, `cancelConsumption` |
| `ConsumptionTabV2.vue` | 耗用 V2（扁平模式） | `rows` | `openPicker`, `cancelConsumption` |
| `OutputTab.vue` | 產出 V1 容器（自產/委外） | `output`, `procName` | `add`, `update`, `delete` |
| `OutputSection.vue` | 可重用產出區塊 | `rows`, `section`, `title` | `add`, `update`, `delete`, `openModal` |
| `OutputTabV2.vue` | 產出 V2（含執行方式） | `rows` | `add`, `update`, `delete` |
| `QualityTab.vue` | 品檢資訊輸入 | `allLots`, `qualityData` | `change` |

---

## 6. 關鍵函式清單

### App.vue

| 函式 | 說明 |
|------|------|
| `doQuery()` | 依單號查詢，支援 IQ/D5 兩種格式 |
| `iqCreate()` | 建立詢價單，自動產生 IQ 單號 |
| `iqFileChange(e)` | 處理詢價單附件選擇 |
| `iqOpenLockPicker()` | 開啟庫存 Modal（鎖料模式，顯示全部倉庫） |
| `iqConfirmToD5()` | 確認轉換，產生 D5 單號並切換至 D5 Tab |
| `d5FileChange(e)` | 處理 D5 申請單附件選擇（d5.attachmentName，與詢價單附件獨立） |
| `d5Save()` | 儲存 D5 申請單，記錄操作日誌（含受理狀態） |
| `addLog(msg, level, key)` | 新增系統日誌至 iq.sysLog（依 key 去重） |
| `removeLog(key)` | 移除系統日誌 |
| `genWorkOrderNo()` | 產生 WO-YYYYMM-NNNN |
| `ensureWorkOrder(key, proc, execType)` | 若不存在則建立工單 |
| `removeWorkOrder(key)` | 刪除工單 |
| `openPicker({section}, procName)` | 開啟庫存 Modal（耗用模式，V1） |
| `openPickerV2({execMode})` | 開啟庫存 Modal（耗用模式，V2） |
| `filterInv()` | 依關鍵字過濾庫存（lock 模式顯示全部） |
| `getBaseInv()` | 依 mode/section 過濾可用倉庫 |
| `pickerToggleOne(i)` | 單筆選取切換 |
| `pickerToggleAll(checked)` | 全選/取消全選 |
| `confirmPicker()` | 確認選取：lock 模式→鎖料；consumption 模式→耗用 |
| `cancelConsumption(procName, {section, ids})` | 刪除耗用記錄 |
| `handleOutputAdd(proc, {section, row})` | 新增產出記錄 |
| `handleOutputUpdate(proc, {section, idx, row})` | 修改產出記錄 |
| `handleOutputDelete(proc, {section, ids})` | 刪除產出記錄 |
| `handleQualityChange({lotNo, paramKey, value})` | 更新品檢參數 |

### Computed Properties

| 屬性 | 說明 |
|------|------|
| `visibleTabIndices` | 永遠包含 [0,1]；需 d5.saved && d5.accepted === 'Y' 且依 iq 委託類型決定後續顯示 Tab |
| `workOrderList` | 將 workOrders 物件轉為陣列 |
| `lotOverview` | 所有產出批號依製程分組 |
| `allLots` | 彙整所有製程的產出批號 |
| `hasMachSelf/Out`, `hasCoatSelf/Out`, `hasPureSelf/Out` | 偵測各製程是否有資料（驅動工單 watch） |

---

## 7. 商業規則

### 詢價單規則

- 交貨日為必填欄位（新增主檔前驗證）
- 詢價單新增主檔後，才可執行鎖料與轉 D5 操作
- 鎖料批號：重複 lotNo 不重複加入，可多次追加
- 一張詢價單只能轉換一次 D5 申請單（d5.id 一旦產生，按鈕不可重複觸發）

### D5申請單規則

- **受理與否 + 儲存**：設定「受理與否」後必須按下「儲存」，後續製程頁籤才會根據條件顯示；修改受理值後儲存狀態自動重設，需重新儲存
- **備註**：D5 申請單備註（d5.remark）與詢價單備註（iq.remark）各自獨立管理
- **附件**：D5 申請單附件（d5.attachmentName）與詢價單附件（iq.attachmentName）各自獨立管理
- **儲存**：工具列提供「儲存」按鈕（d5Save），點擊後記錄日誌並顯示成功訊息
- 交期、委託類型、爐號、費用、客戶、客供料、鎖料批號為唯讀，即時反映詢價單（iq.*）
- 工單清單、產出批號總覽、系統備註可在 D5 Tab 查看
- 系統日誌（iq.sysLog）由詢價單與 D5 共用，完整記錄全流程操作

### 耗用規則

| 執行方式 | 可用倉庫 |
|----------|----------|
| 自產（Self） | BB（碳材料生產工廠）|
| 委外（Outsource） | 除 BBPN 外全部倉庫 |
| 鎖料（Lock） | 全部倉庫（不限） |

### 工單規則

- 第一筆耗用或產出新增時，自動產生工單
- 所有耗用及產出刪除後，工單自動移除
- 每個（製程 × 執行方式）組合對應一張工單

### 品檢規則

- 品檢參數依產品代碼不同（6~15 個欄位）
- 每個參數設有規格值（spec）供比對
- 品檢資料以批號 × 參數鍵值儲存

---

## 8. 資料庫對應表（DB Schema）

| 資料表 | 對應功能 | 主要欄位 |
|--------|----------|----------|
| `T_IQ_APPLICATION` | 詢價單主表（新增）| iq_id, delivery_date, type_machining/coating/purification, furnace_no, cost, customer_code |
| `T_IQ_LOCKED_LOT` | 詢價單鎖料批號（新增）| iq_id, lot_no, item_no, qty, unit |
| `T_D5_APPLICATION` | D5 申請單主表 | appl_id, iq_id（關聯詢價單）, delivery_date, ..., d5_status |
| `T_D5_CONSUMPTION` | 耗用記錄 | appl_id, process_type, exec_mode, item_no, warehouse, lot_no, qty, unit |
| `T_D5_OUTPUT` | 產出記錄 | appl_id, process_type, exec_mode, product_key, spec1/2/3, qty, lot_no |
| `T_D5_WORK_ORDER` | 工單 | wo_id, appl_id, process_type, exec_mode, wo_status |
| `T_D5_QUALITY` | 品檢資訊 | quality_id, appl_id, lot_no, product_key, param_key, actual_value |
| `T_D5_SYS_LOG` | 系統日誌（詢價單+D5共用）| appl_id, iq_id, time, msg, level, key |

---

## 9. 常數與設定參數

### 客戶代碼（CUSTOMERS）

| 名稱 | 代碼 |
|------|------|
| 正達 | GT |
| 永泉 | YQ |
| 盛新 | TS |
| 漢民 | HM |
| 環球晶 | GW |
| 合晶 | WW |
| 臺譜 | TP |

### 倉庫代碼（WAREHOUSE_LABEL）

| 代碼 | 名稱 | 鎖料可用 | 自產耗用 | 委外耗用 |
|------|------|----------|----------|----------|
| BB | 碳材料生產工廠 | ✓ | ✓ | ✓ |
| BBPN | D8屏南儲區 | ✓ | ✗ | ✗ |
| BBOUTTMP | 委外暫存庫 | ✓ | ✗ | ✓ |

### 編號格式

| 類型 | 格式 | 範例 |
|------|------|------|
| 詢價單號 | IQ-YYYYMM-NNNN | IQ-202604-3721 |
| 申請單號 | D5-YYYYMM-NNNN | D5-202604-0851 |
| 工單號 | WO-YYYYMM-NNNN | WO-202604-5612 |

### Tab 索引對照（固定）

| Index | Tab 名稱 | 顯示條件 |
|-------|----------|----------|
| 0 | 詢價單 | 永遠顯示 |
| 1 | D5申請單 | 永遠顯示 |
| 2 | 機加工(耗用) | typeMachining + V1 |
| 3 | 機加工(產出) | typeMachining + V1 |
| 4 | 機加工(耗用)_version2 | typeMachining + V2 |
| 5 | 機加工(產出)_version2 | typeMachining + V2 |
| 6 | 鍍膜(耗用) | typeCoating |
| 7 | 鍍膜(產出) | typeCoating |
| 8 | 純化(耗用) | typePurification |
| 9 | 純化(產出) | typePurification |
| 10 | 品檢資訊 | 任一製程已選 |

---

## 10. 系統日誌格式

```javascript
{
  time:  "YYYY/MM/DD HH:mm:ss",
  msg:   "操作描述",
  level: "info" | "create" | "input" | "edit" | "cancel" | "warn" | "error",
  key:   "唯一識別鍵（用於去重，null=不去重）"
}
```

| level | 使用時機 |
|-------|----------|
| `create` | 建立詢價單、建立 D5 申請單、工單產生 |
| `input` | 耗用/產出/品檢/鎖料資料輸入 |
| `edit` | 修改操作 |
| `cancel` | 取消耗用 |
| `info` | 查詢結果、一般提示 |
| `warn` | 一般警告 |
| `error` | 錯誤發生 |

---

## 11. 待整合事項

| 項目 | 說明 |
|------|------|
| **後端 API** | 目前為純前端 in-memory 狀態，資料重整頁面後遺失 |
| **詢價單 API** | T_IQ_APPLICATION / T_IQ_LOCKED_LOT 需新增後端對應 |
| **庫存資料** | `allInv` 目前為 7 筆 mock 資料，需串接真實庫存 API |
| **鎖料後庫存扣減** | 鎖料完成後應從庫存可用量扣減，目前未實作 |
| **工單寫入** | 工單號已產生但未寫入後端 |
| **附件上傳** | 檔案選取事件已實作，但未串接儲存路徑 |
| **客戶下拉** | 目前為固定清單，可改為 API 動態載入 |
| **入庫/移庫 Modal** | 產出 Tab 按鈕已佔位，視窗邏輯尚未實作 |

---

*文件更新日期：2026-04-29 | 由 Claude Code 自動分析產生*

---

## 12. 變更記錄

| 日期 | 版本 | 說明 |
|------|------|------|
| 2026-04-29 | v1.1 | D5申請單新增：受理與否欄位、儲存功能、備註與附件獨立管理、受理為「是」才顯示後續製程頁籤 |
| 2026-04-29 | v1.2 | 調整頁籤顯示條件：需「儲存」後才根據受理結果顯示後續頁籤；受理值變動時自動重設儲存狀態 |
