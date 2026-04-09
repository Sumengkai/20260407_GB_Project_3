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
      <span class="query-label">申請單單號</span>
      <div class="query-input-wrap">
        <span class="query-icon">🔍</span>
        <input
          v-model="queryNo"
          type="text"
          class="query-input"
          placeholder="請輸入申請單單號"
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

      <!-- ===== 0. D5申請單 ===== -->
      <div v-show="activeTab === 0" class="tab-panel">
        <div class="panel-toolbar">
          <span class="toolbar-label">功能</span>
          <div class="toolbar-id" v-if="d5.id">單號：{{ d5.id }}</div>
          <button v-if="!d5.id" class="btn btn-primary" @click="d5Create">新增</button>
          <button v-else class="btn btn-warn" @click="d5Update">修改</button>
        </div>
        <div class="form-grid">
          <div class="form-row">
            <div class="form-cell lbl req-lbl">交期 *</div>
            <div class="form-cell inp">
              <input type="date" v-model="d5.deliveryDate" class="f-input" />
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
              <label class="chk-label"><input type="checkbox" v-model="d5.typeMachining" /><span>機加工</span></label>
              <label class="chk-label"><input type="checkbox" v-model="d5.typeCoating" /><span>鍍膜</span></label>
              <label class="chk-label"><input type="checkbox" v-model="d5.typePurification" /><span>純化</span></label>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">爐次</div>
            <div class="form-cell inp">
              <input type="text" v-model="d5.furnaceNo" class="f-input" placeholder="請輸入爐次" />
            </div>
            <div class="form-cell lbl">費用</div>
            <div class="form-cell inp">
              <input type="number" v-model="d5.cost" class="f-input" placeholder="0" />
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">客戶</div>
            <div class="form-cell inp c3">
              <select v-model="d5.customer" class="f-input">
                <option value="">— 請選擇客戶 —</option>
                <option v-for="c in CUSTOMERS" :key="c.code" :value="c.code">{{ c.name }}（{{ c.code }}）</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-cell lbl">備註</div>
            <div class="form-cell inp c3">
              <textarea v-model="d5.remark" class="f-textarea" rows="2" placeholder="請輸入備註"></textarea>
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
            <div class="form-cell lbl sys-lbl">系統備註</div>
            <div class="form-cell inp c3">
              <div class="sys-log">
                <div v-if="!d5.sysLog.length" class="log-empty">— 尚無記錄 —</div>
                <div v-for="(entry, i) in d5.sysLog" :key="i" :class="['log-entry', entry.level]">
                  <span class="log-time">{{ entry.time }}</span>
                  <span class="log-dot">›</span>
                  <span class="log-msg">{{ entry.msg }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- ===== 1. 機加工（耗用 + 產出）===== -->
      <div v-show="activeTab === 1" class="tab-panel">
        <ProcessTab
          :show-self="true" :show-outsource="true"
          :self-consumption-rows="machining.consumption.self"
          :outsource-consumption-rows="machining.consumption.outsource"
          :self-output-rows="machining.output.self"
          :outsource-output-rows="machining.output.outsource"
          @open-picker="openPicker($event, 'machining')"
          @cancel-rows="cancelConsumption('machining', $event)"
          @output-add="handleOutputAdd('machining', $event)"
          @output-update="handleOutputUpdate('machining', $event)"
          @output-delete="handleOutputDelete('machining', $event)"
        />
      </div>

      <!-- ===== 2. 鍍膜（耗用 + 產出）===== -->
      <div v-show="activeTab === 2" class="tab-panel">
        <ProcessTab
          :show-self="false" :show-outsource="true"
          :self-consumption-rows="coating.consumption.self"
          :outsource-consumption-rows="coating.consumption.outsource"
          :self-output-rows="coating.output.self"
          :outsource-output-rows="coating.output.outsource"
          @open-picker="openPicker($event, 'coating')"
          @cancel-rows="cancelConsumption('coating', $event)"
          @output-add="handleOutputAdd('coating', $event)"
          @output-update="handleOutputUpdate('coating', $event)"
          @output-delete="handleOutputDelete('coating', $event)"
        />
      </div>

      <!-- ===== 3. 純化（耗用 + 產出）===== -->
      <div v-show="activeTab === 3" class="tab-panel">
        <ProcessTab
          :show-self="true" :show-outsource="false"
          :self-consumption-rows="purification.consumption.self"
          :outsource-consumption-rows="purification.consumption.outsource"
          :self-output-rows="purification.output.self"
          :outsource-output-rows="purification.output.outsource"
          @open-picker="openPicker($event, 'purification')"
          @cancel-rows="cancelConsumption('purification', $event)"
          @output-add="handleOutputAdd('purification', $event)"
          @output-update="handleOutputUpdate('purification', $event)"
          @output-delete="handleOutputDelete('purification', $event)"
        />
      </div>

      <!-- ===== 4. 品檢資訊 ===== -->
      <div v-show="activeTab === 4" class="tab-panel">
        <QualityTab
          :all-lots="allLots"
          :quality-data="qualityData"
          @change="handleQualityChange($event)"
        />
      </div>

    </div>

    <!-- ===== 庫存挑選 Modal（多選） ===== -->
    <div v-if="picker.show" class="modal-overlay" @click.self="picker.show = false">
      <div class="modal">
        <div class="modal-hdr">
          <span>挑選庫存</span>
          <span class="picker-src">
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
          <button class="btn btn-success" @click="confirmPicker">確認耗用（{{ picker.selIds.size }} 筆）</button>
          <button class="btn btn-default" @click="picker.show = false">取消</button>
        </div>
      </div>
    </div>


  </div>
</template>

<script setup>
import { ref, reactive, computed, watch } from 'vue'
import ProcessTab from './components/ProcessTab.vue'
import QualityTab from './components/QualityTab.vue'

// 日期
const now = new Date()
const days = ['日','一','二','三','四','五','六']
const currentDate = `${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} (${days[now.getDay()]})`

// 頁籤
// 全部頁籤定義（固定索引）
// 頁籤
const tabs = ['D5申請單', '機加工', '鍍膜', '純化', '品檢資訊']
const activeTab = ref(0)

// 訊息
const message = reactive({ text: '歡迎使用本系統...', type: 'info' })
const showMsg = (text, type='info') => { message.text = text; message.type = type }

// 查詢
const queryNo = ref('')
function doQuery() {
  const q = queryNo.value.trim()
  if (!q) { showMsg('請輸入申請單單號', 'error'); return }
  if (d5.id && d5.id.toUpperCase() === q.toUpperCase()) {
    activeTab.value = 0
    showMsg(`查詢成功：已載入申請單 ${d5.id}`, 'success')
  } else {
    showMsg(`查無申請單：${q}`, 'error')
  }
}

// D5 資料（欄位直接開放輸入，按新增完成建檔）
const d5 = reactive({
  id: null,
  deliveryDate:'', attachmentName:'',
  typeMachining:false, typeCoating:false, typePurification:false,
  furnaceNo:'', cost:null, customer:'', remark:'', sysLog:[]
})

// 客戶下拉資料
const CUSTOMERS = [
  { name:'正達', code:'GT' },
  { name:'永泉', code:'YQ' },
  { name:'盛新', code:'TS' },
  { name:'漢民', code:'HM' },
  { name:'環球晶', code:'GW' },
  { name:'合晶',  code:'WW' },
  { name:'臺譜',  code:'TP' },
]

// 製程 / 區塊 中文對照
const PROC_NAME = { machining:'機加工', coating:'鍍膜', purification:'純化' }
const SEC_NAME  = { self:'自產', outsource:'委外' }

// 寫入系統 LOG（同 key 只記錄一次）
function addLog(msg, level = 'info', key = null) {
  if (key && d5.sysLog.some(e => e.key === key)) return
  const now = new Date()
  const time = `${now.getFullYear()}/${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}:${String(now.getSeconds()).padStart(2,'0')}`
  d5.sysLog.push({ time, msg, level, key })
}
function removeLog(key) {
  const idx = d5.sysLog.findIndex(e => e.key === key)
  if (idx >= 0) d5.sysLog.splice(idx, 1)
}
function procCount(proc, type) {
  const p = getProc(proc)
  return p[type].self.length + p[type].outsource.length
}

// 依委託類型動態顯示頁籤（需在 d5 定義後）
const visibleTabIndices = computed(() => {
  const idx = [0]
  if (d5.id) {
    if (d5.typeMachining)    idx.push(1)
    if (d5.typeCoating)      idx.push(2)
    if (d5.typePurification) idx.push(3)
    idx.push(4) // 品檢資訊
  }
  return idx
})
watch(visibleTabIndices, (newList) => {
  if (!newList.includes(activeTab.value)) activeTab.value = 0
})
function d5Create() {
  if (!d5.deliveryDate) { showMsg('交期為必填欄位', 'error'); return }
  // 模擬後端自動建立單號
  const ym = `${new Date().getFullYear()}${String(new Date().getMonth()+1).padStart(2,'0')}`
  d5.id = `D5-${ym}-${String(Math.floor(Math.random()*9000)+1000)}`
  addLog(`建立 D5 申請單，單號：${d5.id}`, 'create')
  showMsg(`D5 申請單建檔完成，單號：${d5.id}`, 'success')
}
function d5Update() {
  if (!d5.deliveryDate) { showMsg('交期為必填欄位', 'error'); return }
  removeLog('d5-update')
  addLog(`修改 D5 申請單，單號：${d5.id}`, 'info', 'd5-update')
  showMsg(`D5 申請單已修改，單號：${d5.id}`, 'success')
}
function d5FileChange(e) { d5.attachmentName = e.target.files[0]?.name || '' }

// 各製程資料
function mkProc() { return { consumption:{self:[], outsource:[]}, output:{self:[], outsource:[]} } }
const machining    = reactive(mkProc())
const coating      = reactive(mkProc())
const purification = reactive(mkProc())

const getProc = n => ({machining,coating,purification}[n])

// 庫別代碼對照
const WAREHOUSE_LABEL = { BB:'碳材料生產工廠', BBPN:'D8屏南儲區', BBOUTTMP:'委外暫存庫' }

// 庫存假資料（含庫別代碼）
const allInv = [
  { itemNo:'ACS15U', itemName:'超級電容ACS15U',  warehouse:'BB',       lotNo:'GB-001', qty:500, unit:'kg'  },
  { itemNo:'GC-001', itemName:'石墨化碳材料',     warehouse:'BB',       lotNo:'GB-002', qty:200, unit:'kg'  },
  { itemNo:'CB-050', itemName:'碳微球CB-050',     warehouse:'BB',       lotNo:'GB-003', qty:150, unit:'kg'  },
  { itemNo:'PG-100', itemName:'純化石墨PG-100',   warehouse:'BBPN',     lotNo:'GB-004', qty:80,  unit:'kg'  },
  { itemNo:'MC-200', itemName:'機加工碳塊MC-200', warehouse:'BBPN',     lotNo:'GB-005', qty:320, unit:'pcs' },
  { itemNo:'CF-300', itemName:'碳纖維CF-300',     warehouse:'BBOUTTMP', lotNo:'GB-006', qty:100, unit:'kg'  },
  { itemNo:'GR-400', itemName:'石墨電極GR-400',   warehouse:'BBOUTTMP', lotNo:'GB-007', qty:60,  unit:'pcs' },
]

// 自產：只允許 BB；委外：排除 BBPN
function getBaseInv() {
  if (picker.section === 'self') return allInv.filter(i => i.warehouse === 'BB')
  return allInv.filter(i => i.warehouse !== 'BBPN')
}

const filteredInv = ref([...allInv])
const picker = reactive({ show:false, keyword:'', selIds: new Set(), proc:'', section:'' })
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
  picker.proc = procName; picker.section = section
  picker.keyword = ''; picker.selIds.clear()
  filteredInv.value = getBaseInv()
  picker.show = true
}
function confirmPicker() {
  if (!picker.selIds.size) { showMsg('請先勾選庫存項目', 'error'); return }
  const arr = getProc(picker.proc).consumption[picker.section]
  const names = []
  picker.selIds.forEach(i => {
    const inv = filteredInv.value[i]
    arr.push({ id: Date.now() + i, ...inv })
    names.push(inv.itemName)
  })
  const count = picker.selIds.size
  picker.show = false
  const logKey = `consumption-${picker.proc}`
  addLog(`輸入【${PROC_NAME[picker.proc]}(耗用)】`, 'input', logKey)
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

// 產出明細處理（行內新增/修改/刪除）
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
// 品檢資訊 { lotNo: { paramKey: value } }
const qualityData = reactive({})

// 批號總覽：收集各製程產出的所有批號（去重、過濾空值）
const lotOverview = computed(() => {
  const entries = [
    { proc: '機加工', data: machining.output },
    { proc: '鍍膜',   data: coating.output   },
    { proc: '純化',   data: purification.output },
  ]
  return entries.reduce((acc, { proc, data }) => {
    const lots = [...new Set(
      [...data.self, ...data.outsource]
        .map(r => r.lotNo).filter(Boolean)
    )]
    if (lots.length) acc.push({ proc, lots: lots.join('、') })
    return acc
  }, [])
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
  collect('coating',   '鍍膜')
  collect('purification', '純化')
  return lots
})

function handleQualityChange({ lotNo, paramKey, value }) {
  if (!qualityData[lotNo]) qualityData[lotNo] = {}
  qualityData[lotNo][paramKey] = value
  addLog(`輸入【品檢資訊】批號 ${lotNo}`, 'input', `quality-${lotNo}`)
}

function handleOutputDelete(proc, { section, ids }) {
  const idSet = new Set(ids)
  const arr = getProc(proc).output[section]
  const removed = idSet.size
  arr.splice(0, arr.length, ...arr.filter(r => !idSet.has(r.id)))
  if (procCount(proc, 'output') === 0) removeLog(`output-${proc}`)
  showMsg(`已刪除 ${removed} 筆產出明細`, 'info')
}
</script>

<style scoped>
.erp-app { min-height:100vh; display:flex; flex-direction:column; background:#dce6f0; font-size:13px; }

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
  width: 220px;
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
  background:#d0e0f0; color:#1a3a6e; cursor:pointer; transition:background .15s;
}
.tab-btn:hover  { background:#e0ecf8; }
.tab-btn.active { background:#fff; font-weight:bold; border-color:#3a6abf; }

/* 頁籤內容 */
.tab-content { flex:1; padding:12px; }
.tab-panel   { background:#fff; border:1px solid #b0c4de; border-radius:0 4px 4px 4px; }

/* 工具列 */
.panel-toolbar {
  background:#d8e8f8; border-bottom:1px solid #b0c4de;
  padding:6px 12px; display:flex; align-items:center; gap:6px; flex-wrap:wrap;
}
.toolbar-label { background:#3a6abf; color:#fff; padding:2px 10px; border-radius:3px; font-size:12px; margin-right:4px; }

/* 按鈕 */
.btn { padding:3px 12px; font-size:12px; font-family:inherit; border:1px solid; border-radius:3px; cursor:pointer; transition:filter .15s; }
.btn:hover    { filter:brightness(1.1); }
.btn:disabled { opacity:.4; cursor:not-allowed; filter:none; }
.btn-primary { background:#3a6abf; color:#fff; border-color:#2a5aaf; }
.btn-default { background:#e8e8e8; color:#333; border-color:#bbb; }
.btn-success { background:#27ae60; color:#fff; border-color:#1e8449; }
.btn-danger  { background:#c0392b; color:#fff; border-color:#a93226; }

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

/* log 等級色彩 */
.log-entry.create  .log-msg { color: #1a7a40; }
.log-entry.confirm .log-msg { color: #7a5500; font-weight: bold; }
.log-entry.input   .log-msg { color: #1a4a8a; }
.log-entry.edit    .log-msg { color: #6a3a9a; }
.log-entry.cancel  .log-msg { color: #aa2222; }
.log-entry.info    .log-msg { color: #333; }
.sec-header { background:#4a7abf; color:#fff; font-weight:bold; font-size:12px; padding:4px 12px; }
.req-lbl { color:#c0392b; }

/* 輸入元件 */
.f-input {
  width:100%; padding:3px 6px; border:1px solid #b0c4de; border-radius:3px;
  font-size:13px; font-family:inherit; background:#fff;
}
.f-input:disabled { background:#edf3fa; color:#555; }
.f-input:focus    { outline:none; border-color:#3a6abf; box-shadow:0 0 0 2px rgba(58,106,191,.2); }
.req-inp { border-color:#c0392b; }
.f-textarea {
  width:100%; padding:4px 6px; border:1px solid #b0c4de; border-radius:3px;
  font-size:13px; font-family:inherit; resize:vertical;
}
.f-textarea:disabled { background:#edf3fa; }
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
