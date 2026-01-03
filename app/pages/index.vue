<template>
  <div class="min-h-screen bg-base-200">
    <!-- Hero Section -->
    <div class="hero bg-gradient-to-br from-primary to-secondary text-primary-content py-16">
      <div class="hero-content text-center">
        <div class="max-w-md">
          <h1 class="text-5xl font-bold">🐱 CATAAS</h1>
          <p class="py-6">Cat as a Service - 获取随机可爱猫咪图片的神奇应用</p>
          <button class="btn btn-accent" @click="getRandomCat">
            获取随机猫咪 🎲
          </button>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="container mx-auto px-4 py-8">
      <!-- Stats -->
      <div class="stats shadow w-full mb-8 bg-base-100">
        <div class="stat">
          <div class="stat-figure text-primary">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="inline-block w-8 h-8 stroke-current">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"></path>
            </svg>
          </div>
          <div class="stat-title">可用猫咪</div>
          <div class="stat-value text-primary">1987+</div>
          <div class="stat-desc">来自 CATAAS API</div>
        </div>
        <div class="stat">
          <div class="stat-figure text-secondary">
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="inline-block w-8 h-8 stroke-current">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path>
            </svg>
          </div>
          <div class="stat-title">已加载标签</div>
          <div class="stat-value text-secondary">{{ tags.length }}</div>
          <div class="stat-desc">点击下方标签筛选</div>
        </div>
      </div>

      <!-- Controls -->
      <div class="card bg-base-100 shadow-xl mb-8">
        <div class="card-body">
          <h2 class="card-title">🎨 自定义选项</h2>
          
          <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
            <!-- Tag Select -->
            <div class="form-control">
              <label class="label">
                <span class="label-text">选择标签</span>
              </label>
              <select v-model="selectedTag" class="select select-bordered w-full">
                <option value="">全部</option>
                <option v-for="tag in tags" :key="tag" :value="tag">{{ tag }}</option>
              </select>
            </div>

            <!-- Filter Select -->
            <div class="form-control">
              <label class="label">
                <span class="label-text">滤镜效果</span>
              </label>
              <select v-model="selectedFilter" class="select select-bordered w-full">
                <option value="">无滤镜</option>
                <option value="blur">模糊</option>
                <option value="mono">黑白</option>
                <option value="sepia">复古</option>
                <option value="negative">反色</option>
                <option value="paint">油画</option>
                <option value="pixel">像素化</option>
              </select>
            </div>

            <!-- Width Input -->
            <div class="form-control">
              <label class="label">
                <span class="label-text">宽度 (px)</span>
              </label>
              <input 
                v-model.number="imageWidth" 
                type="number" 
                placeholder="400" 
                class="input input-bordered w-full" 
                min="100"
                max="1000"
              />
            </div>

            <!-- Type Select -->
            <div class="form-control">
              <label class="label">
                <span class="label-text">图片类型</span>
              </label>
              <select v-model="imageType" class="select select-bordered w-full">
                <option value="">原始</option>
                <option value="square">正方形</option>
                <option value="medium">中等</option>
                <option value="small">小图</option>
                <option value="xsmall">超小</option>
              </select>
            </div>
          </div>

          <!-- Text Input for Cat Says -->
          <div class="form-control mt-4">
            <label class="label">
              <span class="label-text">让猫咪说话 🗣️</span>
            </label>
            <div class="join w-full">
              <input 
                v-model="catText" 
                type="text" 
                placeholder="输入猫咪要说的话..." 
                class="input input-bordered join-item flex-1"
              />
              <select v-model="fontSize" class="select select-bordered join-item">
                <option value="20">小字</option>
                <option value="30">中字</option>
                <option value="50">大字</option>
              </select>
              <input 
                v-model="fontColor" 
                type="color" 
                class="join-item w-12 h-12 cursor-pointer"
                title="字体颜色"
              />
            </div>
          </div>

          <div class="card-actions justify-end mt-4">
            <button class="btn btn-outline" @click="resetOptions">重置</button>
            <button class="btn btn-primary" @click="getRandomCat">
              <span v-if="loading" class="loading loading-spinner"></span>
              获取猫咪
            </button>
            <button class="btn btn-secondary" @click="getGifCat">
              获取 GIF 🎬
            </button>
          </div>
        </div>
      </div>

      <!-- Current Cat Display -->
      <div class="card bg-base-100 shadow-xl mb-8">
        <div class="card-body">
          <h2 class="card-title">
            🖼️ 当前猫咪
            <div class="badge badge-secondary" v-if="isGif">GIF</div>
          </h2>
          
          <div class="flex justify-center items-center min-h-64 bg-base-200 rounded-lg p-4">
            <div v-if="loading" class="flex flex-col items-center gap-4">
              <span class="loading loading-dots loading-lg text-primary"></span>
              <p class="text-base-content/60">正在寻找可爱猫咪...</p>
            </div>
            <figure v-else-if="currentCatUrl" class="relative">
              <img 
                :src="currentCatUrl" 
                :alt="'Cat image'" 
                class="rounded-lg max-h-96 object-contain"
                @load="onImageLoad"
                @error="onImageError"
              />
            </figure>
            <div v-else class="text-center text-base-content/60">
              <p class="text-6xl mb-4">🐱</p>
              <p>点击上方按钮获取猫咪图片</p>
            </div>
          </div>

          <div class="card-actions justify-center mt-4" v-if="currentCatUrl">
            <button class="btn btn-sm btn-ghost" @click="copyUrl">
              📋 复制链接
            </button>
            <a :href="currentCatUrl" target="_blank" class="btn btn-sm btn-ghost">
              🔗 新窗口打开
            </a>
            <button class="btn btn-sm btn-ghost" @click="downloadCat">
              ⬇️ 下载
            </button>
          </div>
        </div>
      </div>

      <!-- Tags Cloud -->
      <div class="card bg-base-100 shadow-xl mb-8">
        <div class="card-body">
          <h2 class="card-title">🏷️ 猫咪标签</h2>
          <p class="text-base-content/60 mb-4">点击标签快速筛选</p>
          
          <div v-if="tagsLoading" class="flex justify-center py-8">
            <span class="loading loading-ring loading-lg"></span>
          </div>
          <div v-else class="flex flex-wrap gap-2">
            <button 
              v-for="tag in tags" 
              :key="tag"
              class="btn btn-sm"
              :class="selectedTag === tag ? 'btn-primary' : 'btn-outline'"
              @click="selectTag(tag)"
            >
              {{ tag }}
            </button>
          </div>
        </div>
      </div>

      <!-- Gallery -->
      <div class="card bg-base-100 shadow-xl">
        <div class="card-body">
          <div class="flex justify-between items-center">
            <h2 class="card-title">📸 猫咪图库</h2>
            <button class="btn btn-sm btn-primary" @click="loadGallery">
              <span v-if="galleryLoading" class="loading loading-spinner loading-sm"></span>
              刷新图库
            </button>
          </div>
          
          <div v-if="galleryLoading" class="flex justify-center py-12">
            <span class="loading loading-spinner loading-lg text-primary"></span>
          </div>
          <div v-else class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 mt-4">
            <div 
              v-for="(cat, index) in galleryCats" 
              :key="getCatId(cat) || index"
              class="card bg-base-200 shadow hover:shadow-lg transition-shadow cursor-pointer"
              @click="selectGalleryCat(cat)"
            >
              <figure class="px-2 pt-2">
                <img 
                  :src="`https://cataas.com/cat/${getCatId(cat)}?width=200`" 
                  :alt="cat.tags?.join(', ') || 'Cat'"
                  class="rounded-lg h-32 w-full object-cover"
                  loading="lazy"
                />
              </figure>
              <div class="card-body p-2">
                <div class="flex flex-wrap gap-1">
                  <span 
                    v-for="tag in (cat.tags || []).slice(0, 3)" 
                    :key="tag"
                    class="badge badge-xs badge-ghost"
                  >
                    {{ tag }}
                  </span>
                </div>
              </div>
            </div>
          </div>

          <!-- Pagination -->
          <div class="flex justify-center mt-6">
            <div class="join">
              <button 
                class="join-item btn" 
                :disabled="currentPage === 0"
                @click="prevPage"
              >
                «
              </button>
              <button class="join-item btn">第 {{ currentPage + 1 }} 页</button>
              <button 
                class="join-item btn"
                @click="nextPage"
              >
                »
              </button>
            </div>
          </div>
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
interface Cat {
  _id?: string;
  id?: string;
  tags: string[];
  mimetype?: string;
  size?: number;
  createdAt?: string;
}

