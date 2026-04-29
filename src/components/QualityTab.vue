<template>
  <div>
    <!-- 製程選擇列 -->
    <div class="proc-bar">
      <span class="proc-label">製程</span>
      <div class="proc-btns">
        <button
          v-for="p in PROCS" :key="p.key"
          :class="['proc-btn', { active: selectedProc === p.key }]"
          @click="selectedProc = p.key"
        >{{ p.label }}</button>
      </div>
    </div>

    <div v-if="!selectedProc" class="no-hint">請先選擇製程以查看品檢資訊</div>

    <template v-else>
      <div v-if="!filteredGroups.length" class="no-hint">
        此製程目前無產出批號，請先至對應產出頁籤新增資料
      </div>

      <!-- 依產品分組顯示 -->
      <div v-for="group in filteredGroups" :key="group.productKey" class="product-section">
        <div class="product-hdr">
          <span class="product-badge">產品</span>
          <span class="product-title">{{ PRODUCT_LABEL[group.productKey] ?? group.productKey }}</span>
          <span class="product-meta">{{ selectedProcLabel }} ｜ {{ group.lots.length }} 批</span>
        </div>

        <div class="table-wrap">
          <table class="dtable">
            <thead>
              <tr>
                <th class="col-lot">批號</th>
                <th v-for="col in cols(group.productKey)" :key="col.key" class="col-param">{{ col.label }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="lot in group.lots" :key="lot.lotNo" class="data-row">
                <td class="td-lot">{{ lot.lotNo }}</td>
                <td v-for="col in cols(group.productKey)" :key="col.key">
                  <input
                    type="text"
                    class="cell-input"
                    :value="getVal(lot.lotNo, col.key)"
                    @change="onChange(lot.lotNo, col.key, $event.target.value)"
                    :placeholder="col.std ?? ''"
                  />
                </td>
              </tr>
            </tbody>
            <tfoot>
              <tr class="spec-row">
                <td class="spec-hdr">內控規格值</td>
                <td v-for="col in cols(group.productKey)" :key="col.key" class="spec-val">
                  {{ col.std ?? '—' }}
                </td>
              </tr>
            </tfoot>
          </table>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  allLots:     { type: Array,  default: () => [] },
  qualityData: { type: Object, default: () => ({}) },
})
const emit = defineEmits(['change'])

// ── 製程定義 ──────────────────────────────────────
const PROCS = [
  { key: '機加工', label: '機加工(產出)' },
  { key: '鍍膜',   label: '鍍膜(產出)'   },
  { key: '純化',   label: '純化(產出)'   },
]
const selectedProc = ref('')
const selectedProcLabel = computed(() => PROCS.find(p => p.key === selectedProc.value)?.label ?? '')

// ── 產品品名對照 ──────────────────────────────────
const PRODUCT_LABEL = {
  'ACS15U': 'ACS15U - 超級電容ACS15U',
  'GC-001': 'GC-001 - 石墨化碳材料',
  'CB-050': 'CB-050 - 碳微球CB-050',
  'PG-100': 'PG-100 - 純化石墨PG-100',
  'MC-200': 'MC-200 - 機加工碳塊MC-200',
  'CF-300': 'CF-300 - 碳纖維CF-300',
  'GR-400': 'GR-400 - 石墨電極GR-400',
  'BC-500': 'BC-500 - 電池碳材BC-500',
}

