<template>
  <div class="min-h-screen flex flex-col">
    <!-- Navbar -->
    <div class="navbar bg-base-100 shadow-lg sticky top-0 z-50 px-4">
      <!-- Logo -->
      <div class="flex-1">
        <NuxtLink to="/" class="btn btn-ghost text-xl">
          🐱 {{ t('appTitle') }}
        </NuxtLink>
      </div>
      
      <!-- Desktop Menu -->
      <div class="flex-none hidden md:block">
        <ul class="menu menu-horizontal gap-1">
          <li><NuxtLink to="/" class="btn btn-ghost btn-sm">{{ t('home') }}</NuxtLink></li>
          <li><NuxtLink to="/gallery" class="btn btn-ghost btn-sm">{{ t('gallery') }}</NuxtLink></li>
          <li><NuxtLink to="/generator" class="btn btn-ghost btn-sm">{{ t('generator') }}</NuxtLink></li>
        </ul>
      </div>
      
      <!-- Language Toggle -->
      <div class="flex-none">
        <LanguageToggle />
      </div>

      <!-- Theme Toggle -->
      <div class="flex-none">
        <ThemeToggle />
      </div>
      
      <!-- Mobile Menu -->
      <div class="flex-none md:hidden">
        <div class="dropdown dropdown-end">
          <div tabindex="0" role="button" class="btn btn-ghost">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h8m-8 6h16" />
            </svg>
          </div>
          <ul tabindex="0" class="menu menu-sm dropdown-content mt-3 z-[1] p-2 shadow bg-base-100 rounded-box w-40">
            <li><NuxtLink to="/">{{ t('home') }}</NuxtLink></li>
            <li><NuxtLink to="/gallery">{{ t('gallery') }}</NuxtLink></li>
            <li><NuxtLink to="/generator">{{ t('generator') }}</NuxtLink></li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Main Content -->
    <main class="flex-1">
      <slot />
    </main>

    <!-- Footer -->
    <footer class="footer footer-center p-10 bg-base-200 text-base-content">
      <aside>
        <p>{{ t('footerText') }}</p>
        <p>{{ t('copyright') }}</p>
      </aside>
    </footer>
  </div>
</template>

<script setup lang="ts">
const { t } = useLanguage()
</script>
