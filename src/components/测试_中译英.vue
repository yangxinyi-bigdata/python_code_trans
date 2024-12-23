<template>
  <div class="flex h-screen bg-gray-100">
    <!-- Sidebar -->
    <div class="w-20 bg-white shadow-md">
      <div class="flex flex-col items-center py-4">
        <button
          v-for="tab in tabs"
          :key="tab.name"
          :class="[
            'p-3 rounded-lg mb-4',
            activeTab === tab.name ? 'bg-blue-100 text-blue-600' : 'text-gray-600 hover:bg-gray-100'
          ]"
          @click="setActiveTab(tab.name)"
        >
          <component :is="tab.icon" class="w-6 h-6" :class="{ 'transform rotate-180': tab.name === '英译中' }" />
        </button>
      </div>
    </div>

    <!-- Main content -->
    <div class="flex-1 p-8">
      <h1 class="text-2xl font-semibold mb-6">{{ activeTab }}</h1>
      <div class="bg-white rounded-lg shadow-md p-6">
        <h2 class="text-lg font-medium mb-4">翻译模式:</h2>
        <div class="flex space-x-4 mb-6">
          <label v-for="mode in translationModes" :key="mode" class="flex items-center">
            <input
              type="radio"
              class="form-radio text-blue-600"
              name="translationMode"
              :value="mode"
              v-model="translationMode"
            />
            <span class="ml-2">{{ mode }}</span>
          </label>
        </div>

        <div class="mb-4">
          <label for="projectPath" class="block text-sm font-medium text-gray-700 mb-1">
            项目路径:
          </label>
          <div class="flex">
            <input
              type="text"
              id="projectPath"
              class="flex-grow px-3 py-2 border border-gray-300 rounded-l-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="选择项目路径"
              v-model="projectPath"
            />
            <button
              class="px-3 py-2 bg-blue-600 text-white rounded-r-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
              @click="selectProjectPath"
            >
              选择
            </button>
          </div>
        </div>

        <div class="mb-6">
          <label for="translationPath" class="block text-sm font-medium text-gray-700 mb-1">
            翻译路径:
          </label>
          <div class="flex">
            <input
              type="text"
              id="translationPath"
              class="flex-grow px-3 py-2 border border-gray-300 rounded-l-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="选择翻译路径"
              v-model="translationPath"
            />
            <button
              class="px-3 py-2 bg-blue-600 text-white rounded-r-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
              @click="selectTranslationPath"
            >
              选择
            </button>
          </div>
        </div>

        <div class="flex space-x-4 mb-6">
          <button
            class="px-4 py-2 rounded-md text-white"
            :class="isTranslating ? 'bg-gray-400 cursor-not-allowed' : 'bg-blue-600 hover:bg-blue-700'"
            @click="startTranslation"
            :disabled="isTranslating"
          >
            开始翻译
          </button>
          <button
            class="px-4 py-2 rounded-md text-white"
            :class="!isTranslating ? 'bg-gray-400 cursor-not-allowed' : 'bg-red-600 hover:bg-red-700'"
            @click="stopTranslation"
            :disabled="!isTranslating"
          >
            停止翻译
          </button>
        </div>

        <div v-if="isTranslating" class="mb-4">
          <div class="w-full bg-gray-200 rounded-full h-2.5">
            <div
              class="bg-blue-600 h-2.5 rounded-full transition-all duration-500 ease-out"
              :style="{ width: `${state.progress}%` }"
            ></div>
          </div>
        </div>

        <div class="bg-gray-100 rounded-md p-4">
          <h3 class="text-sm font-medium text-gray-700 mb-2">执行进程:</h3>
          <p class="text-sm text-gray-600">
            {{ isTranslating ? state.状态信息 : '等待开始翻译' }}
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onUnmounted } from 'vue'
import { CodeBracketIcon, LanguageIcon } from '@heroicons/vue/24/outline'
import { ElMessage } from "element-plus"
import http from "@/utils/http"

const activeTab = ref('中译英')
const translationMode = ref('直接在原项目中')
const projectPath = ref('')
const translationPath = ref('')
const isTranslating = ref(false)

const tabs = [
  { name: '中译英', icon: LanguageIcon },
  { name: '英译中', icon: LanguageIcon },
  { name: '代码解释', icon: CodeBracketIcon },
]

const translationModes = ['直接在原项目中', '新项目', '单个Py代码']

const state = reactive({
  progress: 0,
  progressTimer: null,
  状态信息: "",
  状态计时器: null,
  bannerDialogVisible: false,
  translationStarted: false,
})

const setActiveTab = (tabName) => {
  activeTab.value = tabName
}

const selectProjectPath = async () => {
  try {
    const result = await window.myAPI.openDirectory()
    if (result.filePaths && result.filePaths.length > 0) {
      projectPath.value = result.filePaths[0]
    }
  } catch (error) {
    console.error('选择项目路径时出错:', error)
  }
}

const selectTranslationPath = async () => {
  try {
    const result = await window.myAPI.openDirectory()
    if (result.filePaths && result.filePaths.length > 0) {
      translationPath.value = result.filePaths[0]
    }
  } catch (error) {
    console.error('选择翻译路径时出错:', error)
  }
}

const startTranslation = () => {
  console.log("开始翻译")
  console.log(projectPath.value)
  console.log(translationPath.value)
  state.translationStarted = true
  // 假设 $http 是全局注入的，如果不是，需要导入或以其他方式获取
  http.开始翻译_中文项目({ 项目路径: projectPath.value, 翻译路径: translationPath.value })
    .then(() => {
      startPollingProgress()
      开始获取状态()
    })
    .catch((resp) => {
      ElMessage.error("error: " + resp["message"])
    })
}

const stopTranslation = () => {
  console.log("停止翻译")
  http.停止翻译_英文项目()
    .then(() => {
      clearInterval(state.状态计时器)
      state.状态信息 = ""
      clearInterval(state.progressTimer)
      state.progress = 0
      state.translationStarted = false
    })
}

const startPollingProgress = () => {
  state.progressTimer = setInterval(fetchProgress, 1000)
}

const fetchProgress = () => {
  http.获取进度()
    .then((resp) => {
      const progressData = resp.data
      state.progress = progressData["任务进度"]
      console.log("当前进度: " + state.progress)
      if (state.progress === 'completed' || state.progress >= 100) {
        clearInterval(state.progressTimer)
        state.translationStarted = false
        ElMessage.success("翻译成功, 生成代码路径:" + translationPath.value)
        state.progress = 100
      }
    })
    .catch((error) => {
      console.error("Error fetching progress:", error)
      clearInterval(state.progressTimer)
    })
}

const 开始获取状态 = () => {
  state.状态计时器 = setInterval(获取状态, 1000)
}

const 获取状态 = () => {
  http.获取状态信息()
    .then((resp) => {
      const progressData = resp.data
      if (progressData["状态信息"] !== state.状态信息) {
        state.状态信息 = progressData["状态信息"]
        console.log("当前状态: " + state.状态信息)
      }
    })
    .catch((error) => {
      console.error("Error fetching progress:", error)
    })
}

// 清理定时器
onUnmounted(() => {
  if (state.progressTimer) clearInterval(state.progressTimer)
  if (state.状态计时器) clearInterval(state.状态计时器)
})
</script>