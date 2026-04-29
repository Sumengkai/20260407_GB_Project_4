<template>
  <div class="erp-app">
    <!-- 頁首 -->
    <div class="page-header">
      <div class="header-title">☆ 塊材申請作業</div>
      <div class="header-info">
        <span>{{ currentDate }}</span>
        <span>系統管理員 (ICSCIG)</span>
      </div>
    </div>

    <!-- 查詢列 -->
    <div class="query-bar">
      <span class="query-label">單號查詢</span>
      <div class="query-input-wrap">
        <span class="query-icon">🔍</span>
        <input
          v-model="queryNo"
          type="text"
          class="query-input"
          placeholder="請輸入詢價單或申請單單號"
          @keyup.enter="doQuery"
        />
      </div>
      <button class="btn btn-query" @click="doQuery">查詢</button>
    </div>

    <!-- 訊息列 -->
    <div class="message-bar">
      <span class="msg-label">訊息</span>
      <span :class="['msg-text', message.type]">{{ message.text }}</span>
    </div>

    <!-- 頁籤導覽（依委託類型動態顯示） -->
    <div class="tab-nav">
      <button
        v-for="idx in visibleTabIndices"
        :key="idx"
        :class="['tab-btn', { active: activeTab === idx }]"
        @click="activeTab = idx"
      >{{ tabs[idx] }}</button>
    </div>

    <!-- 頁籤內容 -->
    <div class="tab-content">

      <!-- ===== 0. 詢價單 ===== -->
      <div v-show="activeTab === 0" class="tab-panel">
        <div class="panel-toolbar">
          <span class="toolbar-label">功能</span>
          <div class="toolbar-id" v-if="iq.id">詢價單號：{{ iq.id }}</div>
          <button v-if="!iq.id" class="btn btn-primary" @click="iqCreate">新增主檔</button>
          <template v-if="iq.id">
            <button class="btn btn-primary" @click="iqOpenLockPicker">打開視窗挑選庫存（鎖料）</button>
            <button
              class="btn"
              :class="d5.id ? 'btn-default' : 'btn-success'"
              @click="iqConfirmToD5"
            >{{ d5.id ? '已轉換：' + d5.id : '確認轉D5申請單' }}</button>
          </template>
        </div>

        <!-- 機加工版本選擇（右下角固定） -->
        <div class="version-box">
          <div class="version-title">機加工版本</div>
          <label class="version-chk">
            <input type="checkbox" v-model="enableMachiningV1" />
            <span>啟用 version1</span>
          </label>
          <label class="version-chk">
            <input type="checkbox" v-model="enableMachiningV2" />
            <span>啟用 version2</span>
          </label>
        </div>

        <div class="form-grid">
          <div class="form-row">
            <div class="form-cell lbl req-lbl">交期 *</div>
            <div class="form-cell inp">
              <input type="date" v-model="iq.deliveryDate" class="f-input" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">附件</div>
            <div class="form-cell inp c3">
              <input type="file" class="f-input" @change="iqFileChange" />
              <span v-if="iq.attachmentName" class="file-name">{{ iq.attachmentName }}</span>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">委託類型</div>
            <div class="form-cell inp c3">
              <label class="chk-label"><input type="checkbox" v-model="iq.typeMachining" /><span>機加工</span></label>
              <label class="chk-label"><input type="checkbox" v-model="iq.typeCoating" /><span>鍍膜</span></label>
              <label class="chk-label"><input type="checkbox" v-model="iq.typePurification" /><span>純化</span></label>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">爐次</div>
            <div class="form-cell inp">
              <input type="text" v-model="iq.furnaceNo" class="f-input" placeholder="請輸入爐次" />
            </div>
            <div class="form-cell lbl">費用</div>
            <div class="form-cell inp">
              <input type="number" v-model="iq.cost" class="f-input" placeholder="0" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客戶</div>
            <div class="form-cell inp c3">
              <select v-model="iq.customer" class="f-input">
                <option value="">— 請選擇客戶 —</option>
                <option v-for="c in CUSTOMERS" :key="c.code" :value="c.code">{{ c.name }}（{{ c.code }}）</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客戶圖號</div>
            <div class="form-cell inp">
              <input type="text" v-model="iq.customerDrawingNo" class="f-input" placeholder="請輸入客戶圖號" />
            </div>
            <div class="form-cell lbl">中碳圖號</div>
            <div class="form-cell inp">
              <input type="text" v-model="iq.icscDrawingNo" class="f-input" placeholder="請輸入中碳圖號" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">備註</div>
            <div class="form-cell inp c3">
              <textarea v-model="iq.remark" class="f-textarea" rows="2" placeholder="請輸入備註"></textarea>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客供料</div>
            <div class="form-cell inp c3">
              <textarea v-model="iq.clientMaterial" class="f-textarea" rows="2" placeholder="請輸入客供料說明"></textarea>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">鎖料批號</div>
            <div class="form-cell inp c3">
              <table class="dtable lock-table">
                <thead>
                  <tr><th>批號</th><th>品號</th><th>品名</th><th>數量</th><th>單位</th></tr>
                </thead>
                <tbody>
                  <tr v-for="(r, i) in iq.lockedLots" :key="i">
                    <td>{{ r.lotNo }}</td><td>{{ r.itemNo }}</td><td>{{ r.itemName }}</td>
                    <td>{{ r.qty }}</td><td>{{ r.unit }}</td>
                  </tr>
                  <tr v-if="!iq.lockedLots.length">
                    <td colspan="5" class="empty-row">— 尚無鎖料批號 —</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl sys-lbl">系統備註</div>
            <div class="form-cell inp c3">
              <div class="sys-log">
                <div v-if="!iq.sysLog.length" class="log-empty">— 尚無記錄 —</div>
                <div v-for="(entry, i) in iq.sysLog" :key="i" :class="['log-entry', entry.level]">
                  <span class="log-time">{{ entry.time }}</span>
                  <span class="log-dot">›</span>
                  <span class="log-msg">{{ entry.msg }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ===== 1. D5申請單（唯讀，資料連動詢價單） ===== -->
      <div v-show="activeTab === 1" class="tab-panel">
        <div class="panel-toolbar">
          <span class="toolbar-label">D5申請單</span>
          <div class="toolbar-id" v-if="d5.id">單號：{{ d5.id }}</div>
          <div v-else class="toolbar-hint">請先在詢價單按下「確認轉D5申請單」</div>
          <button v-if="d5.id" class="btn btn-primary" @click="d5Save">儲存</button>
        </div>

        <div class="form-grid">
          <div class="form-row">
            <div class="form-cell lbl">受理與否</div>
            <div class="form-cell inp c3">
              <label class="chk-label"><input type="radio" v-model="d5.accepted" value="Y" /><span>是</span></label>
              <label class="chk-label"><input type="radio" v-model="d5.accepted" value="N" /><span>否</span></label>
              <span v-if="!d5.accepted" style="color:#888;font-size:12px;margin-left:8px">— 尚未設定 —</span>
              <span v-if="d5.accepted === 'N'" style="color:#c0392b;font-size:12px;margin-left:8px;font-weight:bold">不受理：後續製程頁籤將會隱藏</span>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl req-lbl">交期 *</div>
            <div class="form-cell inp">
              <input type="date" :value="iq.deliveryDate" class="f-input" readonly />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">附件</div>
            <div class="form-cell inp c3">
              <input type="file" class="f-input" @change="d5FileChange" />
              <span v-if="d5.attachmentName" class="file-name">{{ d5.attachmentName }}</span>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">委託類型</div>
            <div class="form-cell inp c3">
              <label class="chk-label"><input type="checkbox" :checked="iq.typeMachining" disabled /><span>機加工</span></label>
              <label class="chk-label"><input type="checkbox" :checked="iq.typeCoating" disabled /><span>鍍膜</span></label>
              <label class="chk-label"><input type="checkbox" :checked="iq.typePurification" disabled /><span>純化</span></label>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">爐次</div>
            <div class="form-cell inp">
              <input type="text" :value="iq.furnaceNo" class="f-input" readonly placeholder="—" />
            </div>
            <div class="form-cell lbl">費用</div>
            <div class="form-cell inp">
              <input type="number" :value="iq.cost" class="f-input" readonly />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客戶</div>
            <div class="form-cell inp c3">
              <input type="text"
                :value="iq.customer ? (CUSTOMERS.find(c => c.code === iq.customer)?.name + '（' + iq.customer + '）') : '—'"
                class="f-input" readonly />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客戶圖號</div>
            <div class="form-cell inp">
              <input type="text" :value="iq.customerDrawingNo" class="f-input" readonly placeholder="—" />
            </div>
            <div class="form-cell lbl">中碳圖號</div>
            <div class="form-cell inp">
              <input type="text" :value="iq.icscDrawingNo" class="f-input" readonly placeholder="—" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl sys-lbl">詢價單備註</div>
            <div class="form-cell inp c3">
              <textarea :value="iq.remark" class="f-textarea" rows="2" readonly></textarea>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">備註</div>
            <div class="form-cell inp c3">
              <textarea v-model="d5.remark" class="f-textarea" rows="2" placeholder="請輸入D5申請單備註（與詢價單備註各自管理）"></textarea>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客供料</div>
            <div class="form-cell inp c3">
              <textarea :value="iq.clientMaterial" class="f-textarea" rows="2" readonly></textarea>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">鎖料批號</div>
            <div class="form-cell inp c3">
              <table class="dtable lock-table">
                <thead>
                  <tr><th>批號</th><th>品號</th><th>品名</th><th>數量</th><th>單位</th></tr>
                </thead>
                <tbody>
                  <tr v-for="(r, i) in iq.lockedLots" :key="i">
                    <td>{{ r.lotNo }}</td><td>{{ r.itemNo }}</td><td>{{ r.itemName }}</td>
                    <td>{{ r.qty }}</td><td>{{ r.unit }}</td>
                  </tr>
                  <tr v-if="!iq.lockedLots.length">
                    <td colspan="5" class="empty-row">— 尚無鎖料批號 —</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl sys-lbl">產出批號總覽</div>
            <div class="form-cell inp c3">
              <div class="lot-overview">
                <div v-if="!lotOverview.length" class="lot-ov-empty">— 尚無產出批號 —</div>
                <div v-for="item in lotOverview" :key="item.proc" class="lot-ov-row">
                  <span class="lot-ov-proc">{{ item.proc }}產出批號</span>
                  <span class="lot-ov-lots">{{ item.lots }}</span>
                </div>
              </div>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl sys-lbl">工單清單</div>
            <div class="form-cell inp c3">
              <div class="wo-list">
                <div v-if="!workOrderList.length" class="wo-empty">— 尚無工單（各製程有耗用/產出明細時自動生成）—</div>
                <table v-else class="dtable">
                  <thead>
                    <tr>
                      <th>工單號</th><th>製程</th><th>執行方式</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="wo in workOrderList" :key="wo.key">
                      <td class="wo-no">{{ wo.no }}</td>
                      <td>{{ wo.proc }}</td>
                      <td><span :class="['wo-badge', wo.execType === '自產' ? 'badge-self' : 'badge-out']">{{ wo.execType }}</span></td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl sys-lbl">系統備註</div>
            <div class="form-cell inp c3">
              <div class="sys-log">
                <div v-if="!iq.sysLog.length" class="log-empty">— 尚無記錄 —</div>
                <div v-for="(entry, i) in iq.sysLog" :key="i" :class="['log-entry', entry.level]">
                  <span class="log-time">{{ entry.time }}</span>
                  <span class="log-dot">›</span>
                  <span class="log-msg">{{ entry.msg }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ===== 2. 機加工(耗用) ===== -->
      <div v-show="activeTab === 2" class="tab-panel">
        <ConsumptionTab
          :show-self="true" :show-outsource="true"
          :self-rows="machining.consumption.self"
          :outsource-rows="machining.consumption.outsource"
          @open-picker="openPicker($event, 'machining')"
          @cancel-rows="cancelConsumption('machining', $event)"
        />
      </div>

      <!-- ===== 3. 機加工(產出) ===== -->
      <div v-show="activeTab === 3" class="tab-panel">
        <OutputTab
          :show-self="true" :show-outsource="true"
          :self-rows="machining.output.self"
          :outsource-rows="machining.output.outsource"
          @add="handleOutputAdd('machining', $event)"
          @update="handleOutputUpdate('machining', $event)"
          @delete-items="handleOutputDelete('machining', $event)"
        />
      </div>

      <!-- ===== 4. 機加工(耗用)_version2 ===== -->
      <div v-show="activeTab === 4" class="tab-panel">
        <ConsumptionTabV2
          :rows="machining2.consumption"
          @open-picker="openPickerV2($event)"
          @cancel-rows="handleM2ConsumptionCancel($event)"
        />
      </div>

      <!-- ===== 5. 機加工(產出)_version2 ===== -->
      <div v-show="activeTab === 5" class="tab-panel">
        <OutputTabV2
          :rows="machining2.output"
          @add="handleM2OutputAdd($event)"
          @update="handleM2OutputUpdate($event)"
          @delete-items="handleM2OutputDelete($event)"
        />
      </div>

      <!-- ===== 6. 鍍膜(耗用) ===== -->
      <div v-show="activeTab === 6" class="tab-panel">
        <ConsumptionTab
          :show-self="false" :show-outsource="true"
          :self-rows="coating.consumption.self"
          :outsource-rows="coating.consumption.outsource"
          @open-picker="openPicker($event, 'coating')"
          @cancel-rows="cancelConsumption('coating', $event)"
        />
      </div>

      <!-- ===== 7. 鍍膜(產出) ===== -->
      <div v-show="activeTab === 7" class="tab-panel">
        <OutputTab
          :show-self="false" :show-outsource="true"
          :self-rows="coating.output.self"
          :outsource-rows="coating.output.outsource"
          @add="handleOutputAdd('coating', $event)"
          @update="handleOutputUpdate('coating', $event)"
          @delete-items="handleOutputDelete('coating', $event)"
        />
      </div>

      <!-- ===== 8. 純化(耗用) ===== -->
      <div v-show="activeTab === 8" class="tab-panel">
        <ConsumptionTab
          :show-self="true" :show-outsource="false"
          :self-rows="purification.consumption.self"
          :outsource-rows="purification.consumption.outsource"
          @open-picker="openPicker($event, 'purification')"
          @cancel-rows="cancelConsumption('purification', $event)"
        />
      </div>

      <!-- ===== 9. 純化(產出) ===== -->
      <div v-show="activeTab === 9" class="tab-panel">
        <OutputTab
          :show-self="true" :show-outsource="false"
          :self-rows="purification.output.self"
          :outsource-rows="purification.output.outsource"
          @add="handleOutputAdd('purification', $event)"
          @update="handleOutputUpdate('purification', $event)"
          @delete-items="handleOutputDelete('purification', $event)"
        />
      </div>

      <!-- ===== 10. 品檢資訊 ===== -->
      <div v-show="activeTab === 10" class="tab-panel">
        <QualityTab
          :all-lots="allLots"
          :quality-data="qualityData"
          @change="handleQualityChange($event)"
        />
      </div>

    </div>

    <!-- ===== 庫存挑選 Modal（耗用 / 鎖料） ===== -->
    <div v-if="picker.show" class="modal-overlay" @click.self="picker.show = false">
      <div class="modal">
        <div class="modal-hdr">
          <span>{{ picker.mode === 'lock' ? '挑選庫存（鎖料）' : '挑選庫存' }}</span>
          <span v-if="picker.mode !== 'lock'" class="picker-src">
            {{ picker.section === 'self' ? '來源：BB 碳材料生產工廠' : '來源：不限庫別' }}
          </span>
          <span class="picker-count" v-if="picker.selIds.size">已選 {{ picker.selIds.size }} 筆</span>
          <button class="modal-x" @click="picker.show = false">✕</button>
        </div>
        <div class="modal-body">
          <div style="margin-bottom:8px">
            <input v-model="picker.keyword" type="text" class="f-input" placeholder="搜尋品號 / 品名..." @input="filterInv" />
          </div>
          <table class="dtable">
            <thead>
              <tr>
                <th width="36">
                  <input type="checkbox"
                    :checked="filteredInv.length > 0 && picker.selIds.size === filteredInv.length"
                    :indeterminate="picker.selIds.size > 0 && picker.selIds.size < filteredInv.length"
                    @change="pickerToggleAll($event.target.checked)" />
                </th>
                <th>品號</th><th>品名</th><th>倉庫</th><th>批號</th><th>可用數量</th><th>單位</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(inv, i) in filteredInv" :key="i"
                  :class="{ 'row-sel': picker.selIds.has(i) }"
                  @click="pickerToggleOne(i)">
                <td @click.stop>
                  <input type="checkbox" :checked="picker.selIds.has(i)" @change="pickerToggleOne(i)" />
                </td>
                <td>{{ inv.itemNo }}</td><td>{{ inv.itemName }}</td>
                <td>{{ WAREHOUSE_LABEL[inv.warehouse] ?? inv.warehouse }}</td><td>{{ inv.lotNo }}</td>
                <td>{{ inv.qty }}</td><td>{{ inv.unit }}</td>
              </tr>
              <tr v-if="!filteredInv.length">
                <td colspan="7" class="empty-row">查無庫存資料</td>
              </tr>
            </tbody>
          </table>
        </div>
        <div class="modal-ftr">
          <button class="btn btn-success" @click="confirmPicker">
            {{ picker.mode === 'lock' ? '確認鎖料' : '確認耗用' }}（{{ picker.selIds.size }} 筆）
          </button>
          <button class="btn btn-default" @click="picker.show = false">取消</button>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, reactive, computed, watch } from 'vue'
import ConsumptionTab   from './components/ConsumptionTab.vue'
import ConsumptionTabV2 from './components/ConsumptionTabV2.vue'
import OutputTab        from './components/OutputTab.vue'
import OutputTabV2      from './components/OutputTabV2.vue'
import QualityTab       from './components/QualityTab.vue'

// 日期
const now = new Date()
const days = ['日','一','二','三','四','五','六']
const currentDate = `${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} (${days[now.getDay()]})`

// 頁籤（固定索引：0=詢價單, 1=D5申請單, 2-10=製程/品檢）
const tabs = ['詢價單','D5申請單','機加工(耗用)','機加工(產出)','機加工(耗用)_version2','機加工(產出)_version2','鍍膜(耗用)','鍍膜(產出)','純化(耗用)','純化(產出)','品檢資訊']
const activeTab         = ref(0)
const enableMachiningV1 = ref(true)
const enableMachiningV2 = ref(false)

// 訊息
const message = reactive({ text: '歡迎使用本系統...', type: 'info' })
const showMsg = (text, type='info') => { message.text = text; message.type = type }

// 查詢
const queryNo = ref('')
function doQuery() {
  const q = queryNo.value.trim()
  if (!q) { showMsg('請輸入詢價單或申請單單號', 'error'); return }
  if (iq.id && iq.id.toUpperCase() === q.toUpperCase()) {
    activeTab.value = 0
    showMsg(`查詢成功：已載入詢價單 ${iq.id}`, 'success')
  } else if (d5.id && d5.id.toUpperCase() === q.toUpperCase()) {
    activeTab.value = 1
    showMsg(`查詢成功：已載入 D5 申請單 ${d5.id}`, 'success')
  } else {
    showMsg(`查無單號：${q}`, 'error')
  }
}

// 預設交期（當月+1）
const defaultDeliveryDate = (() => {
  const d = new Date()
  d.setMonth(d.getMonth() + 1)
  return `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`
})()

// 詢價單資料（主要資料來源）
const iq = reactive({
  id: null,
  deliveryDate: defaultDeliveryDate,
  attachmentName: '',
  typeMachining: false, typeCoating: false, typePurification: false,
  furnaceNo: '', cost: null, customer: '', remark: '', clientMaterial: '',
  customerDrawingNo: '', icscDrawingNo: '',
  sysLog: [],
  lockedLots: []
})

// D5申請單（單號 + D5專屬欄位，備註/附件與詢價單分開管理）
// saved: 按下儲存後才為 true，後續製程頁籤才會顯示
const d5 = reactive({ id: null, accepted: '', remark: '', attachmentName: '', saved: false })

// 客戶下拉
const CUSTOMERS = [
  { name:'正達', code:'GT' },
  { name:'永泉', code:'YQ' },
  { name:'盛新', code:'TS' },
  { name:'漢民', code:'HM' },
  { name:'環球晶', code:'GW' },
  { name:'合晶',  code:'WW' },
  { name:'臺譜',  code:'TP' },
]

const PROC_NAME = { machining:'機加工', machining2:'機加工', coating:'鍍膜', purification:'純化' }
const SEC_NAME  = { self:'自產', outsource:'委外' }

// 系統 LOG（寫入詢價單 sysLog，D5 tab 亦顯示同一份）
function addLog(msg, level = 'info', key = null) {
  if (key && iq.sysLog.some(e => e.key === key)) return
  const t = new Date()
  const time = `${t.getFullYear()}/${String(t.getMonth()+1).padStart(2,'0')}/${String(t.getDate()).padStart(2,'0')} ${String(t.getHours()).padStart(2,'0')}:${String(t.getMinutes()).padStart(2,'0')}:${String(t.getSeconds()).padStart(2,'0')}`
  iq.sysLog.push({ time, msg, level, key })
}
function removeLog(key) {
  const idx = iq.sysLog.findIndex(e => e.key === key)
  if (idx >= 0) iq.sysLog.splice(idx, 1)
}
function procCount(proc, type) {
  const p = getProc(proc)
  return p[type].self.length + p[type].outsource.length
}

// 依委託類型動態顯示頁籤
// 條件：D5 已儲存（d5.saved）且受理與否為「是」，後續製程頁籤才根據委託類型顯示
const visibleTabIndices = computed(() => {
  const idx = [0, 1]
  if (!d5.saved || d5.accepted !== 'Y') return idx
  if (iq.typeMachining) {
    if (enableMachiningV1.value) idx.push(2, 3)
    if (enableMachiningV2.value) idx.push(4, 5)
  }
  if (iq.typeCoating)      idx.push(6, 7)
  if (iq.typePurification) idx.push(8, 9)
  if (iq.typeMachining || iq.typeCoating || iq.typePurification) idx.push(10)
  return idx
})
watch(visibleTabIndices, (newList) => {
  if (!newList.includes(activeTab.value)) activeTab.value = 0
})
// 受理與否變動時重設儲存狀態，需重新儲存才能反映新條件
watch(() => d5.accepted, () => { d5.saved = false })

// ===== 詢價單操作 =====
function iqCreate() {
  if (!iq.deliveryDate) { showMsg('交期為必填欄位', 'error'); return }
  const ym = `${new Date().getFullYear()}${String(new Date().getMonth()+1).padStart(2,'0')}`
  iq.id = `IQ-${ym}-${String(Math.floor(Math.random()*9000)+1000)}`
  addLog(`建立詢價單，單號：${iq.id}`, 'create', 'iq-create')
  showMsg(`詢價單建檔完成，單號：${iq.id}`, 'success')
}

function iqFileChange(e) { iq.attachmentName = e.target.files[0]?.name || '' }

function iqOpenLockPicker() {
  if (!iq.id) { showMsg('請先建立詢價單主檔', 'error'); return }
  picker.mode = 'lock'
  picker.proc = ''; picker.section = ''; picker.execMode = ''
  picker.keyword = ''; picker.selIds.clear()
  filteredInv.value = [...allInv]
  picker.show = true
}

function d5FileChange(e) { d5.attachmentName = e.target.files[0]?.name || '' }

function d5Save() {
  if (!d5.id) { showMsg('請先建立 D5 申請單', 'error'); return }
  if (!d5.accepted) { showMsg('請先設定「受理與否」再儲存', 'error'); return }
  d5.saved = true
  const acceptedLabel = d5.accepted === 'Y' ? '是' : '否'
  addLog(`儲存 D5 申請單：${d5.id}（受理：${acceptedLabel}）`, 'edit', null)
  showMsg(`D5 申請單已儲存${d5.accepted === 'Y' ? '，後續製程頁籤已開放' : '，不受理'}`, 'success')
}

function iqConfirmToD5() {
  if (!iq.id) { showMsg('請先建立詢價單主檔', 'error'); return }
  if (!iq.deliveryDate) { showMsg('交期為必填欄位', 'error'); return }
  if (d5.id) {
    showMsg(`已轉換為 D5 申請單，單號：${d5.id}`, 'info')
    activeTab.value = 1
    return
  }
  const ym = `${new Date().getFullYear()}${String(new Date().getMonth()+1).padStart(2,'0')}`
  d5.id = `D5-${ym}-${String(Math.floor(Math.random()*9000)+1000)}`
  addLog(`由詢價單 ${iq.id} 轉換，建立 D5 申請單：${d5.id}`, 'create', 'd5-create')
  showMsg(`已轉換為 D5 申請單，單號：${d5.id}`, 'success')
  activeTab.value = 1
}

// ===== 各製程資料 =====
function mkProc() { return { consumption:{self:[], outsource:[]}, output:{self:[], outsource:[]} } }
const machining    = reactive(mkProc())
const machining2   = reactive({ consumption: [], output: [] })
const coating      = reactive(mkProc())
const purification = reactive(mkProc())

const getProc = n => ({machining, coating, purification}[n])

// ===== 工單管理 =====
const workOrders = reactive({})

function genWorkOrderNo() {
  const ym = `${new Date().getFullYear()}${String(new Date().getMonth()+1).padStart(2,'0')}`
  return `WO-${ym}-${String(Math.floor(Math.random()*9000)+1000)}`
}
function ensureWorkOrder(key, proc, execType) {
  if (!workOrders[key]) {
    workOrders[key] = { no: genWorkOrderNo(), proc, execType }
    addLog(`生成【${proc}(${execType})】工單：${workOrders[key].no}`, 'create', `wo-${key}`)
  }
}
function removeWorkOrder(key) {
  if (workOrders[key]) {
    removeLog(`wo-${key}`)
    delete workOrders[key]
  }
}

const hasMachSelf = computed(() =>
  machining.consumption.self.length > 0 || machining.output.self.length > 0 ||
  machining2.consumption.some(r => r.execMode === '01') ||
  machining2.output.some(r => r.execMode === '01')
)
const hasMachOut = computed(() =>
  machining.consumption.outsource.length > 0 || machining.output.outsource.length > 0 ||
  machining2.consumption.some(r => r.execMode === '02') ||
  machining2.output.some(r => r.execMode === '02')
)
const hasCoatSelf = computed(() =>
  coating.consumption.self.length > 0 || coating.output.self.length > 0
)
const hasCoatOut = computed(() =>
  coating.consumption.outsource.length > 0 || coating.output.outsource.length > 0
)
const hasPureSelf = computed(() =>
  purification.consumption.self.length > 0 || purification.output.self.length > 0
)
const hasPureOut = computed(() =>
  purification.consumption.outsource.length > 0 || purification.output.outsource.length > 0
)

watch(hasMachSelf, v => v ? ensureWorkOrder('machining-self', '機加工', '自產') : removeWorkOrder('machining-self'))
watch(hasMachOut,  v => v ? ensureWorkOrder('machining-out',  '機加工', '委外') : removeWorkOrder('machining-out'))
watch(hasCoatSelf, v => v ? ensureWorkOrder('coating-self',   '鍍膜',   '自產') : removeWorkOrder('coating-self'))
watch(hasCoatOut,  v => v ? ensureWorkOrder('coating-out',    '鍍膜',   '委外') : removeWorkOrder('coating-out'))
watch(hasPureSelf, v => v ? ensureWorkOrder('purification-self', '純化', '自產') : removeWorkOrder('purification-self'))
watch(hasPureOut,  v => v ? ensureWorkOrder('purification-out',  '純化', '委外') : removeWorkOrder('purification-out'))

const workOrderList = computed(() => Object.entries(workOrders).map(([key, wo]) => ({ key, ...wo })))

// ===== 庫存資料 =====
const WAREHOUSE_LABEL = { BB:'碳材料生產工廠', BBPN:'D8屏南儲區', BBOUTTMP:'委外暫存庫' }

const allInv = [
  { itemNo:'ACS15U', itemName:'超級電容ACS15U',  warehouse:'BB',       lotNo:'GB-001', qty:500, unit:'kg'  },
  { itemNo:'GC-001', itemName:'石墨化碳材料',     warehouse:'BB',       lotNo:'GB-002', qty:200, unit:'kg'  },
  { itemNo:'CB-050', itemName:'碳微球CB-050',     warehouse:'BB',       lotNo:'GB-003', qty:150, unit:'kg'  },
  { itemNo:'PG-100', itemName:'純化石墨PG-100',   warehouse:'BBPN',     lotNo:'GB-004', qty:80,  unit:'kg'  },
  { itemNo:'MC-200', itemName:'機加工碳塊MC-200', warehouse:'BBPN',     lotNo:'GB-005', qty:320, unit:'pcs' },
  { itemNo:'CF-300', itemName:'碳纖維CF-300',     warehouse:'BBOUTTMP', lotNo:'GB-006', qty:100, unit:'kg'  },
  { itemNo:'GR-400', itemName:'石墨電極GR-400',   warehouse:'BBOUTTMP', lotNo:'GB-007', qty:60,  unit:'pcs' },
]

function getBaseInv() {
  if (picker.mode === 'lock') return [...allInv]
  if (picker.section === 'self') return allInv.filter(i => i.warehouse === 'BB')
  return allInv.filter(i => i.warehouse !== 'BBPN')
}

const filteredInv = ref([...allInv])
const picker = reactive({ show:false, keyword:'', selIds: new Set(), proc:'', section:'', execMode:'', mode:'' })

function filterInv() {
  const kw = picker.keyword.toLowerCase()
  filteredInv.value = getBaseInv().filter(i =>
    i.itemNo.toLowerCase().includes(kw) || i.itemName.toLowerCase().includes(kw)
  )
  picker.selIds.clear()
}
function pickerToggleOne(i) {
  if (picker.selIds.has(i)) picker.selIds.delete(i)
  else picker.selIds.add(i)
}
function pickerToggleAll(checked) {
  picker.selIds.clear()
  if (checked) filteredInv.value.forEach((_, i) => picker.selIds.add(i))
}
function openPicker({ section }, procName) {
  picker.mode = 'consumption'
  picker.proc = procName; picker.section = section; picker.execMode = ''
  picker.keyword = ''; picker.selIds.clear()
  filteredInv.value = getBaseInv()
  picker.show = true
}
function openPickerV2({ execMode }) {
  picker.mode = 'consumption'
  picker.proc = 'machining2'
  picker.execMode = execMode
  picker.section  = execMode === '01' ? 'self' : 'outsource'
  picker.keyword  = ''; picker.selIds.clear()
  filteredInv.value = getBaseInv()
  picker.show = true
}

function confirmPicker() {
  if (!picker.selIds.size) { showMsg('請先勾選庫存項目', 'error'); return }

  // 鎖料模式：加入詢價單鎖料批號
  if (picker.mode === 'lock') {
    const addedNames = []
    picker.selIds.forEach(i => {
      const inv = filteredInv.value[i]
      if (!iq.lockedLots.some(l => l.lotNo === inv.lotNo)) {
        iq.lockedLots.push({ lotNo: inv.lotNo, itemNo: inv.itemNo, itemName: inv.itemName, qty: inv.qty, unit: inv.unit })
        addedNames.push(inv.itemName)
      }
    })
    if (addedNames.length > 0) {
      addLog(`鎖料 ${addedNames.length} 筆批號：${addedNames.join('、')}`, 'input')
      showMsg(`已鎖定 ${addedNames.length} 筆庫存批號`, 'success')
    } else {
      showMsg('所選批號已全部鎖定，無新增', 'info')
    }
    picker.show = false
    picker.selIds.clear()
    return
  }

  // 耗用模式
  const names = []
  if (picker.proc === 'machining2') {
    picker.selIds.forEach(i => {
      const inv = filteredInv.value[i]
      machining2.consumption.push({ id: Date.now() + i, ...inv, execMode: picker.execMode })
      names.push(inv.itemName)
    })
  } else {
    const arr = getProc(picker.proc).consumption[picker.section]
    picker.selIds.forEach(i => {
      const inv = filteredInv.value[i]
      arr.push({ id: Date.now() + i, ...inv })
      names.push(inv.itemName)
    })
  }
  const count = picker.selIds.size
  picker.show = false
  addLog(`輸入【${PROC_NAME[picker.proc]}(耗用)】`, 'input', `consumption-${picker.proc}`)
  showMsg(`已確認耗用 ${count} 筆：${names.join('、')}`, 'success')
  picker.selIds.clear()
}

function cancelConsumption(procName, { section, ids }) {
  const idSet = new Set(ids)
  const arr = getProc(procName).consumption[section]
  arr.splice(0, arr.length, ...arr.filter(r => !idSet.has(r.id)))
  if (procCount(procName, 'consumption') === 0) removeLog(`consumption-${procName}`)
  showMsg(`已取消 ${idSet.size} 筆耗用`, 'info')
}
function handleM2ConsumptionCancel({ ids }) {
  const idSet = new Set(ids)
  machining2.consumption.splice(0, machining2.consumption.length,
    ...machining2.consumption.filter(r => !idSet.has(r.id)))
  if (!machining2.consumption.length) removeLog('consumption-machining2')
  showMsg(`已取消 ${idSet.size} 筆耗用`, 'info')
}

function handleM2OutputAdd(row) {
  machining2.output.push({ ...row, id: Date.now() })
  addLog('輸入【機加工(產出)】', 'input', 'output-machining2')
  showMsg('產出明細已新增', 'success')
}
function handleM2OutputUpdate({ idx, row }) {
  machining2.output[idx] = { ...row, id: machining2.output[idx].id }
  showMsg('產出明細已修改', 'success')
}
function handleM2OutputDelete({ ids }) {
  const idSet = new Set(ids)
  machining2.output.splice(0, machining2.output.length,
    ...machining2.output.filter(r => !idSet.has(r.id)))
  if (!machining2.output.length) removeLog('output-machining2')
  showMsg(`已刪除 ${idSet.size} 筆產出明細`, 'info')
}

function handleOutputAdd(proc, { section, row }) {
  getProc(proc).output[section].push({ ...row, id: Date.now() })
  addLog(`輸入【${PROC_NAME[proc]}(產出)】`, 'input', `output-${proc}`)
  showMsg('產出明細已新增', 'success')
}
function handleOutputUpdate(proc, { section, idx, row }) {
  const arr = getProc(proc).output[section]
  arr[idx] = { ...row, id: arr[idx].id }
  showMsg('產出明細已修改', 'success')
}
function handleOutputDelete(proc, { section, ids }) {
  const idSet = new Set(ids)
  const arr = getProc(proc).output[section]
  arr.splice(0, arr.length, ...arr.filter(r => !idSet.has(r.id)))
  if (procCount(proc, 'output') === 0) removeLog(`output-${proc}`)
  showMsg(`已刪除 ${idSet.size} 筆產出明細`, 'info')
}

// 品檢資訊
const qualityData = reactive({})

const lotOverview = computed(() => {
  const procMap = new Map()
  const addLots = (proc, rows) => rows.forEach(r => {
    if (!r.lotNo) return
    if (!procMap.has(proc)) procMap.set(proc, new Set())
    procMap.get(proc).add(r.lotNo)
  })
  addLots('機加工', [...machining.output.self, ...machining.output.outsource])
  addLots('機加工', machining2.output)
  addLots('鍍膜',   [...coating.output.self,   ...coating.output.outsource])
  addLots('純化',   [...purification.output.self, ...purification.output.outsource])
  return [...procMap.entries()].map(([proc, lots]) => ({ proc, lots: [...lots].join('、') }))
})

const allLots = computed(() => {
  const lots = [], seen = new Set()
  const collect = (proc, procName) => {
    ['self','outsource'].forEach(sec => {
      getProc(proc).output[sec].forEach(row => {
        if (row.lotNo && !seen.has(row.lotNo)) {
          seen.add(row.lotNo)
          lots.push({ lotNo: row.lotNo, productKey: row.productKey, procName, section: SEC_NAME[sec] })
        }
      })
    })
  }
  collect('machining', '機加工')
  machining2.output.forEach(row => {
    if (row.lotNo && !seen.has(row.lotNo)) {
      seen.add(row.lotNo)
      lots.push({ lotNo: row.lotNo, productKey: row.productKey, procName: '機加工', section: row.execMode === '01' ? '自產' : '委外' })
    }
  })
  collect('coating',    '鍍膜')
  collect('purification', '純化')
  return lots
})

function handleQualityChange({ lotNo, paramKey, value }) {
  if (!qualityData[lotNo]) qualityData[lotNo] = {}
  qualityData[lotNo][paramKey] = value
  addLog(`輸入【品檢資訊】批號 ${lotNo}`, 'input', `quality-${lotNo}`)
}
</script>

<style scoped>
.erp-app { min-height:100vh; display:flex; flex-direction:column; background:#fff; font-size:13px; }

/* 頁首 */
.page-header {
  background: linear-gradient(135deg, #1a3a6e 0%, #2a5aab 100%);
  color:#fff; padding:8px 16px;
  display:flex; align-items:center; justify-content:space-between;
  box-shadow:0 2px 6px rgba(0,0,0,.4);
}
.header-title { font-size:16px; font-weight:bold; letter-spacing:1px; }
.header-info  { font-size:12px; display:flex; gap:16px; opacity:.9; }

/* 查詢列 */
.query-bar {
  background: #c8d8ec;
  border-bottom: 1px solid #a0b8d8;
  padding: 5px 16px;
  display: flex;
  align-items: center;
  gap: 8px;
}
.query-label {
  background: #3a6abf;
  color: #fff;
  padding: 2px 10px;
  border-radius: 3px;
  font-size: 12px;
  white-space: nowrap;
}
.query-input-wrap {
  display: flex;
  align-items: center;
  border: 1px solid #8aaad0;
  border-radius: 3px;
  background: #fff;
  overflow: hidden;
  width: 240px;
}
.query-icon { padding: 0 6px; font-size: 12px; color: #7a9ac8; }
.query-input {
  flex: 1;
  border: none;
  outline: none;
  padding: 3px 4px;
  font-size: 13px;
  font-family: inherit;
  background: transparent;
}
.btn-query {
  background: #3a6abf;
  color: #fff;
  border: 1px solid #2a5aaf;
  padding: 3px 14px;
  font-size: 12px;
  font-family: inherit;
  border-radius: 3px;
  cursor: pointer;
}
.btn-query:hover { filter: brightness(1.1); }

/* 訊息列 */
.message-bar {
  background:#c8d8ec; border-bottom:1px solid #a0b8d8;
  padding:4px 16px; display:flex; align-items:center; gap:8px; min-height:28px;
}
.msg-label { background:#3a6abf; color:#fff; padding:1px 8px; border-radius:3px; font-size:12px; white-space:nowrap; }
.msg-text { font-size:13px; color:#c0392b; font-weight:bold; }

/* 頁籤導覽 */
.tab-nav {
  display:flex; flex-wrap:wrap; background:#b0c8e8;
  border-bottom:2px solid #3a6abf; padding:6px 8px 0; gap:3px;
}
.tab-btn {
  padding:5px 14px; font-size:13px; font-family:inherit;
  border:1px solid #7a9ac8; border-bottom:none; border-radius:4px 4px 0 0;
  background:#d0e0f0; color:#000; cursor:pointer; transition:background .15s;
}
.tab-btn:hover  { background:#e0ecf8; color:#000; }
.tab-btn.active { background:#fff; font-weight:bold; border-color:#3a6abf; color:#000; }

/* 頁籤內容 */
.tab-content { flex:1; padding:12px; }
.tab-panel   { background:#fff; border:1px solid #b0c4de; border-radius:0 4px 4px 4px; }

.version-box {
  position: fixed; bottom: 12px; right: 12px; z-index: 500;
  border: 1px solid #b0c4de; border-radius: 6px;
  background: #f4f8ff; padding: 10px 14px;
  display: flex; flex-direction: column; gap: 6px;
  box-shadow: 0 2px 6px rgba(0,0,0,.08);
}
.version-title {
  font-size: 11px; font-weight: bold; color: #3a6abf;
  letter-spacing: .5px; margin-bottom: 2px;
}
.version-chk { display: flex; align-items: center; gap: 6px; font-size: 12px; cursor: pointer; }
.version-chk input { cursor: pointer; }

/* 執行方式選擇列 */
.mode-bar {
  background: #eef4ff; border-bottom: 1px solid #b8cfe8;
  padding: 7px 14px; display: flex; align-items: center; gap: 10px;
}
.mode-label {
  background: #5a7ec8; color: #fff;
  padding: 2px 10px; border-radius: 3px; font-size: 12px; white-space: nowrap;
}
.mode-select {
  padding: 3px 10px; border: 1px solid #b0c4de; border-radius: 3px;
  font-size: 13px; font-family: inherit; background: #fff; color: #1a3a6e;
  cursor: pointer;
}

/* 工具列 */
.panel-toolbar {
  background:#d8e8f8; border-bottom:1px solid #b0c4de;
  padding:6px 12px; display:flex; align-items:center; gap:6px; flex-wrap:wrap;
}
.toolbar-label { background:#3a6abf; color:#fff; padding:2px 10px; border-radius:3px; font-size:12px; margin-right:4px; }
.toolbar-hint  { font-size:12px; color:#888; font-style:italic; padding:2px 8px; }

/* 按鈕 */
.btn { padding:3px 12px; font-size:12px; font-family:inherit; border:1px solid; border-radius:3px; cursor:pointer; transition:filter .15s; }
.btn:hover    { filter:brightness(1.1); }
.btn:disabled { opacity:.4; cursor:not-allowed; filter:none; }
.btn-primary { background:#3a6abf; color:#fff; border-color:#2a5aaf; }
.btn-default { background:#e8e8e8; color:#333; border-color:#bbb; }
.btn-success { background:#27ae60; color:#fff; border-color:#1e8449; }
.btn-danger  { background:#c0392b; color:#fff; border-color:#a93226; }
.btn-warn    { background:#e67e22; color:#fff; border-color:#d35400; }

/* 表單格線 */
.form-grid { padding:8px 12px; display:flex; flex-direction:column; gap:0; }
.form-row  { display:grid; grid-template-columns:140px 1fr 140px 1fr; border-bottom:1px solid #e0eaf5; min-height:32px; }
.form-cell { padding:4px 8px; display:flex; align-items:center; }
.lbl { background:#c8daf0; color:#1a3a6e; font-weight:bold; font-size:12px; border-right:1px solid #b0c4de; white-space:nowrap; }
.inp { background:#f5f9ff; }
.c3  { grid-column:span 3; }
.c4  { grid-column:span 4; }
.sys-lbl { background:#d8d8d8; color:#555; }

/* 批號總覽 */
.lot-overview {
  width: 100%; background: #f5f9ff; border: 1px solid #d0dce8;
  border-radius: 3px; padding: 5px 8px;
  display: flex; flex-direction: column; gap: 3px; min-height: 32px;
}
.lot-ov-empty { color: #aaa; font-style: italic; font-size: 12px; }
.lot-ov-row   { display: flex; align-items: center; gap: 8px; font-size: 12px; }
.lot-ov-proc  { color: #2a5aaf; font-weight: bold; white-space: nowrap; min-width: 90px; }
.lot-ov-lots  { color: #1a2b4a; }

/* 系統 LOG */
.sys-log {
  width: 100%; max-height: 110px; overflow-y: auto;
  background: #f0f0f0; border: 1px solid #d0d8e0; border-radius: 3px;
  padding: 5px 8px; font-size: 12px;
  display: flex; flex-direction: column; gap: 1px;
}
.log-empty { color: #aaa; font-style: italic; text-align: center; padding: 4px; }
.log-entry { display: flex; align-items: baseline; gap: 6px; line-height: 1.7; border-bottom: 1px solid #e4e8ee; }
.log-entry:last-child { border-bottom: none; }
.log-time  { color: #7a8fa8; white-space: nowrap; font-size: 11px; flex-shrink: 0; }
.log-dot   { color: #bbb; }
.log-msg   { color: #333; word-break: break-all; }

.log-entry.create  .log-msg { color: #1a7a40; }
.log-entry.confirm .log-msg { color: #7a5500; font-weight: bold; }
.log-entry.input   .log-msg { color: #1a4a8a; }
.log-entry.edit    .log-msg { color: #6a3a9a; }
.log-entry.cancel  .log-msg { color: #aa2222; }
.log-entry.info    .log-msg { color: #333; }
.sec-header { background:#4a7abf; color:#fff; font-weight:bold; font-size:12px; padding:4px 12px; }
.req-lbl { color:#c0392b; }

/* 工單清單 */
.wo-list {
  width: 100%; background: #f5f9ff; border: 1px solid #d0dce8;
  border-radius: 3px; padding: 5px 8px; min-height: 32px;
}
.lock-table { width: auto; min-width: 360px; }
.wo-empty { color: #aaa; font-style: italic; font-size: 12px; padding: 4px; }
.wo-no    { font-family: monospace; font-weight: bold; color: #1a3a6e; }
.wo-badge { display: inline-block; padding: 1px 8px; border-radius: 8px; font-size: 11px; font-weight: bold; }
.badge-self { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }
.badge-out  { background: #fff3cd; color: #856404; border: 1px solid #ffc107; }

/* 輸入元件 */
.f-input {
  width:100%; padding:3px 6px; border:1px solid #b0c4de; border-radius:3px;
  font-size:13px; font-family:inherit; background:#fff;
}
.f-input[readonly], .f-input:disabled { background:#edf3fa; color:#555; cursor:default; }
.f-input:not([readonly]):not(:disabled):focus { outline:none; border-color:#3a6abf; box-shadow:0 0 0 2px rgba(58,106,191,.2); }
.req-inp { border-color:#c0392b; }
.f-textarea {
  width:100%; padding:4px 6px; border:1px solid #b0c4de; border-radius:3px;
  font-size:13px; font-family:inherit; resize:vertical;
}
.f-textarea[readonly], .f-textarea:disabled { background:#edf3fa; color:#555; cursor:default; }
.sys-ta { background:#f0f0f0 !important; color:#666; }

.chk-label { display:flex; align-items:center; gap:4px; margin-right:12px; cursor:pointer; }
.chk-label input { width:14px; height:14px; }
.tag { display:inline-block; background:#3a6abf; color:#fff; padding:1px 8px; border-radius:10px; font-size:12px; margin-right:6px; }
.file-name { font-size:12px; color:#666; margin-left:8px; }
.toolbar-id { font-size:12px; color:#1a3a6e; font-weight:bold; background:#dceeff; padding:2px 10px; border-radius:3px; border:1px solid #7aaad8; }
.status-badge { font-size:12px; padding:2px 10px; border-radius:10px; }
.status-badge.confirmed { background:#d4edda; color:#155724; border:1px solid #c3e6cb; font-weight:bold; }

/* 資料表格 */
.dtable { width:100%; border-collapse:collapse; font-size:12px; }
.dtable th { background:#3a6abf; color:#fff; padding:5px 8px; border:1px solid #2a5aaf; white-space:nowrap; text-align:center; }
.dtable td { padding:4px 8px; border:1px solid #d0dce8; text-align:center; white-space:nowrap; }
.dtable tr:nth-child(even) td { background:#f0f6ff; }
.dtable tr:hover td           { background:#dceeff; cursor:pointer; }
.dtable tr.row-sel td         { background:#c8e0ff; font-weight:bold; }
.empty-row { color:#999; padding:16px; text-align:center; }

/* Modal */
.modal-overlay { position:fixed; inset:0; background:rgba(0,0,0,.45); display:flex; align-items:center; justify-content:center; z-index:1000; }
.modal    { background:#fff; border-radius:6px; box-shadow:0 8px 32px rgba(0,0,0,.35); min-width:700px; max-width:92vw; max-height:86vh; display:flex; flex-direction:column; }
.modal-sm { min-width:440px; }
.modal-hdr { background:linear-gradient(135deg,#1a3a6e,#2855a0); color:#fff; padding:8px 16px; border-radius:6px 6px 0 0; display:flex; align-items:center; gap:10px; font-weight:bold; }
.picker-src   { font-size:11px; background:rgba(255,255,255,.15); padding:1px 8px; border-radius:8px; opacity:.9; }
.picker-count { font-size:12px; background:rgba(255,255,255,.2); padding:1px 8px; border-radius:8px; margin-left:4px; }
.modal-hdr .modal-x { margin-left:auto; }
.modal-x  { background:none; border:none; color:#fff; font-size:16px; cursor:pointer; padding:0 4px; }
.modal-body { padding:12px; overflow-y:auto; flex:1; }
.modal-ftr  { padding:8px 16px; border-top:1px solid #d0dce8; display:flex; gap:8px; justify-content:flex-end; }
</style>
