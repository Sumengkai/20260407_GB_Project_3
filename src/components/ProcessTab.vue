<template>
  <div>

    <!-- ══ 自產耗用 ══════════════════════════════════════════ -->
    <template v-if="showSelf">
      <div class="block-divider self-consumption" @click="toggle('selfConsumption')">
        <span class="tag tag-consumption">耗用</span>
        <span class="block-title">自產耗用</span>
        <button class="collapse-btn" :class="{ collapsed: !open.selfConsumption }">
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
        </button>
      </div>
      <div v-show="open.selfConsumption" class="block-body">
        <ConsumptionTab
          :show-self="true" :show-outsource="false"
          :self-rows="selfConsumptionRows"
          :outsource-rows="[]"
          @open-picker="$emit('open-picker', $event)"
          @cancel-rows="$emit('cancel-rows', $event)"
        />
      </div>
    </template>

    <!-- ══ 自產產出 ══════════════════════════════════════════ -->
    <template v-if="showSelf">
      <div class="block-divider self-output" @click="toggle('selfOutput')">
        <span class="tag tag-output">產出</span>
        <span class="block-title">自產產出</span>
        <button class="collapse-btn" :class="{ collapsed: !open.selfOutput }">
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
        </button>
      </div>
      <div v-show="open.selfOutput" class="block-body">
        <OutputTab
          :show-self="true" :show-outsource="false"
          :self-rows="selfOutputRows"
          :outsource-rows="[]"
          @add="$emit('output-add', $event)"
          @update="$emit('output-update', $event)"
          @delete-items="$emit('output-delete', $event)"
        />
      </div>
    </template>

    <!-- ══ 委外耗用 ══════════════════════════════════════════ -->
    <template v-if="showOutsource">
      <div class="block-divider outsource-consumption" @click="toggle('outsourceConsumption')">
        <span class="tag tag-consumption">耗用</span>
        <span class="block-title">委外耗用</span>
        <button class="collapse-btn" :class="{ collapsed: !open.outsourceConsumption }">
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
        </button>
      </div>
      <div v-show="open.outsourceConsumption" class="block-body">
        <ConsumptionTab
          :show-self="false" :show-outsource="true"
          :self-rows="[]"
          :outsource-rows="outsourceConsumptionRows"
          @open-picker="$emit('open-picker', $event)"
          @cancel-rows="$emit('cancel-rows', $event)"
        />
      </div>
    </template>

    <!-- ══ 委外產出 ══════════════════════════════════════════ -->
    <template v-if="showOutsource">
      <div class="block-divider outsource-output" @click="toggle('outsourceOutput')">
        <span class="tag tag-output">產出</span>
        <span class="block-title">委外產出</span>
        <button class="collapse-btn" :class="{ collapsed: !open.outsourceOutput }">
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
          <span class="hamburger-line"></span>
        </button>
      </div>
      <div v-show="open.outsourceOutput" class="block-body">
        <OutputTab
          :show-self="false" :show-outsource="true"
          :self-rows="[]"
          :outsource-rows="outsourceOutputRows"
          @add="$emit('output-add', $event)"
          @update="$emit('output-update', $event)"
          @delete-items="$emit('output-delete', $event)"
        />
      </div>
    </template>

  </div>
</template>

<script setup>
import { reactive } from 'vue'
import ConsumptionTab from './ConsumptionTab.vue'
import OutputTab from './OutputTab.vue'

defineProps({
  showSelf:                 { type: Boolean, default: true },
  showOutsource:            { type: Boolean, default: true },
  selfConsumptionRows:      { type: Array,   default: () => [] },
  outsourceConsumptionRows: { type: Array,   default: () => [] },
  selfOutputRows:           { type: Array,   default: () => [] },
  outsourceOutputRows:      { type: Array,   default: () => [] },
})
defineEmits(['open-picker', 'cancel-rows', 'output-add', 'output-update', 'output-delete'])

// 各區塊展開狀態，預設全部展開
const open = reactive({
  selfConsumption:      false,
  selfOutput:           false,
  outsourceConsumption: false,
  outsourceOutput:      false,
})

function toggle(key) { open[key] = !open[key] }
</script>

<style scoped>
/* ── 共用基底 ─────────────────────────────────────────── */
.block-divider {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 9px 16px;
  font-size: 18px;
  font-weight: bold;
  letter-spacing: 3px;
  color: #fff;
  border-left: 6px solid rgba(255, 255, 255, 0.45);
  margin-top: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.25);
  cursor: pointer;
  user-select: none;
}
.block-divider:hover { filter: brightness(1.12); }

/* ── 四種區塊各自顏色 ──────────────────────────────────── */
.block-divider.self-consumption {
  background: linear-gradient(90deg, #162660, #2244aa);
  border-left-color: #5580e0;
}
.block-divider.self-output {
  background: linear-gradient(90deg, #0e4060, #1a70a8);
  border-left-color: #50b0e0;
}
.block-divider.outsource-consumption {
  background: linear-gradient(90deg, #0e3828, #1a6448);
  border-left-color: #40c080;
}
.block-divider.outsource-output {
  background: linear-gradient(90deg, #1a5030, #2a8858);
  border-left-color: #60d890;
}

/* ── 標題文字 ──────────────────────────────────────────── */
.block-title { flex: 1; }

/* ── 耗用 / 產出 標籤徽章 ─────────────────────────────── */
.tag {
  display: inline-block;
  padding: 1px 8px;
  font-size: 11px;
  font-weight: bold;
  letter-spacing: 1px;
  border-radius: 10px;
  line-height: 1.6;
}
.tag-consumption { background: rgba(255, 200, 80, 0.9);  color: #5a3000; }
.tag-output      { background: rgba(160, 240, 190, 0.9); color: #0a3a20; }

/* ── 收摺按鈕（三條橫線） ────────────────────────────── */
.collapse-btn {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 4px;
  width: 30px;
  height: 30px;
  padding: 0;
  background: rgba(255, 255, 255, 0.18);
  border: 1px solid rgba(255, 255, 255, 0.35);
  border-radius: 4px;
  cursor: pointer;
  flex-shrink: 0;
  transition: background 0.15s;
}
.collapse-btn:hover { background: rgba(255, 255, 255, 0.32); }

.hamburger-line {
  display: block;
  width: 16px;
  height: 2px;
  background: #fff;
  border-radius: 2px;
  transition: transform 0.2s, opacity 0.2s;
}

/* 收起狀態：中間線消失，上下兩線交叉成 × */
.collapse-btn.collapsed .hamburger-line:nth-child(1) { transform: translateY(6px) rotate(45deg); }
.collapse-btn.collapsed .hamburger-line:nth-child(2) { opacity: 0; }
.collapse-btn.collapsed .hamburger-line:nth-child(3) { transform: translateY(-6px) rotate(-45deg); }

/* ── 明細區塊 ─────────────────────────────────────────── */
.block-body { overflow: hidden; }
</style>
