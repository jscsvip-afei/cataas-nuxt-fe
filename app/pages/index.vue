<template>
  <div class="min-h-screen bg-base-200">
    <!-- Hero Section -->
    <div class="hero bg-gradient-to-br from-primary to-secondary text-primary-content py-10">
      <div class="hero-content text-center">
        <div class="max-w-md">
          <h1 class="text-5xl font-bold">🐱 哈基米</h1>
          <p class="py-3">获取随机可爱猫咪图片</p>
          <button class="btn btn-primary mt-4" @click="getRandomCat" :disabled="loading">
            <span v-if="loading" class="loading loading-spinner loading-sm"></span>
            换一只
          </button>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <div class="container mx-auto px-4 py-8">
      <!-- Random Cat Section -->
      <div class="card bg-base-100 shadow-xl mb-8">
        <div class="card-body">
          <div class="flex justify-center items-center mb-4">
            <h2 class="card-title text-2xl">🎲 随机猫咪</h2>
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
// State
const loading = ref(false)
const currentCatUrl = ref('')

// Toast
const toast = reactive({
  show: false,
  message: '',
  type: 'alert-info'
})

// Methods
const getRandomCat = async () => {
  loading.value = true
  try {
    const res = await fetch('https://cataas.com/cat?json=true')
    const data = await res.json()
    const catId = data._id || data.id
    if (catId) {
      currentCatUrl.value = `https://cataas.com/cat/${catId}`
    } else {
      // Fallback
      currentCatUrl.value = `https://cataas.com/cat?t=${Date.now()}`
    }
  } catch (error) {
    console.error('Failed to fetch random cat:', error)
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

</script>
