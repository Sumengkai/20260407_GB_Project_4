<template>
  <div>
    <div class="section-bar">
      <span class="section-title">產出明細</span>
      <button class="btn btn-light" @click="handleAdd">新增</button>
      <button class="btn btn-warn"   :disabled="!checkedIds.size" @click="handleEdit">修改</button>
      <button class="btn btn-danger" :disabled="!checkedIds.size" @click="handleDelete">
        刪除<span v-if="checkedIds.size" class="badge">{{ checkedIds.size }}</span>
      </button>
      <div class="bar-divider"></div>
      <select v-model="actionMode" class="action-select">
        <option value="outsource">委外</option>
        <option value="self">產出</option>
      </select>
      <button
        :class="['btn', actionMode === 'outsource' ? 'btn-teal' : 'btn-green']"
        @click="openModal(actionMode === 'outsource' ? '模擬移庫視窗' : '模擬入庫視窗')"
      >{{ actionMode === 'outsource' ? '打開視窗做移庫' : '打開視窗做入庫' }}</button>
    </div>

    <div class="table-wrap">
      <table class="dtable">
        <thead>
          <tr>
            <th width="36">勾選</th>
            <th>執行方式</th>
            <th>品名</th>
            <th>規格1</th><th>規格2</th><th>規格3</th>
            <th>數量</th><th>批號</th><th>備註</th>
          </tr>
        </thead>
        <tbody>
          <!-- 已存資料列 -->
          <tr v-for="row in rows" :key="row.id"
              :class="{ 'row-checked': checkedIds.has(row.id) }">
            <td><input type="checkbox" :checked="checkedIds.has(row.id)" @change="toggleCheck(row.id)" /></td>
            <td>
              <select v-if="editForms[row.id]" v-model="editForms[row.id].execMode" class="cell-input cell-select req">
                <option value="01">01 - 自產</option>
                <option value="02">02 - 委外</option>
              </select>
            </td>
            <td>
              <select v-if="editForms[row.id]" v-model="editForms[row.id].productKey" class="cell-input cell-select req">
                <option value="">請選擇品名</option>
                <option v-for="p in PRODUCTS" :key="p.key" :value="p.key">{{ p.label }}</option>
              </select>
            </td>
            <td><input v-if="editForms[row.id]" v-model="editForms[row.id].spec1"  class="cell-input" placeholder="規格1" /></td>
            <td><input v-if="editForms[row.id]" v-model="editForms[row.id].spec2"  class="cell-input" placeholder="規格2" /></td>
            <td><input v-if="editForms[row.id]" v-model="editForms[row.id].spec3"  class="cell-input" placeholder="規格3" /></td>
            <td><input v-if="editForms[row.id]" v-model="editForms[row.id].qty"    class="cell-input req" type="number" placeholder="數量*" /></td>
            <td><input v-if="editForms[row.id]" v-model="editForms[row.id].lotNo"  class="cell-input req" placeholder="批號*" /></td>
            <td><input v-if="editForms[row.id]" v-model="editForms[row.id].remark" class="cell-input" placeholder="備註" /></td>
          </tr>

          <!-- 新增輸入列 -->
          <tr class="new-row" :class="{ 'new-row-checked': newRow.checked }">
            <td><input type="checkbox" v-model="newRow.checked" /></td>
            <td>
              <select v-model="newRow.execMode" class="cell-input cell-select req">
                <option value="01">01 - 自產</option>
                <option value="02">02 - 委外</option>
              </select>
            </td>
            <td>
              <select v-model="newRow.productKey" class="cell-input cell-select req">
                <option value="">請選擇品名</option>
                <option v-for="p in PRODUCTS" :key="p.key" :value="p.key">{{ p.label }}</option>
              </select>
            </td>
            <td><input v-model="newRow.spec1"  class="cell-input" placeholder="規格1" /></td>
            <td><input v-model="newRow.spec2"  class="cell-input" placeholder="規格2" /></td>
            <td><input v-model="newRow.spec3"  class="cell-input" placeholder="規格3" /></td>
            <td><input v-model="newRow.qty"    class="cell-input req" type="number" placeholder="數量*" /></td>
            <td><input v-model="newRow.lotNo"  class="cell-input req" placeholder="批號*" /></td>
            <td><input v-model="newRow.remark" class="cell-input" placeholder="備註" /></td>
          </tr>

          <tr v-if="!rows.length">
            <td colspan="9" class="hint-row">請先選擇【執行方式】，再填寫其他欄位，勾選後點擊「新增」</td>
          </tr>
        </tbody>
      </table>
    </div>

  <!-- Modal -->
  <div v-if="modalText" class="sim-overlay" @click.self="modalText = ''">
    <div class="sim-box">
      <span class="sim-title">{{ modalText }}</span>
      <button class="sim-close" @click="modalText = ''">✕</button>
    </div>
  </div>
  </div>
