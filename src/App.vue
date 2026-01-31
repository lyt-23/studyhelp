<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-50">
    <!-- 顶部导航栏 -->
    <header class="bg-white shadow-sm sticky top-0 z-10">
      <div class="container mx-auto px-4 py-4 flex justify-between items-center">
        <div class="flex items-center gap-2">
          <div class="w-10 h-10 bg-gradient rounded-full flex items-center justify-center">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6V4m0 2a2 2 0 100 4m0-4a2 2 0 110 4m-6 8a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4m6 6v10m6-2a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4" />
            </svg>
          </div>
          <h1 class="text-xl font-bold text-gray-800">学习记录助手</h1>
        </div>
        <div class="flex items-center gap-4">
          <button @click="showHistory = !showHistory" class="btn btn-outline">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            历史记录
          </button>
        </div>
      </div>
    </header>

    <!-- 主内容区 -->
    <main class="container mx-auto px-4 py-8">
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <!-- 左侧：计时器和统计 -->
        <div class="lg:col-span-1">
          <div class="card mb-6">
            <h2 class="text-lg font-semibold mb-4">学习计时器</h2>
            <div class="flex flex-col items-center">
              <div class="text-5xl font-bold mb-6 text-gray-800" id="timer">
                {{ formatTime(studyTime) }}
              </div>
              <div class="flex gap-4">
                <button 
                  v-if="!isStudying" 
                  @click="startStudy" 
                  class="btn btn-primary"
                >
                  开始学习
                </button>
                <button 
                  v-else 
                  @click="pauseStudy" 
                  class="btn btn-accent"
                >
                  暂停
                </button>
                <button 
                  v-if="isStudying" 
                  @click="stopStudy(true)" 
                  class="btn btn-outline"
                >
                  结束学习
                </button>
              </div>
            </div>
          </div>

          <div class="card">
            <h2 class="text-lg font-semibold mb-4">学习统计</h2>
            <div class="space-y-4">
              <div class="flex justify-between items-center">
                <span class="text-gray-600">今日学习时长</span>
                <span class="font-semibold">{{ formatTime(todayStudyTime) }}</span>
              </div>
              <div class="flex justify-between items-center">
                <span class="text-gray-600">本周学习时长</span>
                <span class="font-semibold">{{ formatTime(weekStudyTime) }}</span>
              </div>
              <div class="flex justify-between items-center">
                <span class="text-gray-600">本月学习时长</span>
                <span class="font-semibold">{{ formatTime(monthStudyTime) }}</span>
              </div>
              <div class="w-full bg-gray-200 rounded-full h-2.5 mt-4">
                <div 
                  class="bg-primary h-2.5 rounded-full" 
                  :style="{ width: `${(todayStudyTime / (targetStudyTime * 3600)) * 100}%` }"
                ></div>
              </div>
              <p class="text-sm text-gray-500 text-center mt-2">
                今日学习目标：{{ targetStudyTime }}小时
              </p>
              <button 
                @click="showGoalSettings = true" 
                class="btn btn-outline w-full mt-2"
              >
                设置目标时长
              </button>
            </div>
          </div>

          <div class="card mt-6">
            <h2 class="text-lg font-semibold mb-4">学习计划</h2>
            <div class="space-y-3">
              <div class="flex items-center gap-2">
                <input 
                  v-model="newTodo" 
                  type="text" 
                  class="input-field flex-1" 
                  placeholder="添加学习任务..."
                  @keyup.enter="addTodo"
                />
                <button @click="addTodo" class="btn btn-primary">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" />
                  </svg>
                </button>
              </div>
              <div class="space-y-2 max-h-48 overflow-y-auto pr-2">
                <div 
                  v-for="(todo, index) in todos" 
                  :key="index"
                  class="flex items-center gap-2 bg-gray-50 p-3 rounded-lg"
                >
                  <input 
                    v-model="todo.completed" 
                    type="checkbox" 
                    class="rounded text-primary focus:ring-primary"
                    @change="saveTodos"
                  />
                  <span 
                    :class="['flex-1 text-sm', todo.completed ? 'line-through text-gray-500' : '']"
                  >
                    {{ todo.text }}
                  </span>
                  <button 
                    @click="removeTodo(index)" 
                    class="text-red-500 hover:text-red-700"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                  </button>
                </div>
                <div v-if="todos.length === 0" class="text-center py-4 text-gray-500 text-sm">
                  暂无学习任务
                </div>
              </div>
              <div class="flex justify-between items-center text-sm text-gray-600 mt-2">
                <span>已完成 {{ completedTodosCount }}/{{ todos.length }}</span>
                <button 
                  v-if="todos.length > 0" 
                  @click="clearCompletedTodos" 
                  class="text-gray-500 hover:text-gray-700"
                >
                  清除已完成
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- 右侧：笔记和总结 -->
        <div class="lg:col-span-2">
          <div class="card mb-6">
            <h2 class="text-lg font-semibold mb-4">学习笔记</h2>
            <textarea 
              v-model="notes" 
              class="textarea-field h-64" 
              placeholder="在这里记录你的学习笔记..."
            ></textarea>
          </div>

          <div class="card mb-6">
            <h2 class="text-lg font-semibold mb-4">上传学习资料</h2>
            <div class="space-y-4">
              <div class="border-2 border-dashed border-gray-300 rounded-lg p-6 text-center hover:border-primary transition-colors">
                <input 
                  type="file" 
                  @change="handleFileUpload" 
                  multiple 
                  accept="image/*,video/*,.pdf,.doc,.docx,.txt"
                  class="hidden" 
                  id="file-upload"
                />
                <label for="file-upload" class="cursor-pointer">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-12 w-12 mx-auto text-gray-400 mb-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12" />
                  </svg>
                  <p class="text-gray-600">点击或拖拽上传文件</p>
                  <p class="text-sm text-gray-400 mt-1">支持图片、视频、PDF、Word、文本文件</p>
                </label>
              </div>
              <div v-if="uploadedFiles.length > 0" class="space-y-2">
                <h4 class="text-sm font-medium text-gray-600">已上传文件：</h4>
                <div 
                  v-for="(file, index) in uploadedFiles" 
                  :key="index"
                  class="flex items-center justify-between bg-gray-50 rounded-lg p-3"
                >
                  <div class="flex items-center gap-3">
                    <div class="w-10 h-10 bg-primary/10 rounded-full flex items-center justify-center">
                      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-primary" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                      </svg>
                    </div>
                    <div>
                      <p class="text-sm font-medium">{{ file.name }}</p>
                      <p class="text-xs text-gray-500">{{ (file.size / 1024).toFixed(1) }} KB</p>
                    </div>
                  </div>
                  <button 
                    @click="removeFile(index)" 
                    class="text-red-500 hover:text-red-700"
                  >
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                    </svg>
                  </button>
                </div>
              </div>
            </div>
          </div>

          <div class="card">
            <h2 class="text-lg font-semibold mb-4">学习总结</h2>
            <div class="space-y-4">
              <input 
                v-model="studyTitle" 
                class="input-field" 
                placeholder="学习主题"
              />
              <textarea 
                v-model="summary" 
                class="textarea-field h-48" 
                placeholder="总结本次学习内容..."
              ></textarea>
              <button 
                @click="saveStudyRecord" 
                class="btn btn-secondary w-full"
                :disabled="!studyTitle || !summary"
              >
                保存学习记录
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- 历史记录弹窗 -->
      <div v-if="showHistory" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-xl shadow-xl max-w-2xl w-full max-h-[80vh] overflow-y-auto">
          <div class="p-6">
            <div class="flex justify-between items-center mb-4">
              <h2 class="text-xl font-bold">学习历史记录</h2>
              <button @click="showHistory = false" class="text-gray-500 hover:text-gray-700">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
            <div class="space-y-4">
              <div 
                v-for="(record, index) in studyRecords" 
                :key="index"
                @click="viewRecordDetail(record)"
                :class="['border rounded-lg p-4 cursor-pointer transition-all duration-200 hover:shadow-md', getCardColor(index)]"
              >
                <div class="flex justify-between items-start">
                  <h3 class="font-semibold">{{ record.title }}</h3>
                  <span class="text-sm text-gray-500">{{ record.date }}</span>
                </div>
                <p class="text-sm text-gray-600 mb-2">时长：{{ formatTime(record.duration) }}</p>
                <div v-if="record.notes" class="mb-3">
                  <h4 class="text-sm font-medium text-gray-600 mb-1">笔记：</h4>
                  <p class="text-gray-700 text-sm mb-2 line-clamp-2">{{ record.notes }}</p>
                </div>
                <div>
                  <h4 class="text-sm font-medium text-gray-600 mb-1">总结：</h4>
                  <p class="text-gray-700 line-clamp-2">{{ record.summary }}</p>
                </div>
                <div v-if="record.files && record.files.length > 0" class="mt-2 flex items-center gap-1 text-sm text-gray-600">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486L20.5 13" />
                  </svg>
                  <span>{{ record.files.length }} 个附件</span>
                </div>
                <div class="mt-3 text-sm text-primary hover:underline">
                  点击查看详情 →
                </div>
              </div>
              <div v-if="studyRecords.length === 0" class="text-center py-8 text-gray-500">
                暂无学习记录
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 鼓励提示弹窗 -->
      <div v-if="showEncouragement" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-xl shadow-xl max-w-md w-full p-6 text-center">
          <div class="text-6xl mb-4">🎉</div>
          <h2 class="text-xl font-bold mb-2">{{ encouragementMessage.title }}</h2>
          <p class="text-gray-600 mb-6">{{ encouragementMessage.message }}</p>
          <button @click="showEncouragement = false" class="btn btn-primary">
            确定
          </button>
        </div>
      </div>

      <!-- 记录详情弹窗 -->
      <div v-if="showRecordDetail" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-xl shadow-xl max-w-lg w-full p-6">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold">学习记录详情</h2>
            <button @click="showRecordDetail = false" class="text-gray-500 hover:text-gray-700">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <h3 class="text-sm font-medium text-gray-600 mb-1">学习主题</h3>
              <p class="font-semibold">{{ currentRecord?.title }}</p>
            </div>
            <div>
              <h3 class="text-sm font-medium text-gray-600 mb-1">学习时长</h3>
              <p>{{ formatTime(currentRecord?.duration || 0) }}</p>
            </div>
            <div>
              <h3 class="text-sm font-medium text-gray-600 mb-1">记录时间</h3>
              <p>{{ currentRecord?.date }}</p>
            </div>
            <div v-if="currentRecord?.notes">
              <h3 class="text-sm font-medium text-gray-600 mb-1">学习笔记</h3>
              <p class="whitespace-pre-line">{{ currentRecord?.notes }}</p>
            </div>
            <div>
              <h3 class="text-sm font-medium text-gray-600 mb-1">学习总结</h3>
              <p class="whitespace-pre-line">{{ currentRecord?.summary }}</p>
            </div>
            <div v-if="currentRecord?.files && currentRecord.files.length > 0">
              <h3 class="text-sm font-medium text-gray-600 mb-2">学习资料</h3>
              <div class="space-y-2">
                <div 
                  v-for="(file, index) in currentRecord.files" 
                  :key="index"
                  class="bg-gray-50 rounded-lg p-3"
                >
                  <div class="flex items-center gap-3 mb-2">
                    <div class="w-8 h-8 bg-primary/10 rounded-full flex items-center justify-center">
                      <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 text-primary" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                      </svg>
                    </div>
                    <p class="text-sm font-medium">{{ currentRecord.fileNames[index] }}</p>
                  </div>
                  <div v-if="file.startsWith('data:image')">
                    <img :src="file" class="max-w-full h-auto rounded-lg" alt="学习资料" />
                  </div>
                  <div v-else-if="file.startsWith('data:video')">
                    <video :src="file" controls class="max-w-full h-auto rounded-lg"></video>
                  </div>
                  <div v-else>
                    <a :href="file" :download="currentRecord.fileNames[index]" class="text-primary hover:underline text-sm">
                      点击下载文件
                    </a>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="mt-6 flex justify-end">
            <button @click="showRecordDetail = false" class="btn btn-outline">
              关闭
            </button>
          </div>
        </div>
      </div>

      <!-- 目标设置弹窗 -->
      <div v-if="showGoalSettings" class="fixed inset-0 bg-black/50 z-50 flex items-center justify-center p-4">
        <div class="bg-white rounded-xl shadow-xl max-w-md w-full p-6">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-bold">设置学习目标</h2>
            <button @click="showGoalSettings = false" class="text-gray-500 hover:text-gray-700">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-700 mb-2">今日学习目标（小时）</label>
              <input 
                v-model.number="targetStudyTime" 
                type="number" 
                min="1" 
                max="24" 
                class="input-field"
                placeholder="请输入目标时长"
              />
            </div>
            <div class="grid grid-cols-3 gap-2">
              <button 
                @click="targetStudyTime = 2" 
                class="btn btn-outline text-sm"
              >
                2小时
              </button>
              <button 
                @click="targetStudyTime = 4" 
                class="btn btn-outline text-sm"
              >
                4小时
              </button>
              <button 
                @click="targetStudyTime = 6" 
                class="btn btn-outline text-sm"
              >
                6小时
              </button>
              <button 
                @click="targetStudyTime = 8" 
                class="btn btn-outline text-sm"
              >
                8小时
              </button>
              <button 
                @click="targetStudyTime = 10" 
                class="btn btn-outline text-sm"
              >
                10小时
              </button>
              <button 
                @click="targetStudyTime = 12" 
                class="btn btn-outline text-sm"
              >
                12小时
              </button>
            </div>
          </div>
          <div class="mt-6 flex justify-end gap-2">
            <button 
              @click="showGoalSettings = false" 
              class="btn btn-outline"
            >
              取消
            </button>
            <button 
              @click="saveGoalSettings" 
              class="btn btn-primary"
            >
              保存
            </button>
          </div>
        </div>
      </div>
    </main>

    <!-- 底部信息 -->
    <footer class="bg-white border-t mt-12 py-6">
      <div class="container mx-auto px-4 text-center text-gray-500">
        <p>学习记录助手 © 2024</p>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from 'vue'