// ── 各品項品檢欄位定義 ────────────────────────────
const QUALITY_COLS = {
  'GC-001': [
    { key: 'yield',  label: 'Yield%',        std: '>97'       },
    { key: 'ti',     label: 'TI%',           std: '>97'       },
    { key: 'qi',     label: 'QI%',           std: '>95'       },
    { key: 'fc',     label: 'F.C.%',         std: '>95.50'    },
    { key: 'ash',    label: 'Ash%',          std: '<0.2'      },
    { key: 'vm',     label: 'VM%',           std: '6.0-8.5'   },
    { key: 'h2o',    label: 'H2O%',          std: '<0.30'     },
    { key: 'tapd',   label: 'Tap D.(g/ml)',  std: '≥0.75'     },
    { key: 'lt5',    label: '%<5μm,%',       std: '<2.0'      },
    { key: 'lt88',   label: '%<88μm,%',      std: '100'       },
    { key: 'd10',    label: 'D10%,μm',       std: '17-21'     },
    { key: 'd50',    label: 'D50%,μm',       std: '25.3-28.0' },
    { key: 'd90',    label: 'D90%,μm',       std: '35-41'     },
    { key: 'd95',    label: 'D95%,μm',       std: '≤65'       },
    { key: 'd100',   label: 'D100%,μm',      std: ''          },
  ],
  'ACS15U': [
    { key: 'yield',  label: 'Yield%',           std: '>90'   },
    { key: 'bet',    label: 'BET(m²/g)',         std: '>1500' },
    { key: 'cap',    label: '電容量(F/g)',        std: '>100'  },
    { key: 'resist', label: '電阻率(mΩ·cm)',      std: '<5'    },
    { key: 'h2o',    label: '水分(%)',            std: '<2'    },
    { key: 'ash',    label: '灰分(%)',            std: '<0.5'  },
  ],
  'CB-050': [
    { key: 'yield',      label: 'Yield%',         std: '>85'      },
    { key: 'd50',        label: 'D50(μm)',         std: '45-55'    },
    { key: 'd90',        label: 'D90(μm)',         std: '<100'     },
    { key: 'tapd',       label: '振實密度(g/ml)',  std: '>0.4'     },
    { key: 'sphericity', label: '球形度',          std: '>0.90'    },
    { key: 'h2o',        label: '水分(%)',         std: '<1'       },
    { key: 'ash',        label: '灰分(%)',         std: '<0.3'     },
  ],
  'PG-100': [
    { key: 'purity', label: '純度(%)',          std: '>99.9' },
    { key: 'ash',    label: '灰分(ppm)',        std: '<100'  },
    { key: 'mag',    label: '磁性物質(ppm)',    std: '<50'   },
    { key: 'd50',    label: 'D50(μm)',          std: '20-30' },
    { key: 'bet',    label: '比表面積(m²/g)',   std: '<10'   },
    { key: 'h2o',    label: '水分(%)',          std: '<0.5'  },
  ],
  'MC-200': [
    { key: 'weight',  label: '重量(g)',        std: ''     },
    { key: 'length',  label: '長(mm)',         std: ''     },
    { key: 'width',   label: '寬(mm)',         std: ''     },
    { key: 'height',  label: '高(mm)',         std: ''     },
    { key: 'density', label: '密度(g/cm³)',    std: '>1.7' },
    { key: 'appear',  label: '外觀',           std: 'OK'   },
  ],
  'CF-300': [
    { key: 'tensile', label: '拉伸強度(MPa)',   std: '>3000' },
    { key: 'modulus', label: '拉伸模量(GPa)',   std: '>200'  },
    { key: 'elong',   label: '斷裂伸長率(%)',   std: '>1.5'  },
    { key: 'lindens', label: '線密度(tex)',     std: ''      },
    { key: 'ash',     label: '灰分(%)',         std: '<0.3'  },
  ],
  'GR-400': [
    { key: 'resist',  label: '電阻率(μΩm)',    std: '<8'   },
    { key: 'flex',    label: '抗折強度(MPa)',  std: '>10'  },
    { key: 'modulus', label: '彈性模量(GPa)',  std: ''     },
    { key: 'ash',     label: '灰分(%)',        std: '<0.3' },
    { key: 'density', label: '體積密度(g/cm³)',std: '>1.6' },
  ],
  'BC-500': [
    { key: 'yield', label: 'Yield%',          std: '>92'  },
    { key: 'ice',   label: '首效(%)',          std: '>88'  },
    { key: 'cap',   label: '容量(mAh/g)',      std: '>340' },
    { key: 'd50',   label: 'D50(μm)',          std: '15-25'},
    { key: 'bet',   label: '比表面積(m²/g)',   std: '<2'   },
    { key: 'tapd',  label: '振實密度(g/ml)',   std: '>0.8' },
  ],
}

