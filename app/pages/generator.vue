<template>
  <div class="min-h-screen bg-base-200 py-8">
    <div class="container mx-auto px-4">
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-4xl font-bold mb-2">🎨 猫咪生成器</h1>
        <p class="text-base-content/60">创建自定义猫咪图片 - 支持文字叠加、滤镜、尺寸调整</p>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <!-- Controls Panel -->
        <div class="card bg-base-100 shadow-xl">
          <div class="card-body">
            <h2 class="card-title">⚙️ 生成选项</h2>

            <!-- Basic Options -->
            <div class="collapse collapse-arrow bg-base-200 mb-2">
              <input type="checkbox" checked /> 
              <div class="collapse-title font-medium">
                基础设置
              </div>
              <div class="collapse-content">
                <div class="grid grid-cols-2 gap-4">
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">图片类型</span>
                    </label>
                    <select v-model="options.type" class="select select-bordered select-sm">
                      <option value="">原始</option>
                      <option value="square">正方形</option>
                      <option value="medium">中等</option>
                      <option value="small">小图</option>
                      <option value="xsmall">超小</option>
                    </select>
                  </div>
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">格式</span>
                    </label>
                    <div class="flex gap-2">
                      <label class="label cursor-pointer gap-2">
                        <input type="radio" v-model="options.isGif" :value="false" class="radio radio-sm" />
                        <span class="label-text">图片</span>
                      </label>
                      <label class="label cursor-pointer gap-2">
                        <input type="radio" v-model="options.isGif" :value="true" class="radio radio-sm" />
                        <span class="label-text">GIF</span>
                      </label>
                    </div>
                  </div>
                </div>

                <div class="grid grid-cols-2 gap-4 mt-4">
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">宽度: {{ options.width }}px</span>
                    </label>
                    <input type="range" v-model.number="options.width" min="100" max="800" class="range range-sm" />
                  </div>
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">高度: {{ options.height }}px</span>
                    </label>
                    <input type="range" v-model.number="options.height" min="100" max="800" class="range range-sm" />
                  </div>
                </div>
              </div>
            </div>

            <!-- Tag Selection -->
            <div class="collapse collapse-arrow bg-base-200 mb-2">
              <input type="checkbox" checked /> 
              <div class="collapse-title font-medium">
                标签选择
              </div>
              <div class="collapse-content">
                <div class="form-control">
                  <label class="label">
                    <span class="label-text">选择标签 (可多选)</span>
                  </label>
                  <div class="flex flex-wrap gap-2 max-h-40 overflow-y-auto p-2 bg-base-100 rounded-lg border">
                    <label 
                      v-for="tag in availableTags" 
                      :key="tag"
                      class="cursor-pointer"
                    >
                      <input 
                        type="checkbox" 
                        :value="tag" 
                        v-model="options.tags"
                        class="hidden peer"
                      />
                      <span class="badge peer-checked:badge-primary peer-checked:text-primary-content transition-colors">
                        {{ tag }}
                      </span>
                    </label>
                  </div>
                </div>
                <div v-if="options.tags.length > 0" class="mt-2">
                  <span class="text-sm text-base-content/60">已选: </span>
                  <span class="badge badge-primary badge-sm" v-for="tag in options.tags" :key="tag">
                    {{ tag }}
                  </span>
                </div>
              </div>
            </div>

            <!-- Filters -->
            <div class="collapse collapse-arrow bg-base-200 mb-2">
              <input type="checkbox" /> 
              <div class="collapse-title font-medium">
                滤镜效果
              </div>
              <div class="collapse-content">
                <div class="grid grid-cols-3 gap-2">
                  <label 
                    v-for="filter in filters" 
                    :key="filter.value"
                    class="cursor-pointer"
                  >
                    <input 
                      type="radio" 
                      :value="filter.value" 
                      v-model="options.filter"
                      class="hidden peer"
                    />
                    <div class="card bg-base-100 peer-checked:bg-primary peer-checked:text-primary-content p-2 text-center transition-colors">
                      <span class="text-2xl">{{ filter.icon }}</span>
                      <span class="text-xs">{{ filter.label }}</span>
                    </div>
                  </label>
                </div>

                <!-- Custom Filter Options -->
                <div v-if="options.filter === 'custom'" class="mt-4 space-y-4">
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">亮度: {{ options.brightness }}</span>
                    </label>
                    <input type="range" v-model.number="options.brightness" min="-100" max="100" class="range range-sm" />
                  </div>
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">饱和度: {{ options.saturation }}</span>
                    </label>
                    <input type="range" v-model.number="options.saturation" min="-100" max="100" class="range range-sm" />
                  </div>
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">色相: {{ options.hue }}</span>
                    </label>
                    <input type="range" v-model.number="options.hue" min="0" max="360" class="range range-sm" />
                  </div>
                </div>
              </div>
            </div>

            <!-- Text Overlay -->
            <div class="collapse collapse-arrow bg-base-200 mb-2">
              <input type="checkbox" /> 
              <div class="collapse-title font-medium">
                文字叠加
              </div>
              <div class="collapse-content">
                <div class="form-control">
                  <label class="label">
                    <span class="label-text">猫咪说的话</span>
                  </label>
                  <input 
                    v-model="options.text" 
                    type="text" 
                    placeholder="Hello World!" 
                    class="input input-bordered"
                  />
                </div>
                <div class="grid grid-cols-2 gap-4 mt-4">
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">字体大小</span>
                    </label>
                    <input 
                      v-model.number="options.fontSize" 
                      type="number" 
                      min="10" 
                      max="100"
                      class="input input-bordered input-sm"
                    />
                  </div>
                  <div class="form-control">
                    <label class="label">
                      <span class="label-text">字体颜色</span>
                    </label>
                    <div class="flex gap-2 items-center">
                      <input 
                        v-model="options.fontColor" 
                        type="color" 
                        class="w-10 h-10 rounded cursor-pointer"
                      />
                      <input 
                        v-model="options.fontColor" 
                        type="text" 
                        class="input input-bordered input-sm flex-1"
                      />
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Actions -->
            <div class="card-actions justify-between mt-4">
              <button class="btn btn-outline" @click="resetOptions">
                🔄 重置
              </button>
              <div class="flex gap-2">
                <button class="btn btn-secondary" @click="randomize">
                  🎲 随机
                </button>
                <button class="btn btn-primary" @click="generate" :disabled="loading">
                  <span v-if="loading" class="loading loading-spinner loading-sm"></span>
                  ✨ 生成
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- Preview Panel -->
        <div class="space-y-4">
          <!-- Preview Card -->
          <div class="card bg-base-100 shadow-xl">
            <div class="card-body">
              <h2 class="card-title">👀 预览</h2>
              
              <div class="flex justify-center items-center min-h-80 bg-base-200 rounded-lg">
                <div v-if="loading" class="flex flex-col items-center gap-4">
                  <span class="loading loading-infinity loading-lg text-primary"></span>
                  <p>正在生成...</p>
                </div>
                <img 
                  v-else-if="previewUrl" 
                  :src="previewUrl" 
                  alt="Generated cat"
                  class="max-h-96 rounded-lg object-contain"
                  @error="onImageError"
                />
                <div v-else class="text-center text-base-content/60">
                  <p class="text-6xl mb-4">🎨</p>
                  <p>点击"生成"按钮创建猫咪图片</p>
                </div>
              </div>
            </div>
          </div>

          <!-- URL Card -->
          <div class="card bg-base-100 shadow-xl" v-if="previewUrl">
            <div class="card-body">
              <h2 class="card-title">🔗 生成的 URL</h2>
              
              <div class="mockup-code">
                <pre><code class="text-xs break-all">{{ previewUrl }}</code></pre>
              </div>

              <div class="flex flex-wrap gap-2 mt-4">
                <button class="btn btn-sm btn-primary" @click="copyUrl">
                  📋 复制 URL
                </button>
                <a :href="previewUrl" target="_blank" class="btn btn-sm btn-secondary">
                  🔗 新窗口打开
                </a>
                <button class="btn btn-sm btn-accent" @click="downloadImage">
                  ⬇️ 下载
                </button>
              </div>
            </div>
          </div>

          <!-- History -->
          <div class="card bg-base-100 shadow-xl" v-if="history.length > 0">
            <div class="card-body">
              <div class="flex justify-between items-center">
                <h2 class="card-title">📜 历史记录</h2>
                <button class="btn btn-ghost btn-xs" @click="clearHistory">清空</button>
              </div>
              
              <div class="flex gap-2 overflow-x-auto py-2">
                <div 
                  v-for="(item, index) in history" 
                  :key="index"
                  class="flex-shrink-0 cursor-pointer hover:opacity-80 transition-opacity"
                  @click="loadFromHistory(item)"
                >
                  <img 
                    :src="item" 
                    alt="History item"
                    class="w-16 h-16 object-cover rounded-lg"
                  />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface GeneratorOptions {
  type: string;
  isGif: boolean;
  width: number;
  height: number;
  tags: string[];
  filter: string;
  brightness: number;
  saturation: number;
  hue: number;
  text: string;
  fontSize: number;
  fontColor: string;
}