// 状态管理
const isStudying = ref(false)
const studyTime = ref(0)
const timerInterval = ref<number | null>(null)
const notes = ref('')
const studyTitle = ref('')
const summary = ref('')
const showHistory = ref(false)
const showEncouragement = ref(false)
const showRecordDetail = ref(false)
const showGoalSettings = ref(false)
const currentRecord = ref<any>(null)
const encouragementMessage = ref({ title: '', message: '' })
const studyRecords = ref<any[]>([])
const uploadedFiles = ref<File[]>([])
const uploadedFilesData = ref<string[]>([])

// 学习计划
const newTodo = ref('')
const todos = ref<{text: string, completed: boolean}[]>([])
const completedTodosCount = computed(() => {
  return todos.value.filter(todo => todo.completed).length
})

// 统计数据
const todayStudyTime = ref(0)
const weekStudyTime = ref(0)
const monthStudyTime = ref(0)
const targetStudyTime = ref(6) // 默认目标6小时

// 获取卡片颜色
function getCardColor(index: number): string {
  const colors = [
    'bg-blue-50 border-blue-200',
    'bg-green-50 border-green-200',
    'bg-purple-50 border-purple-200',
    'bg-pink-50 border-pink-200',
    'bg-yellow-50 border-yellow-200',
    'bg-indigo-50 border-indigo-200'
  ]
  return colors[index % colors.length]
}

