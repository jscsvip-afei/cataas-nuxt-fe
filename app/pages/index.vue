<template>
  <div class="min-h-screen bg-base-200">
    <!-- Hero Section -->
    <div class="hero bg-gradient-to-br from-primary to-secondary text-primary-content py-10">
      <div class="hero-content text-center">
        <div class="max-w-md">
          <h1 class="text-5xl font-bold">获取随机猫咪图片</h1>
          <p class="py-2 text-lg" v-if="totalCats">
            🐱 已收录 <span class="font-bold text-2xl">{{ totalCats.toLocaleString() }}</span> 只猫咪
          </p>
          <button class="btn btn-primary mt-4" @click="getRandomCat" :disabled="loading">
            <span v-if="loading" class="loading loading-spinner loading-sm"></span>
            给我一只哈基米
          </button>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="container mx-auto px-4 py-8">
      <!-- Quick Tag Cats -->
      <div class="card bg-base-100 shadow-xl mb-8">
        <div class="card-body">
          <h2 class="card-title text-xl mb-4">🏷️ 按标签获取猫咪</h2>
          <div class="flex flex-wrap gap-2">
            <button 
              v-for="tag in quickTags" 
              :key="tag"
              class="btn btn-sm btn-outline"
              :class="{ 'btn-primary': selectedTag === tag }"
              @click="getRandomCatByTag(tag)"
              :disabled="loading"
            >
              {{ tag }}
            </button>
          </div>
        </div>
      </div>

      <!-- Random Cat Section -->
      <div class="card bg-base-100 shadow-xl mb-8">
        <div class="card-body">
          <div class="flex justify-center items-center mb-4">
            <h2 class="card-title text-2xl">🎲 随机猫咪</h2>
            <span v-if="selectedTag" class="badge badge-primary ml-2">{{ selectedTag }}</span>
          </div>
          
          <div class="flex justify-center items-center min-h-80 bg-base-200 rounded-lg p-4 relative">
            <span v-if="loading" class="loading loading-spinner loading-lg text-primary absolute"></span>
            <img 
              v-show="currentCatUrl"
              :src="currentCatUrl" 
              alt="Random Cat" 
              class="rounded-lg max-h-96 object-contain"
              :class="{ 'opacity-30': loading }"
              @load="onImageLoad"
              @error="onImageError"
            />
            <div v-if="!currentCatUrl && !loading" class="text-center text-base-content/60">
              <p class="text-6xl mb-4">🐱</p>
              <p>点击按钮获取随机猫咪</p>
            </div>
          </div>

          <!-- Cat Info -->
          <div v-if="currentCatInfo" class="mt-4">
            <div class="flex flex-wrap gap-2 justify-center">
              <span 
                v-for="tag in currentCatInfo.tags" 
                :key="tag"
                class="badge badge-outline cursor-pointer hover:badge-primary"
                @click="getRandomCatByTag(tag)"
              >
                {{ tag }}
              </span>
            </div>
          </div>

          <div class="flex justify-center gap-2 mt-2" v-if="currentCatUrl">
            <button class="btn btn-sm btn-ghost" @click="copyUrl">
              📋 复制链接
            </button>
            <a :href="currentCatUrl" target="_blank" class="btn btn-sm btn-ghost">
              🔗 新窗口打开
            </a>
          </div>
        </div>
      </div>

      <!-- API Stats -->
      <div class="stats shadow w-full bg-base-100">
        <div class="stat">
          <div class="stat-figure text-primary">
            <span class="text-3xl">🐱</span>
          </div>
          <div class="stat-title">总猫咪数</div>
          <div class="stat-value text-primary">{{ totalCats?.toLocaleString() || '-' }}</div>
          <div class="stat-desc">来自 cataas.com</div>
        </div>
        
        <div class="stat">
          <div class="stat-figure text-secondary">
            <span class="text-3xl">🏷️</span>
          </div>
          <div class="stat-title">标签数量</div>
          <div class="stat-value text-secondary">{{ totalTags || '-' }}</div>
          <div class="stat-desc">可用于筛选</div>
        </div>
        
        <div class="stat">
          <div class="stat-figure text-accent">
            <span class="text-3xl">🎨</span>
          </div>
          <div class="stat-title">API 端点</div>
          <div class="stat-value text-accent">9</div>
          <div class="stat-desc">完整功能支持</div>
        </div>
      </div>
    </div>

    <!-- Toast -->
    <div class="toast toast-end">
      <div v-if="toast.show" class="alert" :class="toast.type">
        <span>{{ toast.message }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
interface CatInfo {
  _id?: string
  id?: string
  tags: string[]
  mimetype?: string
}

// State
const loading = ref(false)
const currentCatUrl = ref('')
const currentCatInfo = ref<CatInfo | null>(null)
const totalCats = ref<number | null>(null)
const totalTags = ref<number | null>(null)
const selectedTag = ref('')

const quickTags = ['cute', 'orange', 'black', 'white', 'gif', 'sleepy', 'fluffy', 'funny']

// Toast
const toast = reactive({
  show: false,
  message: '',
  type: 'alert-info'
})

// Methods
const loadStats = async () => {
  try {
    // GET /api/count
    const countRes = await fetch('https://cataas.com/api/count')
    const countData = await countRes.json()
    totalCats.value = countData.count || countData

    // GET /api/tags
    const tagsRes = await fetch('https://cataas.com/api/tags')
    const tagsData = await tagsRes.json()
    totalTags.value = Array.isArray(tagsData) ? tagsData.length : 0
  } catch (error) {
    console.error('Failed to load stats:', error)
  }
}

const getRandomCat = async () => {
  selectedTag.value = ''
  loading.value = true
  try {
    // GET /cat?json=true
    const res = await fetch('https://cataas.com/cat?json=true')
    const data = await res.json()
    currentCatInfo.value = data
    const catId = data._id || data.id
    if (catId) {
      // GET /cat/{id}
      currentCatUrl.value = `https://cataas.com/cat/${catId}`
    } else {
      currentCatUrl.value = `https://cataas.com/cat?t=${Date.now()}`
    }
  } catch (error) {
    console.error('Failed to fetch random cat:', error)
    showToast('获取猫咪失败，请重试', 'alert-error')
    loading.value = false
  }
}

const getRandomCatByTag = async (tag: string) => {
  selectedTag.value = tag
  loading.value = true
  try {
    // GET /cat/{tag}?json=true
    const res = await fetch(`https://cataas.com/cat/${tag}?json=true`)
    const data = await res.json()
    currentCatInfo.value = data
    const catId = data._id || data.id
    if (catId) {
      currentCatUrl.value = `https://cataas.com/cat/${catId}`
    } else {
      // Fallback: GET /cat/{tag}
      currentCatUrl.value = `https://cataas.com/cat/${tag}?t=${Date.now()}`
    }
  } catch (error) {
    console.error('Failed to fetch cat by tag:', error)
    showToast('获取猫咪失败，请重试', 'alert-error')
    loading.value = false
  }
}

const onImageLoad = () => {
  loading.value = false
}

const onImageError = () => {
  loading.value = false
  showToast('图片加载失败，请重试', 'alert-error')
}

const copyUrl = async () => {
  try {
    await navigator.clipboard.writeText(currentCatUrl.value)
    showToast('链接已复制', 'alert-success')
  } catch {
    showToast('复制失败', 'alert-error')
  }
}

const showToast = (message: string, type = 'alert-info') => {
  toast.message = message
  toast.type = type
  toast.show = true
  setTimeout(() => {
    toast.show = false
  }, 2000)
}

onMounted(() => {
  loadStats()
  getRandomCat()
})
</script>

