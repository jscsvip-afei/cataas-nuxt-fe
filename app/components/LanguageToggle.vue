<template>
  <div class="dropdown dropdown-end">
    <div tabindex="0" role="button" class="btn btn-ghost">
      <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5h12M9 3v2m1.048 9.5A18.022 18.022 0 016.412 9m6.088 9h7M11 21l5-10 5 10M12.751 5C11.783 10.77 8.07 15.61 3 18.129" />
      </svg>
      <span class="hidden sm:inline">{{ t('language') }}</span>
    </div>
    <ul tabindex="0" class="dropdown-content z-[1] p-2 shadow-2xl bg-base-200 rounded-box w-32">
      <li>
        <button 
          class="btn btn-sm btn-ghost justify-start" 
          :class="{ 'btn-active': locale === 'zh' }"
          @click="setLocale('zh')"
        >
          🇨🇳 中文
        </button>
      </li>
      <li>
        <button 
          class="btn btn-sm btn-ghost justify-start" 
          :class="{ 'btn-active': locale === 'en' }"
          @click="setLocale('en')"
        >
          🇺🇸 English
        </button>
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
const { locale, setLocale, t } = useLanguage()
</script>