function cols(productKey) {
  return QUALITY_COLS[productKey] ?? [{ key: 'remark', label: '備註', std: '' }]
}

// ── 依製程過濾並依產品分組 ────────────────────────
const filteredGroups = computed(() => {
  const filtered = props.allLots.filter(l => l.procName === selectedProc.value)
  const map = new Map()
  filtered.forEach(lot => {
    if (!map.has(lot.productKey)) map.set(lot.productKey, [])
    map.get(lot.productKey).push(lot)
  })
  return [...map.entries()].map(([productKey, lots]) => ({ productKey, lots }))
})

// ── 品檢數值存取 ──────────────────────────────────
function getVal(lotNo, paramKey) {
  return props.qualityData[lotNo]?.[paramKey] ?? ''
}
function onChange(lotNo, paramKey, value) {
  emit('change', { lotNo, paramKey, value })
}
</script>

<style scoped>
/* 製程選擇列 */
.proc-bar {
  background: #d8e8f8; border-bottom: 1px solid #b0c4de;
  padding: 7px 14px; display: flex; align-items: center; gap: 12px;
}
.proc-label {
  background: #3a6abf; color: #fff;
  padding: 2px 10px; border-radius: 3px; font-size: 12px; white-space: nowrap;
}
.proc-btns { display: flex; gap: 6px; }
.proc-btn {
  padding: 4px 14px; font-size: 12px; font-family: inherit;
  border: 1px solid #8aaad0; border-radius: 3px;
  background: #fff; color: #3a6abf; cursor: pointer;
  transition: background .15s, color .15s;
}
.proc-btn:hover  { background: #e0ecff; }
.proc-btn.active { background: #3a6abf; color: #fff; border-color: #2a5aaf; font-weight: bold; }

.no-hint { padding: 40px; text-align: center; color: #aaa; font-size: 13px; font-style: italic; }

/* 產品區塊 */
.product-section { margin: 10px 12px 16px; border: 1px solid #b8cfe8; border-radius: 4px; overflow: hidden; }
.product-hdr {
  background: linear-gradient(90deg, #2a5aaf, #4a7acf);
  color: #fff; padding: 6px 14px;
  display: flex; align-items: center; gap: 10px;
}
.product-badge {
  background: rgba(255,255,255,.25); font-size: 11px;
  padding: 1px 7px; border-radius: 10px;
}
.product-title { font-weight: bold; font-size: 13px; }
.product-meta  { font-size: 12px; opacity: .85; margin-left: auto; }

/* 表格 */
.table-wrap { overflow-x: auto; }
.dtable { width: 100%; border-collapse: collapse; font-size: 12px; }
.dtable th {
  background: #4a7acf; color: #fff;
  padding: 5px 8px; border: 1px solid #3a6abf;
  white-space: nowrap; text-align: center;
}
.col-lot   { min-width: 130px; }
.col-param { min-width: 90px; }
.dtable td { padding: 3px 5px; border: 1px solid #d0dce8; text-align: center; }
.td-lot    { font-weight: 500; color: #1a4a9e; background: #eef4ff !important; white-space: nowrap; }
.data-row:nth-child(even) td { background: #f5f8ff; }
.data-row:hover td           { background: #dceeff; }

/* 規格基準列 */
.spec-row td { background: #fff8e6 !important; color: #7a5000; font-size: 11px; font-style: italic; }
.spec-hdr    { font-weight: bold; text-align: right; padding-right: 8px; white-space: nowrap; color: #5a3800; }
.spec-val    { text-align: center; }

/* 儲存格輸入 */
.cell-input {
  width: 100%; padding: 2px 5px;
  border: 1px solid #b0c4de; border-radius: 2px;
  font-size: 12px; font-family: inherit; background: #fff;
  box-sizing: border-box; text-align: center;
}
.cell-input:focus { border-color: #3a6abf; outline: none; background: #f0f8ff; }
.cell-input::placeholder { color: #bbb; font-style: italic; }
</style>
