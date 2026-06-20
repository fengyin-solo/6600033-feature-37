<template>
  <div class="min-h-screen bg-slate-900 text-slate-200">
    <header class="border-b border-slate-700 px-6 py-4">
      <h1 class="text-2xl font-bold text-cyan-400">蒙特卡洛模拟与统计假设检验平台</h1>
      <p class="text-sm text-slate-500 mt-1">随机采样模拟 · 6种MC场景 · 假设检验 · 置信区间可视化</p>
    </header>
    <div class="flex flex-col lg:flex-row gap-4 p-4">
      <div class="lg:w-1/4 space-y-4">
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">模拟场景</h3>
          <div class="space-y-1">
            <div v-for="s in SCENARIOS" :key="s.id" @click="store.setScenario(s)"
              :class="['cursor-pointer p-2 rounded border text-sm transition-all', store.currentScenario.id === s.id ? 'border-cyan-500 bg-cyan-900/30 text-cyan-400' : 'border-slate-700 text-slate-300 hover:border-slate-500']">
              <div class="font-bold">{{ s.name }}</div>
              <div class="text-xs text-slate-500 mt-0.5">{{ s.description }}</div>
            </div>
          </div>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">参数控制</h3>
          <label class="text-xs text-slate-500">迭代次数: {{ store.iterations }}</label>
          <input type="range" min="100" max="5000" step="100" v-model.number="store.iterations" class="w-full mt-1 mb-3 accent-cyan-500" />
          <button @click="store.runSimulation" :disabled="store.isRunning" class="w-full py-2 bg-cyan-600 hover:bg-cyan-500 disabled:opacity-50 rounded text-sm font-bold">
            {{ store.isRunning ? '运行中...' : '▶ 开始模拟' }}
          </button>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700 text-sm">
          <h3 class="text-sm font-bold text-slate-400 mb-3">模拟结果</h3>
          <div v-if="store.result" class="space-y-2">
            <div class="flex justify-between"><span class="text-slate-500">估算值</span><span class="text-cyan-400 font-bold font-mono">{{ store.result.estimate.toFixed(6) }}</span></div>
            <div v-if="store.result.trueValue !== undefined" class="flex justify-between"><span class="text-slate-500">真实值</span><span class="text-green-400 font-mono">{{ store.result.trueValue.toFixed(6) }}</span></div>
            <div v-if="store.result.error !== undefined" class="flex justify-between"><span class="text-slate-500">误差</span><span class="text-orange-400 font-mono">{{ store.result.error.toFixed(6) }}</span></div>
            <div class="flex justify-between"><span class="text-slate-500">样本数</span><span class="text-slate-300">{{ store.result.iterations }}</span></div>
          </div>
          <div v-else class="py-6 text-center">
            <div class="text-2xl opacity-60">🧮</div>
            <div class="text-xs text-slate-400 font-bold mt-2">尚未运行模拟</div>
            <div class="text-xs text-slate-500 mt-1">选择场景并点击「开始模拟」，运行后将展示估算值、真实值与误差</div>
          </div>
        </div>
      </div>
      <div class="lg:w-3/4 space-y-4">
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">收敛过程</h3>
          <div class="relative w-full">
            <div ref="convergenceRef" class="w-full rounded" style="height:240px;background:#0f172a;"></div>
            <div v-if="!store.result" class="absolute inset-0 flex flex-col items-center justify-center text-center px-6">
              <div class="text-3xl opacity-60">📈</div>
              <div class="text-sm font-bold text-slate-400 mt-2">暂无收敛数据</div>
              <div class="text-xs text-slate-500 mt-1 max-w-xs">请在左侧选择模拟场景，点击下方按钮开始模拟，观察估算值随样本量增加的收敛过程</div>
              <button @click="store.runSimulation" :disabled="store.isRunning" class="mt-3 px-3 py-1 bg-cyan-600 hover:bg-cyan-500 disabled:opacity-50 rounded text-xs font-bold">▶ 开始模拟</button>
            </div>
          </div>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">样本分布直方图</h3>
          <div class="relative w-full">
            <div ref="histogramRef" class="w-full rounded" style="height:220px;background:#0f172a;"></div>
            <div v-if="!store.result" class="absolute inset-0 flex flex-col items-center justify-center text-center px-6">
              <div class="text-3xl opacity-60">📊</div>
              <div class="text-sm font-bold text-slate-400 mt-2">暂无样本分布</div>
              <div class="text-xs text-slate-500 mt-1 max-w-xs">运行模拟后，将在此展示本次采样的分布直方图</div>
              <button @click="store.runSimulation" :disabled="store.isRunning" class="mt-3 px-3 py-1 bg-cyan-600 hover:bg-cyan-500 disabled:opacity-50 rounded text-xs font-bold">▶ 开始模拟</button>
            </div>
          </div>
        </div>
        <div class="bg-slate-800 rounded-lg p-4 border border-slate-700">
          <h3 class="text-sm font-bold text-slate-400 mb-3">假设检验 (独立样本 T 检验)</h3>
          <div class="grid grid-cols-2 gap-4 mb-3">
            <div>
              <label class="text-xs text-slate-500">样本组A (逗号分隔)</label>
              <textarea v-model="group1Input" rows="2" class="w-full mt-1 bg-slate-900 border border-slate-600 rounded px-2 py-1 text-xs font-mono focus:outline-none focus:border-cyan-500 resize-none"></textarea>
            </div>
            <div>
              <label class="text-xs text-slate-500">样本组B (逗号分隔)</label>
              <textarea v-model="group2Input" rows="2" class="w-full mt-1 bg-slate-900 border border-slate-600 rounded px-2 py-1 text-xs font-mono focus:outline-none focus:border-cyan-500 resize-none"></textarea>
            </div>
          </div>
          <button @click="runTest" :disabled="!testInputValid" class="px-4 py-1.5 bg-purple-600 hover:bg-purple-500 disabled:opacity-50 rounded text-sm">执行T检验</button>
          <div v-if="store.testResult" class="mt-3 grid grid-cols-4 gap-3 text-sm">
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">统计量 t</div><div class="text-cyan-400 font-bold font-mono">{{ store.testResult.statistic }}</div></div>
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">p 值</div><div class="font-bold font-mono" :class="store.testResult.significant ? 'text-red-400' : 'text-green-400'">{{ store.testResult.pValue }}</div></div>
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">自由度 df</div><div class="text-slate-300 font-mono">{{ store.testResult.df }}</div></div>
            <div class="bg-slate-900 rounded p-2 text-center"><div class="text-xs text-slate-500 mb-1">显著性</div><div class="text-xs font-bold" :class="store.testResult.significant ? 'text-red-400' : 'text-green-400'">{{ store.testResult.significant ? '显著(p<0.05)' : '不显著' }}</div></div>
          </div>
          <div v-else class="mt-3 py-6 text-center bg-slate-900 rounded">
            <div class="text-2xl opacity-60">🔬</div>
            <template v-if="testInputValid">
              <div class="text-xs text-slate-400 font-bold mt-2">已就绪</div>
              <div class="text-xs text-slate-500 mt-1">数据有效，点击「执行T检验」查看统计量 t、p 值与显著性结论</div>
            </template>
            <template v-else>
              <div class="text-xs text-slate-400 font-bold mt-2">暂无检验结果</div>
              <div class="text-xs text-slate-500 mt-1">请为样本组A、B各输入至少2个数值（逗号分隔），再执行T检验</div>
            </template>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import * as echarts from 'echarts'
