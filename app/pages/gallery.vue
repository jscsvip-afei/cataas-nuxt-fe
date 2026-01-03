<template>
  <div class="min-h-screen bg-base-200 py-8">
    <div class="container mx-auto px-4">
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-4xl font-bold mb-2">📸 猫咪图库</h1>
        <p class="text-base-content/60">浏览所有可爱的猫咪</p>
      </div>

      <!-- Filters -->
      <div class="card bg-base-100 shadow-xl mb-8">
        <div class="card-body">
          <h2 class="card-title">🔍 筛选条件</h2>
          <div class="flex flex-wrap gap-4 items-end">
            <div class="form-control flex-1 min-w-48">
              <label class="label">
                <span class="label-text">标签筛选</span>
              </label>
              <select class="select select-bordered w-full" @change="onTagSelect">
                <option value="" disabled selected>选择标签...</option>
                <option v-for="tag in availableTags" :key="tag" :value="tag">{{ tag }}</option>
              </select>
            </div>
            <div class="form-control">
              <label class="label">
                <span class="label-text">每页数量</span>
              </label>
              <select v-model="pageSize" class="select select-bordered">
                <option :value="12">12</option>
                <option :value="24">24</option>
                <option :value="48">48</option>
              </select>
            </div>
            <div class="flex gap-2">
              <button class="btn btn-primary" @click="search">
                搜索
              </button>
              <button class="btn btn-outline" @click="resetFilters">
                重置
              </button>
            </div>
          </div>
          
          <!-- Selected Tags -->
          <div v-if="selectedTags.length > 0" class="flex flex-wrap gap-2 mt-4">
            <span class="text-sm text-base-content/60">已选标签：</span>
            <span 
              v-for="tag in selectedTags" 
              :key="tag"
              class="badge badge-primary gap-1"
            >
              {{ tag }}
              <button @click="removeTag(tag)" class="btn btn-ghost btn-xs p-0">✕</button>
            </span>
          </div>
        </div>
      </div>

      <!-- Loading -->
      <div v-if="loading" class="flex justify-center py-20">
        <div class="flex flex-col items-center gap-4">
          <span class="loading loading-spinner loading-lg text-primary"></span>
          <p>正在加载猫咪...</p>
        </div>
      </div>

      <!-- Gallery Grid -->
      <div v-else class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
        <div 
          v-for="cat in cats" 
          :key="getCatId(cat)"
          class="card bg-base-100 shadow-lg hover:shadow-2xl transition-all duration-300 hover:-translate-y-1 cursor-pointer group"
          @click="openModal(cat)"
        >
          <figure class="relative overflow-hidden">
            <img 
              :src="`https://cataas.com/cat/${getCatId(cat)}?width=200&height=200`" 
              :alt="cat.tags?.join(', ') || 'Cat'"
              class="w-full h-40 object-cover group-hover:scale-110 transition-transform duration-300"
              loading="lazy"
            />
            <div class="absolute inset-0 bg-black/50 opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex items-center justify-center">
              <span class="text-white text-2xl">🔍</span>
            </div>
          </figure>
          <div class="card-body p-3">
            <div class="flex flex-wrap gap-1">
              <span 
                v-for="tag in (cat.tags || []).slice(0, 2)" 
                :key="tag"
                class="badge badge-xs badge-outline"
              >
                {{ tag }}
              </span>
              <span v-if="(cat.tags || []).length > 2" class="badge badge-xs badge-ghost">
                +{{ cat.tags.length - 2 }}
              </span>
            </div>
          </div>
        </div>
      </div>

      <!-- Empty State -->
      <div v-if="!loading && cats.length === 0" class="text-center py-20">
        <p class="text-6xl mb-4">😿</p>
        <p class="text-xl">没有找到猫咪</p>
        <p class="text-base-content/60">尝试更改筛选条件</p>
      </div>

      <!-- Pagination -->
      <div class="flex justify-center mt-8">
        <div class="join">
          <button 
            class="join-item btn" 
            :disabled="currentPage === 0"
            @click="goToPage(0)"
          >
            ««
          </button>
          <button 
            class="join-item btn" 
            :disabled="currentPage === 0"
            @click="goToPage(currentPage - 1)"
          >
            «
          </button>
          <button class="join-item btn btn-active">
            {{ currentPage + 1 }}
          </button>
          <button 
            class="join-item btn"
            :disabled="cats.length < pageSize"
            @click="goToPage(currentPage + 1)"
          >
            »
          </button>
        </div>
      </div>

      <!-- Modal -->
      <dialog ref="modalRef" class="modal">
        <div class="modal-box max-w-4xl">
          <form method="dialog">
            <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
          </form>
          
          <div v-if="selectedCat" class="flex flex-col items-center">
            <div class="relative min-h-48 flex justify-center items-center w-full mb-4">
              <span v-if="modalImageLoading" class="loading loading-spinner loading-lg text-primary absolute"></span>
              <img 
                v-show="!modalImageLoading"
                :src="`https://cataas.com/cat/${getCatId(selectedCat)}`" 
                :alt="selectedCat.tags?.join(', ')"
                class="max-h-96 object-contain rounded-lg"
                @load="modalImageLoading = false"
                @error="modalImageLoading = false"
              />
            </div>
            
            <div class="w-full">
              <h3 class="font-bold text-lg mb-2">猫咪详情</h3>
              
              <div class="flex flex-wrap gap-2 mb-4">
                <span 
                  v-for="tag in selectedCat.tags" 
                  :key="tag"
                  class="badge badge-primary"
                >
                  {{ tag }}
                </span>
              </div>

              <div class="grid grid-cols-2 gap-4 text-sm">
                <div>
                  <span class="text-base-content/60">ID:</span>
                  <span class="ml-2 font-mono">{{ getCatId(selectedCat) }}</span>
                </div>
                <div v-if="selectedCat.mimetype">
                  <span class="text-base-content/60">类型:</span>
                  <span class="ml-2">{{ selectedCat.mimetype }}</span>
                </div>
              </div>

              <div class="divider"></div>

              <div class="flex flex-wrap gap-2">
                <a 
                  :href="`https://cataas.com/cat/${getCatId(selectedCat)}`" 
                  target="_blank" 
                  class="btn btn-primary btn-sm"
                >
                  🔗 原图链接
                </a>
                <button class="btn btn-secondary btn-sm" @click="copyImageUrl">
                  📋 复制链接
                </button>
                <a 
                  :href="`https://cataas.com/cat/${getCatId(selectedCat)}`" 
                  download
                  class="btn btn-accent btn-sm"
                >
                  ⬇️ 下载
                </a>
              </div>
            </div>
          </div>
        </div>
        <form method="dialog" class="modal-backdrop">
          <button>close</button>
        </form>
      </dialog>
    </div>
  </div>