// State
const loading = ref(false)
const tagsLoading = ref(false)
const galleryLoading = ref(false)
const currentCatUrl = ref('')
const isGif = ref(false)
const tags = ref<string[]>([])
const galleryCats = ref<Cat[]>([])
const currentPage = ref(0)
const pageSize = 12

// Options
const selectedTag = ref('')
const selectedFilter = ref('')
const imageWidth = ref(400)
const imageType = ref('')
const catText = ref('')
const fontSize = ref('30')
const fontColor = ref('#ffffff')

// Toast
const toast = reactive({
  show: false,
  message: '',
  type: 'alert-info'
})

// Build URL
const buildCatUrl = (isGifRequest = false) => {
  let url = 'https://cataas.com/cat'
  
  if (isGifRequest) {
    url += '/gif'
  } else if (selectedTag.value) {
    url += `/${selectedTag.value}`
  }
  
  if (catText.value) {
    url += `/says/${encodeURIComponent(catText.value)}`
  }
  
  const params = new URLSearchParams()
  
  if (selectedFilter.value) {
    params.append('filter', selectedFilter.value)
  }
  if (imageWidth.value) {
    params.append('width', imageWidth.value.toString())
  }
  if (imageType.value) {
    params.append('type', imageType.value)
  }
  if (catText.value) {
    params.append('fontSize', fontSize.value)
    params.append('fontColor', fontColor.value.replace('#', ''))
  }
  
  // Add timestamp to prevent caching
  params.append('t', Date.now().toString())
  
  const queryString = params.toString()
  return queryString ? `${url}?${queryString}` : url
}

