<template>
  <div>
    <div class="section-bar">
      <span class="section-title">耗用明細</span>
      <span class="exec-label">執行方式</span>
      <select v-model="execMode" class="exec-select">
        <option value="01">01 - 自產</option>
        <option value="02">02 - 委外</option>
      </select>
      <button class="btn btn-light" @click="openPicker">打開視窗挑選庫存</button>
      <button class="btn btn-danger" :disabled="!checkedIds.size" @click="handleCancel">
        取消耗用<span v-if="checkedIds.size" class="badge">{{ checkedIds.size }}</span>
      </button>
    </div>

    <div class="table-wrap">
      <table class="dtable">
        <thead>
          <tr>
            <th width="36">
              <input type="checkbox"
                :checked="rows.length > 0 && checkedIds.size === rows.length"
                :indeterminate="checkedIds.size > 0 && checkedIds.size < rows.length"
                @change="toggleAll($event.target.checked)" />
            </th>
            <th>執行方式</th>
            <th>品號</th><th>品名</th><th>倉庫</th><th>批號</th><th>耗用數量</th><th>單位</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in rows" :key="row.id"
              :class="{ 'row-checked': checkedIds.has(row.id) }"
              @click="toggle(row.id)">
            <td @click.stop>
              <input type="checkbox" :checked="checkedIds.has(row.id)" @change="toggle(row.id)" />
            </td>
            <td>
              <span :class="['exec-badge', row.execMode === '01' ? 'exec-self' : 'exec-out']">
                {{ row.execMode === '01' ? '01 - 自產' : '02 - 委外' }}
              </span>
            </td>
            <td>{{ row.itemNo }}</td>
            <td>{{ row.itemName }}</td>
            <td>{{ row.warehouse }}</td>
            <td>{{ row.lotNo }}</td>
            <td>{{ row.qty }}</td>
            <td>{{ row.unit }}</td>
          </tr>
          <tr v-if="!rows.length">
            <td colspan="8" class="empty-row">請先選擇執行方式，再點擊「打開視窗挑選庫存」</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const props = defineProps({
  rows: { type: Array, default: () => [] },
})
const emit = defineEmits(['open-picker', 'cancel-rows'])

const execMode   = ref('01')
const checkedIds = reactive(new Set())

function toggle(id) {
  if (checkedIds.has(id)) checkedIds.delete(id)
  else checkedIds.add(id)
}
function toggleAll(checked) {
  checkedIds.clear()
  if (checked) props.rows.forEach(r => checkedIds.add(r.id))
}
function openPicker() {
  emit('open-picker', { execMode: execMode.value })
}
function handleCancel() {
  if (!checkedIds.size) return
  if (!confirm(`確定要取消已勾選的 ${checkedIds.size} 筆耗用？`)) return
  emit('cancel-rows', { ids: [...checkedIds] })
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
.exec-label    { font-size: 12px; opacity: .85; white-space: nowrap; }

.exec-select {
  padding: 3px 8px; border: 1px solid rgba(255,255,255,.5); border-radius: 3px;
  font-size: 12px; font-family: inherit; background: #fff; color: #1a3a6e; cursor: pointer;
}

.table-wrap { padding: 8px 12px; }
.dtable { width: 100%; border-collapse: collapse; font-size: 12px; }
.dtable th { background: #3a6abf; color: #fff; padding: 5px 8px; border: 1px solid #2a5aaf; white-space: nowrap; text-align: center; }
.dtable td { padding: 4px 8px; border: 1px solid #d0dce8; text-align: center; white-space: nowrap; }
.dtable tr:nth-child(even) td { background: #f0f6ff; }
.dtable tr:hover td           { background: #dceeff; cursor: pointer; }
.row-checked td               { background: #ffe0e0 !important; }
.empty-row { color: #999; padding: 16px; text-align: center; }

.exec-badge   { display: inline-block; padding: 1px 8px; border-radius: 10px; font-size: 11px; font-weight: bold; white-space: nowrap; }
.exec-self    { background: #ddeeff; color: #1a4a9e; border: 1px solid #aaccee; }
.exec-out     { background: #ddf5ec; color: #1a6e5a; border: 1px solid #aaddcc; }

.btn { padding: 3px 10px; font-size: 12px; font-family: inherit; border: 1px solid; border-radius: 3px; cursor: pointer; display: inline-flex; align-items: center; gap: 4px; }
.btn:hover    { filter: brightness(1.1); }
.btn:disabled { opacity: .4; cursor: not-allowed; filter: none; }
.btn-light  { background: #fff; color: #2a5aaf; border-color: rgba(255,255,255,.7); }
.btn-light:hover { background: #e8f0ff; }
.btn-danger { background: #c0392b; color: #fff; border-color: #a93226; }
.badge { background: rgba(255,255,255,.9); color: #c0392b; font-size: 11px; font-weight: bold; padding: 0 5px; border-radius: 8px; min-width: 16px; text-align: center; }
</style>