</template>

<script setup>
import { ref, reactive, watch } from 'vue'

const props = defineProps({
  rows: { type: Array, default: () => [] },
})
const emit = defineEmits(['add', 'update', 'delete-items', 'open-transfer', 'open-inbound'])
const actionMode = ref('outsource')
const modalText  = ref('')
function openModal(text) { modalText.value = text }

const PRODUCTS = [
  { key: 'ACS15U', label: 'ACS15U - 超級電容ACS15U' },
  { key: 'GC-001', label: 'GC-001 - 石墨化碳材料' },
  { key: 'CB-050', label: 'CB-050 - 碳微球CB-050' },
  { key: 'PG-100', label: 'PG-100 - 純化石墨PG-100' },
  { key: 'MC-200', label: 'MC-200 - 機加工碳塊MC-200' },
  { key: 'CF-300', label: 'CF-300 - 碳纖維CF-300' },
  { key: 'GR-400', label: 'GR-400 - 石墨電極GR-400' },
  { key: 'BC-500', label: 'BC-500 - 電池碳材BC-500' },
]

const editForms  = reactive({})
const checkedIds = reactive(new Set())

watch(() => props.rows, (newRows) => {
  newRows.forEach(row => {
    if (!editForms[row.id]) editForms[row.id] = { ...row }
  })
  const ids = new Set(newRows.map(r => r.id))
  Object.keys(editForms).forEach(k => {
    if (!ids.has(Number(k))) delete editForms[k]
  })
}, { immediate: true, deep: true })

function toggleCheck(id) {
  if (checkedIds.has(id)) checkedIds.delete(id)
  else checkedIds.add(id)
}

function emptyForm() { return { execMode: '01', productKey: '', spec1: '', spec2: '', spec3: '', qty: null, lotNo: '', remark: '' } }
const newRow = reactive({ checked: false, ...emptyForm() })
function resetNew() { Object.assign(newRow, { checked: false, ...emptyForm() }) }

function handleAdd() {
  if (!newRow.checked)    { alert('請先勾選新增列的方框'); return }
  if (!newRow.execMode)   { alert('請選擇執行方式'); return }
  if (!newRow.productKey) { alert('請選擇品名'); return }
  if (!newRow.qty)        { alert('數量為必填欄位'); return }
  if (!newRow.lotNo)      { alert('批號為必填欄位'); return }
  emit('add', { ...emptyForm(), ...newRow })
  resetNew()
}

function handleEdit() {
  if (!checkedIds.size) return
  let hasError = false
  checkedIds.forEach(id => {
    const form = editForms[id]
    if (!form || !form.execMode || !form.productKey || !form.qty || !form.lotNo) { hasError = true; return }
    const idx = props.rows.findIndex(r => r.id === id)
    if (idx >= 0) emit('update', { idx, row: { ...form } })
  })
  if (hasError) { alert('勾選列中有必填欄位未填，請確認後再修改'); return }
  checkedIds.clear()
}