const filters = [
  { value: '', icon: '🖼️', label: '无' },
  { value: 'blur', icon: '🌫️', label: '模糊' },
  { value: 'mono', icon: '⬛', label: '黑白' },
  { value: 'sepia', icon: '🟤', label: '复古' },
  { value: 'negative', icon: '🔄', label: '反色' },
  { value: 'paint', icon: '🎨', label: '油画' },
  { value: 'pixel', icon: '🟩', label: '像素' },
  { value: 'custom', icon: '⚙️', label: '自定义' },
]

const options = reactive<GeneratorOptions>({
  type: '',
  isGif: false,
  width: 400,
  height: 400,
  tags: [],
  filter: '',
  brightness: 0,
  saturation: 0,
  hue: 0,
  text: '',
  fontSize: 30,
  fontColor: '#ffffff'
})

const availableTags = ref<string[]>([])
const loading = ref(false)
const previewUrl = ref('')
const history = ref<string[]>([])

const loadTags = async () => {
  try {
    const response = await fetch('https://cataas.com/api/tags')
    availableTags.value = await response.json()
  } catch (error) {
    console.error('Failed to load tags:', error)
  }
}

const buildUrl = () => {
  let url = 'https://cataas.com/cat'
  
  if (options.isGif) {
    url += '/gif'
  } else if (options.tags.length > 0) {
    url += `/${options.tags[0]}`
  }
  
  if (options.text) {
    url += `/says/${encodeURIComponent(options.text)}`
  }
  
  const params = new URLSearchParams()
  
  if (options.type) {
    params.append('type', options.type)
  }
  if (options.width) {
    params.append('width', options.width.toString())
  }
  if (options.height) {
    params.append('height', options.height.toString())
  }
  if (options.filter) {
    params.append('filter', options.filter)
    
    if (options.filter === 'custom') {
      params.append('brightness', options.brightness.toString())
      params.append('saturation', options.saturation.toString())
      params.append('hue', options.hue.toString())
    }
  }
  if (options.text) {
    params.append('fontSize', options.fontSize.toString())
    params.append('fontColor', options.fontColor.replace('#', ''))
  }
  
  params.append('t', Date.now().toString())
  
  return `${url}?${params.toString()}`
}

