# 塊材申請作業 — 資料表結構設計

> 系統名稱：塊材申請作業  
> 版本：v1.0  
> 設計日期：2026-04-10  

---

## 表格索引

| 表名 | 說明 |
|------|------|
| [T_D5_APPLICATION](#1-t_d5_application--主申請單表) | 主申請單表 |
| [T_D5_CONSUMPTION](#2-t_d5_consumption--耗用明細表) | 耗用明細表 |
| [T_D5_OUTPUT](#3-t_d5_output--產出明細表) | 產出明細表 |
| [T_D5_LOCKED_LOT](#4-t_d5_locked_lot--鎖料批號表) | 鎖料批號表 |
| [T_D5_WORK_ORDER](#5-t_d5_work_order--工單表) | 工單表 |
| [T_D5_QUALITY](#6-t_d5_quality--品檢資訊表) | 品檢資訊表 |
| [T_D5_SYS_LOG](#7-t_d5_sys_log--系統日誌表) | 系統日誌表 |

---

## 1. T_D5_APPLICATION — 主申請單表

> 塊材申請單主檔，每筆對應一張申請單

| # | Field | DataType | Interface | Description | FieldWidth (DB) | Format | Default |
|---|-------|----------|-----------|-------------|-----------------|--------|---------|
| 1 | appl_id | varchar | readonly | 申請單單號（主鍵） | 20 | D5-YYYYMM-NNNN | — |
| 2 | delivery_date | date | date | 交期（必填） | — | YYYY-MM-DD | — |
| 3 | attachment_name | varchar | file | 附件名稱 | 200 | — | NULL |
| 4 | attachment_path | varchar | — | 附件儲存路徑 | 500 | — | NULL |
| 5 | type_machining | char | checkbox | 委託類型：機加工 | 1 | Y/N | 'N' |
| 6 | type_coating | char | checkbox | 委託類型：鍍膜 | 1 | Y/N | 'N' |
| 7 | type_purification | char | checkbox | 委託類型：純化 | 1 | Y/N | 'N' |
| 8 | furnace_no | varchar | input | 爐次 | 50 | — | NULL |
| 9 | cost | decimal | input | 費用（元） | 12,2 | — | 0.00 |
| 10 | customer_code | varchar | select | 客戶代碼 | 10 | GT/YQ/TS/HM/GW/WW/TP | NULL |
| 11 | remark | text | textarea | 備註 | — | — | NULL |
| 12 | client_material | text | textarea | 客供料說明 | — | — | NULL |
| 13 | machining_ver | char | select | 機加工版本（V1/V2/BOTH） | 5 | V1/V2/BOTH | 'V1' |
| 14 | status | varchar | readonly | 單據狀態 | 10 | DRAFT/ACTIVE/CLOSED | 'DRAFT' |
| 15 | created_by | varchar | readonly | 建立人員 | 20 | — | — |
| 16 | created_at | datetime | readonly | 建立時間 | — | YYYY-MM-DD HH:mm:ss | CURRENT_TIMESTAMP |
| 17 | updated_by | varchar | readonly | 最後修改人員 | 20 | — | NULL |
| 18 | updated_at | datetime | readonly | 最後修改時間 | — | YYYY-MM-DD HH:mm:ss | NULL |

**主鍵**：`appl_id`  
**索引**：`delivery_date`、`customer_code`、`status`

---

## 2. T_D5_CONSUMPTION — 耗用明細表

> 各製程的耗用材料明細，支援自產（self）與委外（outsource）

| # | Field | DataType | Interface | Description | FieldWidth (DB) | Format | Default |
|---|-------|----------|-----------|-------------|-----------------|--------|---------|
| 1 | cons_id | int | readonly | 耗用明細序號（主鍵，自動遞增） | — | — | AUTO_INCREMENT |
| 2 | appl_id | varchar | — | 申請單單號（FK → T_D5_APPLICATION） | 20 | D5-YYYYMM-NNNN | — |
| 3 | process_type | varchar | readonly | 製程別 | 15 | machining/coating/purification | — |
| 4 | exec_mode | char | select | 執行方式 | 2 | 01=自產 / 02=委外 | '01' |
| 5 | item_no | varchar | input | 品號（庫存代碼） | 30 | — | — |
| 6 | item_name | varchar | readonly | 品名 | 100 | — | — |
| 7 | warehouse | varchar | select | 倉庫別 | 15 | BB/BBPN/BBOUTTMP | — |
| 8 | lot_no | varchar | input | 批號 | 30 | — | — |
| 9 | qty | decimal | input | 耗用數量 | 12,4 | — | 0.0000 |
| 10 | unit | varchar | select | 單位 | 5 | kg/pcs | — |
| 11 | is_cancelled | char | readonly | 是否取消 | 1 | Y/N | 'N' |
| 12 | sort_order | int | — | 顯示排序 | — | — | 0 |
| 13 | created_at | datetime | readonly | 建立時間 | — | YYYY-MM-DD HH:mm:ss | CURRENT_TIMESTAMP |

**主鍵**：`cons_id`  
**外鍵**：`appl_id → T_D5_APPLICATION.appl_id`  
**索引**：`appl_id`、`process_type`、`exec_mode`  
**庫別規則**：exec_mode=01(自產) 僅允許 BB；exec_mode=02(委外) 排除 BBPN

---

## 3. T_D5_OUTPUT — 產出明細表

> 各製程的產出品項明細，支援自產與委外

| # | Field | DataType | Interface | Description | FieldWidth (DB) | Format | Default |
|---|-------|----------|-----------|-------------|-----------------|--------|---------|
| 1 | output_id | int | readonly | 產出明細序號（主鍵，自動遞增） | — | — | AUTO_INCREMENT |
| 2 | appl_id | varchar | — | 申請單單號（FK → T_D5_APPLICATION） | 20 | D5-YYYYMM-NNNN | — |
| 3 | process_type | varchar | readonly | 製程別 | 15 | machining/coating/purification | — |
| 4 | exec_mode | char | select | 執行方式 | 2 | 01=自產 / 02=委外 | '01' |
| 5 | product_key | varchar | select | 品名代碼（下拉選擇） | 20 | ACS15U/GC-001/CB-050/… | — |
| 6 | product_name | varchar | readonly | 品名（自動帶入） | 100 | — | — |
| 7 | spec1 | varchar | input | 規格1 | 50 | — | NULL |
| 8 | spec2 | varchar | input | 規格2 | 50 | — | NULL |
| 9 | spec3 | varchar | input | 規格3 | 50 | — | NULL |
| 10 | qty | decimal | input | 產出數量 | 12,4 | — | 0.0000 |
| 11 | lot_no | varchar | input | 產出批號 | 30 | — | NULL |
| 12 | rep_lot_no | varchar | input | 代表性批號 | 30 | — | NULL |
| 13 | remark | varchar | input | 備註 | 200 | — | NULL |
| 14 | sort_order | int | — | 顯示排序 | — | — | 0 |
| 15 | created_at | datetime | readonly | 建立時間 | — | YYYY-MM-DD HH:mm:ss | CURRENT_TIMESTAMP |

**主鍵**：`output_id`  
**外鍵**：`appl_id → T_D5_APPLICATION.appl_id`  
**索引**：`appl_id`、`process_type`、`lot_no`

---

## 4. T_D5_LOCKED_LOT — 鎖料批號表

> 記錄申請單中被鎖定（預留）使用的庫存批號

| # | Field | DataType | Interface | Description | FieldWidth (DB) | Format | Default |
|---|-------|----------|-----------|-------------|-----------------|--------|---------|
| 1 | lock_id | int | readonly | 鎖料序號（主鍵，自動遞增） | — | — | AUTO_INCREMENT |
| 2 | appl_id | varchar | — | 申請單單號（FK → T_D5_APPLICATION） | 20 | D5-YYYYMM-NNNN | — |
| 3 | lot_no | varchar | readonly | 批號 | 30 | — | — |
| 4 | item_no | varchar | readonly | 品號 | 30 | — | — |
| 5 | item_name | varchar | readonly | 品名 | 100 | — | — |
| 6 | qty | decimal | readonly | 鎖定數量 | 12,4 | — | 0.0000 |
| 7 | unit | varchar | readonly | 單位 | 5 | kg/pcs | — |
| 8 | locked_at | datetime | readonly | 鎖定時間 | — | YYYY-MM-DD HH:mm:ss | CURRENT_TIMESTAMP |

**主鍵**：`lock_id`  
**外鍵**：`appl_id → T_D5_APPLICATION.appl_id`  
**索引**：`appl_id`、`lot_no`

---

## 5. T_D5_WORK_ORDER — 工單表

> 依各製程×執行方式自動生成的工單，有耗用/產出明細即自動建立

| # | Field | DataType | Interface | Description | FieldWidth (DB) | Format | Default |
|---|-------|----------|-----------|-------------|-----------------|--------|---------|
| 1 | wo_id | varchar | readonly | 工單號（主鍵） | 20 | WO-YYYYMM-NNNN | — |
| 2 | appl_id | varchar | — | 申請單單號（FK → T_D5_APPLICATION） | 20 | D5-YYYYMM-NNNN | — |
| 3 | process_type | varchar | readonly | 製程別 | 15 | machining/coating/purification | — |
| 4 | exec_mode | char | readonly | 執行方式 | 2 | 01=自產 / 02=委外 | — |
| 5 | wo_status | varchar | readonly | 工單狀態 | 10 | OPEN/IN_PROG/DONE/CANCEL | 'OPEN' |
| 6 | auto_generated | char | readonly | 是否系統自動產生 | 1 | Y/N | 'Y' |
| 7 | created_at | datetime | readonly | 建立時間 | — | YYYY-MM-DD HH:mm:ss | CURRENT_TIMESTAMP |
| 8 | updated_at | datetime | readonly | 最後更新時間 | — | YYYY-MM-DD HH:mm:ss | NULL |

**主鍵**：`wo_id`  
**外鍵**：`appl_id → T_D5_APPLICATION.appl_id`  
**唯一鍵**：`(appl_id, process_type, exec_mode)`  
**索引**：`appl_id`、`wo_status`

---

## 6. T_D5_QUALITY — 品檢資訊表

> 各產出批號的品檢參數值，欄位依品項（product_key）動態對應

| # | Field | DataType | Interface | Description | FieldWidth (DB) | Format | Default |
|---|-------|----------|-----------|-------------|-----------------|--------|---------|
| 1 | quality_id | int | readonly | 品檢序號（主鍵，自動遞增） | — | — | AUTO_INCREMENT |
| 2 | appl_id | varchar | — | 申請單單號（FK → T_D5_APPLICATION） | 20 | D5-YYYYMM-NNNN | — |
| 3 | output_id | int | — | 產出明細序號（FK → T_D5_OUTPUT） | — | — | — |
| 4 | process_type | varchar | readonly | 製程別 | 15 | machining/coating/purification | — |
| 5 | lot_no | varchar | readonly | 批號（對應產出批號） | 30 | — | — |
| 6 | product_key | varchar | readonly | 品名代碼 | 20 | ACS15U/GC-001/CB-050/… | — |
| 7 | param_key | varchar | readonly | 品檢參數代碼 | 30 | Yield%/BET/D50/… | — |
| 8 | param_name | varchar | readonly | 品檢參數名稱 | 50 | — | — |
| 9 | spec_value | varchar | readonly | 內控規格值 | 50 | — | NULL |
| 10 | actual_value | varchar | input | 實測值 | 50 | — | NULL |
| 11 | is_pass | char | readonly | 是否合格 | 1 | Y/N/— | '—' |
| 12 | inspect_by | varchar | readonly | 品檢人員 | 20 | — | NULL |
| 13 | inspect_at | datetime | readonly | 品檢時間 | — | YYYY-MM-DD HH:mm:ss | NULL |
| 14 | updated_at | datetime | readonly | 最後更新時間 | — | YYYY-MM-DD HH:mm:ss | NULL |

**主鍵**：`quality_id`  
**外鍵**：`appl_id → T_D5_APPLICATION.appl_id`、`output_id → T_D5_OUTPUT.output_id`  
**唯一鍵**：`(output_id, param_key)`  
**索引**：`appl_id`、`lot_no`、`product_key`

> **品檢參數對照（依 product_key）**
>
> | product_key | 品名 | 品檢參數 |
> |-------------|------|----------|
> | GC-001 | — | Yield%, TI%, QI%, F.C.%, Ash%, VM%, H2O%, Tap D., D10, D50, D90, D95, D100 |
> | ACS15U | 超級電容 | Yield%, BET, 電容量, 電阻率, 水分, 灰分 |
> | CB-050 | — | Yield%, D50, D90, 振實密度, 球形度, 水分, 灰分 |
> | PG-100 | — | 純度, 灰分, 磁性物質, D50, 比表面積, 水分 |
> | MC-200 | — | 重量, 長寬高, 密度, 外觀 |
> | CF-300 | — | 拉伸強度, 拉伸模量, 斷裂伸長率, 線密度, 灰分 |
> | GR-400 | — | 電阻率, 抗折強度, 彈性模量, 灰分, 體積密度 |
> | BC-500 | — | Yield%, 首效, 容量, D50, 比表面積, 振實密度 |

---

## 7. T_D5_SYS_LOG — 系統日誌表

> 記錄申請單的所有操作歷程，依 log_key 去重

| # | Field | DataType | Interface | Description | FieldWidth (DB) | Format | Default |
|---|-------|----------|-----------|-------------|-----------------|--------|---------|
| 1 | log_id | int | readonly | 日誌序號（主鍵，自動遞增） | — | — | AUTO_INCREMENT |
| 2 | appl_id | varchar | — | 申請單單號（FK → T_D5_APPLICATION） | 20 | D5-YYYYMM-NNNN | — |
| 3 | log_key | varchar | readonly | 日誌唯一鍵（防止重複記錄） | 50 | — | — |
| 4 | log_level | varchar | readonly | 日誌等級 | 10 | info/warn/error | 'info' |
| 5 | log_msg | text | readonly | 日誌訊息內容 | — | — | — |
| 6 | operator | varchar | readonly | 操作人員 | 20 | — | — |
| 7 | log_time | datetime | readonly | 操作時間 | — | YYYY-MM-DD HH:mm:ss | CURRENT_TIMESTAMP |

**主鍵**：`log_id`  
**外鍵**：`appl_id → T_D5_APPLICATION.appl_id`  
**唯一鍵**：`(appl_id, log_key)`  
**索引**：`appl_id`、`log_time`

---

## 資料表關聯圖

```
T_D5_APPLICATION (appl_id) ─┬─ T_D5_CONSUMPTION (appl_id)
                             ├─ T_D5_OUTPUT      (appl_id)
                             │       └─ T_D5_QUALITY (output_id)
                             ├─ T_D5_LOCKED_LOT  (appl_id)
                             ├─ T_D5_WORK_ORDER  (appl_id)
                             └─ T_D5_SYS_LOG     (appl_id)
```

---

## 流水號規則

| 類型 | 格式 | 說明 |
|------|------|------|
| 申請單號 | `D5-YYYYMM-NNNN` | 例：D5-202604-0001，每月重置序號 |
| 工單號 | `WO-YYYYMM-NNNN` | 例：WO-202604-0001，每月重置序號 |

---

## 客戶代碼參照

| code | name |
|------|------|
| GT | 正達 |
| YQ | 永泉 |
| TS | 盛新 |
| HM | 漢民 |
| GW | 環球晶 |
| WW | 合晶 |
| TP | 臺譜 |
