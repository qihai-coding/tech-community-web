<template>
  <div class="output-panel">
    <div class="output-header">
      <div class="output-tabs">
        <button :class="['tab', { active: activeTab === 'output' }]" @click="activeTab = 'output'">
          输出
        </button>
        <button :class="['tab', { active: activeTab === 'error' }]" @click="activeTab = 'error'">
          错误
        </button>
      </div>
      <div v-if="executionTime !== null" class="stats">
        <span class="stat-item">
          <span class="stat-label">执行时间:</span>
          <span class="stat-value">{{ executionTime }}ms</span>
        </span>
        <span class="stat-item">
          <span
            :class="[
              'status-badge',
              {
                success: status === 'success',
                error: status === 'error',
                timeout: status === 'timeout'
              }
            ]"
          >
            {{ statusText }}
          </span>
        </span>
      </div>
    </div>
    <div class="output-content">
      <div v-if="activeTab === 'output'" class="output-text">
        <pre v-if="output">{{ output }}</pre>
        <div v-else class="placeholder">{{ outputPlaceholder }}</div>
      </div>
      <div v-else class="error-text">
        <pre v-if="error" class="error-content">{{ error }}</pre>
        <div v-else class="placeholder">{{ errorPlaceholder }}</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'

interface Props {
  output?: string
  error?: string
  executionTime?: number | null
  status?: 'success' | 'error' | 'timeout'
}

const props = withDefaults(defineProps<Props>(), {
  output: '',
  error: '',
  executionTime: null,
  status: 'success'
})

const activeTab = ref<'output' | 'error'>('output')

const statusText = computed(() => {
  switch (props.status) {
    case 'success':
      return '成功'
    case 'error':
      return '错误'
    case 'timeout':
      return '超时'
    default:
      return '未知'
  }
})

const outputPlaceholder = computed(() => {
  if (props.executionTime === null) {
    return '运行代码后将在此处显示输出结果'
  }

  if (props.status === 'error' && props.error) {
    return '本次执行没有标准输出，请查看“错误”标签'
  }

  if (props.status === 'timeout') {
    return '程序执行超时，没有可显示的标准输出'
  }

  return '程序执行完成，但没有标准输出'
})

const errorPlaceholder = computed(() => {
  if (props.executionTime === null || props.status === 'success') {
    return '无错误信息'
  }

  return '未返回错误详情'
})

watch(
  () => [props.error, props.output, props.status] as const,
  ([error, output, status]) => {
    if (error && status !== 'success') {
      activeTab.value = 'error'
      return
    }

    if (output || status === 'success') {
      activeTab.value = 'output'
    }
  },
  { immediate: true }
)
</script>

<style scoped>
.output-panel {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: #1e1e1e;
  color: #d4d4d4;
  border-radius: 4px;
  overflow: hidden;
}

.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #252526;
  border-bottom: 1px solid #3e3e42;
}

.output-tabs {
  display: flex;
  gap: 4px;
}

.tab {
  padding: 6px 12px;
  background: transparent;
  border: none;
  color: #d4d4d4;
  cursor: pointer;
  border-radius: 4px;
  font-size: 13px;
  transition: background-color 0.2s;
}

.tab:hover {
  background: #2a2d2e;
}

.tab.active {
  background: #1e1e1e;
  color: #ffffff;
}

.stats {
  display: flex;
  gap: 16px;
  font-size: 12px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 4px;
}

.stat-label {
  color: #858585;
}

.stat-value {
  color: #4ec9b0;
  font-weight: 500;
}

.status-badge {
  padding: 2px 8px;
  border-radius: 3px;
  font-size: 11px;
  font-weight: 500;
}

.status-badge.success {
  background: #1a7f37;
  color: #ffffff;
}

.status-badge.error {
  background: #da3633;
  color: #ffffff;
}

.status-badge.timeout {
  background: #f0ad4e;
  color: #000000;
}

.output-content {
  flex: 1;
  overflow: auto;
  padding: 12px;
}

.output-text pre,
.error-text pre {
  margin: 0;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.6;
  white-space: pre-wrap;
  word-break: break-word;
}

.error-content {
  color: #f48771;
}

.placeholder {
  color: #858585;
  font-style: italic;
  text-align: center;
  padding: 24px;
}
</style>