// 格式化时间
function formatTime(seconds: number): string {
  const hours = Math.floor(seconds / 3600)
  const minutes = Math.floor((seconds % 3600) / 60)
  const secs = seconds % 60
  return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

// 开始学习
function startStudy() {
  isStudying.value = true
  timerInterval.value = window.setInterval(() => {
    studyTime.value++
  }, 1000)
}

// 暂停学习
function pauseStudy() {
  isStudying.value = false
  if (timerInterval.value) {
    clearInterval(timerInterval.value)
    timerInterval.value = null
  }
}

// 结束学习
function stopStudy(autoSave = true) {
  isStudying.value = false
  if (timerInterval.value) {
    clearInterval(timerInterval.value)
    timerInterval.value = null
  }
  
  // 显示鼓励提示
  showEncouragementMessage(studyTime.value)
  
  // 自动保存学习时长到统计中
  if (autoSave && studyTime.value > 0) {
    const autoRecord = {
      title: studyTitle.value || '学习记录',
      summary: summary.value || '未填写总结',
      notes: notes.value || '',
      duration: studyTime.value,
      date: new Date().toLocaleString('zh-CN')
    }
    
    studyRecords.value.unshift(autoRecord)
    localStorage.setItem('studyRecords', JSON.stringify(studyRecords.value))
    updateStudyStats()
    
    // 自动保存时清零计时器
    studyTime.value = 0
  }
  // 手动保存时不清零计时器，由保存函数处理
}

// 显示鼓励信息
function showEncouragementMessage(duration: number) {
  if (duration < 300) { // 少于5分钟
    encouragementMessage.value = {
      title: '继续加油！',
      message: '虽然这次学习时间不长，但每一步都是进步。坚持下去，你会越来越好！'
    }
  } else if (duration < 1800) { // 少于30分钟
    encouragementMessage.value = {
      title: '做得不错！',
      message: '你已经完成了一段有效的学习时间。保持这种专注的状态，继续前进！'
    }
  } else if (duration < 3600) { // 少于1小时
    encouragementMessage.value = {
      title: '非常棒！',
      message: '你已经专注学习了相当长的时间。这种坚持和努力一定会有回报的！'
    }
  } else { // 1小时以上
    encouragementMessage.value = {
      title: '超级厉害！',
      message: '你展现了惊人的专注力和毅力。这就是成功的秘诀，继续保持这种学习态度！'
    }
  }
  showEncouragement.value = true
}

// 处理文件上传
function handleFileUpload(event: Event) {
  const target = event.target as HTMLInputElement
  const files = target.files
  if (files) {
    const fileArray = Array.from(files)
    uploadedFiles.value.push(...fileArray)
    
    // 读取文件为Base64
    fileArray.forEach(file => {
      const reader = new FileReader()
      reader.onload = (e) => {
        if (e.target?.result) {
          uploadedFilesData.value.push(e.target.result as string)
        }
      }
      reader.readAsDataURL(file)
    })
  }
}

// 删除上传的文件
function removeFile(index: number) {
  uploadedFiles.value.splice(index, 1)
  uploadedFilesData.value.splice(index, 1)
}

// 保存学习记录
function saveStudyRecord() {
  if (!studyTitle.value || !summary.value) return
  
  // 如果正在学习，先停止计时（不自动保存，因为后续会手动保存）
  if (isStudying.value) {
    stopStudy(false)
  }
  
  const record = {
    title: studyTitle.value,
    summary: summary.value,
    notes: notes.value,
    duration: studyTime.value,
    date: new Date().toLocaleString('zh-CN'),
    files: uploadedFilesData.value,
    fileNames: uploadedFiles.value.map(f => f.name)
  }
  
  studyRecords.value.unshift(record)
  
  // 保存到本地存储
  localStorage.setItem('studyRecords', JSON.stringify(studyRecords.value))
  
  // 重置表单
  studyTitle.value = ''
  summary.value = ''
  notes.value = ''
  studyTime.value = 0 // 保存后清零计时器
  uploadedFiles.value = []
  uploadedFilesData.value = []
  
  // 更新统计数据
  updateStudyStats()
}

// 查看记录详情
function viewRecordDetail(record: any) {
  currentRecord.value = record
  showRecordDetail.value = true
}

// 更新学习统计
function updateStudyStats() {
  const records = JSON.parse(localStorage.getItem('studyRecords') || '[]')
  const now = new Date()
  const today = new Date(now.getFullYear(), now.getMonth(), now.getDate())
  const weekAgo = new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000)
  const monthAgo = new Date(now.getFullYear(), now.getMonth() - 1, now.getDate())
  
  todayStudyTime.value = records
    .filter((record: any) => new Date(record.date) >= today)
    .reduce((total: number, record: any) => total + record.duration, 0)
  
  weekStudyTime.value = records
    .filter((record: any) => new Date(record.date) >= weekAgo)
    .reduce((total: number, record: any) => total + record.duration, 0)
  
  monthStudyTime.value = records
    .filter((record: any) => new Date(record.date) >= monthAgo)
    .reduce((total: number, record: any) => total + record.duration, 0)
}

