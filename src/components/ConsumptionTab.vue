<template>
  <div>
    <!-- 自產區塊 -->
    <template v-if="showSelf">
      <div class="section-bar">
        <span class="section-title">自產</span>
        <button class="btn btn-light" @click="$emit('open-picker', { section: 'self' })">打開視窗挑選庫存</button>
        <button class="btn btn-danger"
          :disabled="!selfChecked.size"
          @click="cancelChecked('self')">
          取消耗用
          <span v-if="selfChecked.size" class="badge">{{ selfChecked.size }}</span>
        </button>
      </div>
      <div class="table-wrap">
        <table class="dtable">
          <thead>
            <tr>
              <th width="36">
                <input type="checkbox"
                  :checked="selfRows.length > 0 && selfChecked.size === selfRows.length"
                  :indeterminate="selfChecked.size > 0 && selfChecked.size < selfRows.length"
                  @change="toggleAll('self', $event.target.checked)" />
              </th>
              <th>品號</th><th>品名</th><th>倉庫</th><th>批號</th><th>耗用數量</th><th>單位</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in selfRows" :key="row.id"
                :class="{ 'row-checked': selfChecked.has(row.id) }"
                @click="toggleOne('self', row.id)">
              <td @click.stop>
                <input type="checkbox" :checked="selfChecked.has(row.id)" @change="toggleOne('self', row.id)" />
              </td>
              <td>{{ row.itemNo }}</td><td>{{ row.itemName }}</td>
              <td>{{ row.warehouse }}</td><td>{{ row.lotNo }}</td>
              <td>{{ row.qty }}</td><td>{{ row.unit }}</td>
            </tr>
            <tr v-if="!selfRows.length">
              <td colspan="7" class="empty-row">尚無耗用資料，請點擊「打開視窗挑選庫存」</td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>

    <!-- 委外區塊 -->
    <template v-if="showOutsource">
      <div class="section-bar outsource">
        <span class="section-title">委外</span>
        <button class="btn btn-light" @click="$emit('open-picker', { section: 'outsource' })">打開視窗挑選庫存</button>
        <button class="btn btn-danger"
          :disabled="!outsourceChecked.size"
          @click="cancelChecked('outsource')">
          取消耗用
          <span v-if="outsourceChecked.size" class="badge">{{ outsourceChecked.size }}</span>
        </button>
      </div>
      <div class="table-wrap">
        <table class="dtable">
          <thead>
            <tr>
              <th width="36">
                <input type="checkbox"
                  :checked="outsourceRows.length > 0 && outsourceChecked.size === outsourceRows.length"
                  :indeterminate="outsourceChecked.size > 0 && outsourceChecked.size < outsourceRows.length"
                  @change="toggleAll('outsource', $event.target.checked)" />
              </th>
              <th>品號</th><th>品名</th><th>倉庫</th><th>批號</th><th>耗用數量</th><th>單位</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in outsourceRows" :key="row.id"
                :class="{ 'row-checked': outsourceChecked.has(row.id) }"
                @click="toggleOne('outsource', row.id)">
              <td @click.stop>
                <input type="checkbox" :checked="outsourceChecked.has(row.id)" @change="toggleOne('outsource', row.id)" />
              </td>
              <td>{{ row.itemNo }}</td><td>{{ row.itemName }}</td>
              <td>{{ row.warehouse }}</td><td>{{ row.lotNo }}</td>
              <td>{{ row.qty }}</td><td>{{ row.unit }}</td>
            </tr>
            <tr v-if="!outsourceRows.length">
              <td colspan="7" class="empty-row">尚無耗用資料，請點擊「打開視窗挑選庫存」</td>
            </tr>
          </tbody>
        </table>
      </div>
    </template>
  </div>
</template>

<script setup>
import { reactive } from 'vue'

const props = defineProps({
  showSelf:       { type: Boolean, default: true },
  showOutsource:  { type: Boolean, default: true },
  selfRows:       { type: Array, default: () => [] },
  outsourceRows:  { type: Array, default: () => [] },
})
const emit = defineEmits(['open-picker', 'cancel-rows'])

// 各區塊的勾選狀態（本地管理）
const selfChecked      = reactive(new Set())
const outsourceChecked = reactive(new Set())

function getSet(section)  { return section === 'self' ? selfChecked : outsourceChecked }
function getRows(section) { return section === 'self' ? props.selfRows : props.outsourceRows }

function toggleOne(section, id) {
  const set = getSet(section)
  if (set.has(id)) set.delete(id)
  else set.add(id)
}

function toggleAll(section, checked) {
  const set = getSet(section)
  set.clear()
  if (checked) getRows(section).forEach(r => set.add(r.id))
}

function cancelChecked(section) {
  const set = getSet(section)
  if (!set.size) return
  if (!confirm(`確定要取消已勾選的 ${set.size} 筆耗用？`)) return
  emit('cancel-rows', { section, ids: [...set] })
  set.clear()
}
</script>

<style scoped>
.section-bar {
  background: linear-gradient(90deg, #3a6abf, #5586d0);
  color: #fff; padding: 6px 14px;
  display: flex; align-items: center; gap: 8px;
  border-bottom: 1px solid #2a5aaf;
}
.section-bar.outsource { background: linear-gradient(90deg, #1a6e5a, #2a9e7a); border-color: #1a6e5a; }
.section-title { font-weight: bold; font-size: 13px; min-width: 40px; margin-right: 4px; }
.table-wrap { padding: 8px 12px; }

.dtable { width:100%; border-collapse:collapse; font-size:12px; }
.dtable th { background:#3a6abf; color:#fff; padding:5px 8px; border:1px solid #2a5aaf; white-space:nowrap; text-align:center; }
.dtable td { padding:4px 8px; border:1px solid #d0dce8; text-align:center; white-space:nowrap; }
.dtable tr:nth-child(even) td { background:#f0f6ff; }
.dtable tr:hover td           { background:#dceeff; cursor:pointer; }
.row-checked td               { background:#ffe0e0 !important; }
.empty-row { color:#999; padding:16px; text-align:center; }

.btn { padding:3px 10px; font-size:12px; font-family:inherit; border:1px solid; border-radius:3px; cursor:pointer; display:inline-flex; align-items:center; gap:4px; }
.btn:hover    { filter:brightness(1.1); }
.btn:disabled { opacity:.4; cursor:not-allowed; filter:none; }
.btn-light   { background:#fff; color:#2a5aaf; border-color:rgba(255,255,255,.7); }
.btn-light:hover { background:#e8f0ff; }
.btn-danger  { background:#c0392b; color:#fff; border-color:#a93226; }

.badge {
  background: rgba(255,255,255,0.9); color: #c0392b;
  font-size: 11px; font-weight: bold;
  padding: 0 5px; border-radius: 8px; min-width: 16px; text-align: center;
}
</style>