</template>

<script setup lang="ts">
interface Cat {
  _id?: string;
  id?: string;
  tags: string[];
  mimetype?: string;
}

const getCatId = (cat: Cat) => cat._id || cat.id

const cats = ref<Cat[]>([])
const availableTags = ref<string[]>([])
const selectedTags = ref<string[]>([])
const loading = ref(false)
const currentPage = ref(0)
const pageSize = ref(24)
const selectedCat = ref<Cat | null>(null)
const modalRef = ref<HTMLDialogElement | null>(null)
const modalImageLoading = ref(false)

const loadTags = async () => {
  try {
    const response = await fetch('https://cataas.com/api/tags')
    availableTags.value = await response.json()
  } catch (error) {
    console.error('Failed to load tags:', error)
  }
}

const loadCats = async () => {
  loading.value = true
  try {
    const params = new URLSearchParams({
      limit: pageSize.value.toString(),
      skip: (currentPage.value * pageSize.value).toString()
    })
    
    if (selectedTags.value.length > 0) {
      params.append('tags', selectedTags.value.join(','))
    }
    
    const response = await fetch(`https://cataas.com/api/cats?${params}`)
    cats.value = await response.json()
  } catch (error) {
    console.error('Failed to load cats:', error)
  } finally {
    loading.value = false
  }
}

const search = () => {
  currentPage.value = 0
  loadCats()
}

const resetFilters = () => {
  selectedTags.value = []
  currentPage.value = 0
  loadCats()
}

const onTagSelect = (event: Event) => {
  const select = event.target as HTMLSelectElement
  const tag = select.value
  if (tag && !selectedTags.value.includes(tag)) {
    selectedTags.value.push(tag)
  }
  select.value = ''
}

const removeTag = (tag: string) => {
  selectedTags.value = selectedTags.value.filter(t => t !== tag)
}

const goToPage = (page: number) => {
  currentPage.value = page
  loadCats()
}

const openModal = (cat: Cat) => {
  selectedCat.value = null // Clear previous cat to ensure clean state
  modalImageLoading.value = true
  // Use nextTick to ensure DOM updates before setting new cat
  nextTick(() => {
    selectedCat.value = cat
    modalRef.value?.showModal()
  })
}

const copyImageUrl = async () => {
  if (selectedCat.value) {
    try {
      await navigator.clipboard.writeText(`https://cataas.com/cat/${getCatId(selectedCat.value)}`)
      alert('链接已复制')
    } catch {
      alert('复制失败')
    }
  }
}

onMounted(() => {
  loadTags()
  loadCats()
})

watch(pageSize, () => {
  currentPage.value = 0
  loadCats()
})
</script>