// 保存目标设置
function saveGoalSettings() {
  localStorage.setItem('targetStudyTime', targetStudyTime.value.toString())
  showGoalSettings.value = false
}

// 学习计划相关函数
function addTodo() {
  if (newTodo.value.trim()) {
    todos.value.push({ text: newTodo.value.trim(), completed: false })
    newTodo.value = ''
    saveTodos()
  }
}

function removeTodo(index: number) {
  todos.value.splice(index, 1)
  saveTodos()
}

function saveTodos() {
  localStorage.setItem('todos', JSON.stringify(todos.value))
}

function clearCompletedTodos() {
  todos.value = todos.value.filter(todo => !todo.completed)
  saveTodos()
}

// 生命周期钩子
onMounted(() => {
  // 从本地存储加载数据
  const savedRecords = localStorage.getItem('studyRecords')
  if (savedRecords) {
    studyRecords.value = JSON.parse(savedRecords)
  }
  
  // 加载目标学习时长
  const savedTargetTime = localStorage.getItem('targetStudyTime')
  if (savedTargetTime) {
    targetStudyTime.value = parseInt(savedTargetTime)
  }
  
  // 加载学习计划
  const savedTodos = localStorage.getItem('todos')
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos)
  }
  
  // 更新统计数据
  updateStudyStats()
})

onUnmounted(() => {
  // 清理计时器
  if (timerInterval.value) {
    clearInterval(timerInterval.value)
  }
})
</script>

<style scoped>
/* 自定义样式 */
#timer {
  font-variant-numeric: tabular-nums;
}
</style>