function handleDelete() {
  if (!checkedIds.size) return
  if (!confirm(`確定要刪除已勾選的 ${checkedIds.size} 筆資料？`)) return
  emit('delete-items', { ids: [...checkedIds] })
  checkedIds.clear()
}
</script>

<style scoped>
.section-bar {
  background: linear-gradient(90deg, #3a6abf, #5586d0);
  color: #fff; padding: 6px 14px;
  display: flex; align-items: center; gap: 8px;
  border-bottom: 1px solid #2a5aaf;
}
.section-title { font-weight: bold; font-size: 13px; min-width: 40px; margin-right: 4px; }
.table-wrap { padding: 8px 12px; overflow-x: auto; }

.dtable { width: 100%; border-collapse: collapse; font-size: 12px; min-width: 800px; }
.dtable th { background: #3a6abf; color: #fff; padding: 5px 8px; border: 1px solid #2a5aaf; white-space: nowrap; text-align: center; }
.dtable td { padding: 3px 6px; border: 1px solid #d0dce8; text-align: center; }
.dtable tr:nth-child(even) td { background: #f0f6ff; }
.dtable tr:hover td           { background: #dceeff; }
.row-checked td      { background: #d4e8ff !important; }
.new-row td          { background: #f0fff4; }
.new-row-checked td  { background: #d0f0e0 !important; }
.hint-row { color: #aaa; text-align: center; padding: 12px; font-style: italic; }

.cell-input { width: 100%; padding: 2px 5px; border: 1px solid #b0c4de; border-radius: 2px; font-size: 12px; font-family: inherit; background: #fff; box-sizing: border-box; }
.cell-input:focus { border-color: #3a6abf; outline: none; }
.cell-input.req   { border-color: #c0392b; }
.cell-select      { cursor: pointer; }

.btn { padding: 3px 10px; font-size: 12px; font-family: inherit; border: 1px solid; border-radius: 3px; cursor: pointer; display: inline-flex; align-items: center; gap: 4px; }
.btn:hover    { filter: brightness(1.1); }
.btn:disabled { opacity: .4; cursor: not-allowed; filter: none; }
.btn-light  { background: #fff; color: #2a5aaf; border-color: rgba(255,255,255,.7); }
.btn-light:hover { background: #e8f0ff; }
.btn-warn    { background: #e67e22; color: #fff; border-color: #ca6f1e; }
.btn-danger  { background: #c0392b; color: #fff; border-color: #a93226; }
.btn-teal    { background:#fff; color:#000; border-color:rgba(255,255,255,.7); }
.btn-teal:hover  { background:#e8f0ff; }
.btn-green   { background:#fff; color:#000; border-color:rgba(255,255,255,.7); }
.btn-green:hover { background:#e8f0ff; }
.bar-divider { width:1px; background:rgba(255,255,255,.4); margin:0 6px; align-self:stretch; }

.sim-overlay { position:fixed; inset:0; background:rgba(0,0,0,.45); display:flex; align-items:center; justify-content:center; z-index:1000; }
.sim-box     { background:#fff; border-radius:8px; padding:60px 80px; text-align:center; position:relative; box-shadow:0 8px 32px rgba(0,0,0,.25); }
.sim-title   { font-size:36px; font-weight:bold; color:#1a3a6e; }
.sim-close   { position:absolute; top:12px; right:16px; background:none; border:none; font-size:18px; cursor:pointer; color:#888; }
.sim-close:hover { color:#333; }
.action-select {
  padding: 3px 8px; border: 1px solid rgba(255,255,255,.5); border-radius: 3px;
  font-size: 12px; font-family: inherit; background: #fff; color: #1a3a6e; cursor: pointer;
}
.badge { background: rgba(255,255,255,.9); color: #c0392b; font-size: 11px; font-weight: bold; padding: 0 5px; border-radius: 8px; min-width: 16px; text-align: center; }
</style>