import { useMCStore, SCENARIOS } from './store/mc'

const store = useMCStore()
const convergenceRef = ref<HTMLDivElement | null>(null)
const histogramRef = ref<HTMLDivElement | null>(null)
const group1Input = ref('5.1,4.8,5.3,4.9,5.2,5.0,4.7,5.1,5.4,4.8')
const group2Input = ref('4.6,4.2,4.9,4.3,4.5,4.7,4.4,4.8,4.1,4.6')
let convChart: echarts.ECharts | null = null
let histChart: echarts.ECharts | null = null

function initCharts() {
  if (convergenceRef.value) convChart = echarts.init(convergenceRef.value, 'dark')
  if (histogramRef.value) histChart = echarts.init(histogramRef.value, 'dark')
}

function updateCharts() {
  if (convChart && store.convergenceData.length > 0) {
    convChart.setOption({
      backgroundColor: '#0f172a',
      grid: { top: 20, bottom: 35, left: 65, right: 20 },
      xAxis: { type: 'value', axisLabel: { color: '#94a3b8', fontSize: 10 } },
      yAxis: { type: 'value', axisLabel: { color: '#94a3b8', fontSize: 10 } },
      series: [{ type: 'line', data: store.convergenceData, smooth: true, lineStyle: { color: '#06b6d4', width: 2 }, areaStyle: { color: 'rgba(6,182,212,0.1)' }, symbol: 'none' }],
      tooltip: { trigger: 'axis', backgroundColor: '#1e293b', borderColor: '#475569' }
    })
  }
  if (histChart && store.histogramData.xAxis.length > 0) {
    histChart.setOption({
      backgroundColor: '#0f172a',
      grid: { top: 15, bottom: 40, left: 55, right: 15 },
      xAxis: { type: 'category', data: store.histogramData.xAxis, axisLabel: { color: '#94a3b8', fontSize: 9, rotate: 30 } },
      yAxis: { type: 'value', axisLabel: { color: '#94a3b8', fontSize: 10 } },
      series: [{ type: 'bar', data: store.histogramData.data, itemStyle: { color: '#8b5cf6' } }],
      tooltip: { trigger: 'axis', backgroundColor: '#1e293b', borderColor: '#475569' }
    })
  }
}

function parseGroup(input: string): number[] {
  return input.split(',').map(s => parseFloat(s.trim())).filter(n => !isNaN(n))
}
const testInputValid = computed(() => parseGroup(group1Input.value).length > 1 && parseGroup(group2Input.value).length > 1)
function runTest() {
  const g1 = parseGroup(group1Input.value)
  const g2 = parseGroup(group2Input.value)
  if (g1.length > 1 && g2.length > 1) store.runTest(g1, g2)
}

onMounted(() => { initCharts(); store.runSimulation() })
watch(() => store.result, () => updateCharts(), { deep: true })
</script>