// Methods
const getRandomCat = () => {
  loading.value = true
  isGif.value = false
  currentCatUrl.value = buildCatUrl(false)
}

const getGifCat = () => {
  loading.value = true
  isGif.value = true
  currentCatUrl.value = buildCatUrl(true)
}

const onImageLoad = () => {
  loading.value = false
}

const onImageError = () => {
  loading.value = false
  showToast('图片加载失败，请重试', 'alert-error')
}

const selectTag = (tag: string) => {
  selectedTag.value = selectedTag.value === tag ? '' : tag
  getRandomCat()
}

const resetOptions = () => {
  selectedTag.value = ''
  selectedFilter.value = ''
  imageWidth.value = 400
  imageType.value = ''
  catText.value = ''
  fontSize.value = '30'
  fontColor.value = '#ffffff'
}

const copyUrl = async () => {
  try {
    await navigator.clipboard.writeText(currentCatUrl.value)
    showToast('链接已复制到剪贴板', 'alert-success')
  } catch {
    showToast('复制失败', 'alert-error')
  }
}

const downloadCat = () => {
  const link = document.createElement('a')
  link.href = currentCatUrl.value
  link.download = `cat-${Date.now()}.${isGif.value ? 'gif' : 'jpg'}`
  link.target = '_blank'
  link.click()
}

const showToast = (message: string, type = 'alert-info') => {
  toast.message = message
  toast.type = type
  toast.show = true
  setTimeout(() => {
    toast.show = false
  }, 3000)
}

// Load tags
const loadTags = async () => {
  tagsLoading.value = true
  try {
    const response = await fetch('https://cataas.com/api/tags')
    tags.value = await response.json()
  } catch (error) {
    console.error('Failed to load tags:', error)
    showToast('加载标签失败', 'alert-error')
  } finally {
    tagsLoading.value = false
  }
}

// Load gallery
const loadGallery = async () => {
  galleryLoading.value = true
  try {
    const params = new URLSearchParams({
      limit: pageSize.toString(),
      skip: (currentPage.value * pageSize).toString()
    })
    if (selectedTag.value) {
      params.append('tags', selectedTag.value)
    }
    
    const response = await fetch(`https://cataas.com/api/cats?${params}`)
    galleryCats.value = await response.json()
  } catch (error) {
    console.error('Failed to load gallery:', error)
    showToast('加载图库失败', 'alert-error')
  } finally {
    galleryLoading.value = false
  }
}

const getCatId = (cat: Cat) => cat._id || cat.id

const selectGalleryCat = (cat: Cat) => {
  currentCatUrl.value = `https://cataas.com/cat/${getCatId(cat)}?width=${imageWidth.value}&t=${Date.now()}`
  isGif.value = cat.mimetype?.includes('gif') || false
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const prevPage = () => {
  if (currentPage.value > 0) {
    currentPage.value--
    loadGallery()
  }
}

const nextPage = () => {
  currentPage.value++
  loadGallery()
}

// Initialize
onMounted(() => {
  loadTags()
  loadGallery()
})

// Watch for tag changes
watch(selectedTag, () => {
  currentPage.value = 0
  loadGallery()
})
</script>