const generate = () => {
  loading.value = true
  const url = buildUrl()
  previewUrl.value = url
  
  // Add to history
  if (!history.value.includes(url)) {
    history.value.unshift(url)
    if (history.value.length > 10) {
      history.value.pop()
    }
  }
}

const onImageError = () => {
  loading.value = false
}

// Watch for image load
watch(previewUrl, () => {
  if (previewUrl.value) {
    const img = new Image()
    img.onload = () => {
      loading.value = false
    }
    img.onerror = () => {
      loading.value = false
    }
    img.src = previewUrl.value
  }
})

const resetOptions = () => {
  options.type = ''
  options.isGif = false
  options.width = 400
  options.height = 400
  options.tags = []
  options.filter = ''
  options.brightness = 0
  options.saturation = 0
  options.hue = 0
  options.text = ''
  options.fontSize = 30
  options.fontColor = '#ffffff'
}

const randomize = () => {
  options.isGif = Math.random() > 0.7
  options.width = Math.floor(Math.random() * 400) + 200
  options.height = Math.floor(Math.random() * 400) + 200
  options.filter = filters[Math.floor(Math.random() * (filters.length - 1))].value
  
  if (availableTags.value.length > 0) {
    const randomTag = availableTags.value[Math.floor(Math.random() * availableTags.value.length)]
    options.tags = [randomTag]
  }
  
  generate()
}

const copyUrl = async () => {
  try {
    await navigator.clipboard.writeText(previewUrl.value)
    alert('URL 已复制到剪贴板')
  } catch {
    alert('复制失败')
  }
}

const downloadImage = () => {
  const link = document.createElement('a')
  link.href = previewUrl.value
  link.download = `cat-${Date.now()}.${options.isGif ? 'gif' : 'jpg'}`
  link.target = '_blank'
  link.click()
}

const loadFromHistory = (url: string) => {
  previewUrl.value = url
}

const clearHistory = () => {
  history.value = []
}

onMounted(() => {
  loadTags()
})
</script>
