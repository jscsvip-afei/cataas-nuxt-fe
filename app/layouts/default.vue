<template>
  <div class="min-h-screen flex flex-col">
    <!-- Navbar -->
    <div class="navbar bg-base-100 shadow-lg sticky top-0 z-50">
      <div class="navbar-start">
        <div class="dropdown">
          <div tabindex="0" role="button" class="btn btn-ghost lg:hidden">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h8m-8 6h16" />
            </svg>
          </div>
          <ul tabindex="0" class="menu menu-sm dropdown-content mt-3 z-[1] p-2 shadow bg-base-100 rounded-box w-52">
            <li><NuxtLink to="/">首页</NuxtLink></li>
            <li><NuxtLink to="/gallery">图库</NuxtLink></li>
            <li><NuxtLink to="/generator">生成器</NuxtLink></li>
          </ul>
        </div>
        <NuxtLink to="/" class="btn btn-ghost text-xl">
          🐱 CATAAS
        </NuxtLink>
      </div>
      <div class="navbar-center hidden lg:flex">
        <ul class="menu menu-horizontal px-1">
          <li><NuxtLink to="/" class="btn btn-ghost">首页</NuxtLink></li>
          <li><NuxtLink to="/gallery" class="btn btn-ghost">图库</NuxtLink></li>
          <li><NuxtLink to="/generator" class="btn btn-ghost">生成器</NuxtLink></li>
        </ul>
      </div>
      <div class="navbar-end">
        <ThemeToggle />
      </div>
    </div>

    <!-- Main Content -->
    <main class="flex-1">
      <slot />
    </main>

    <!-- Footer -->
    <footer class="footer footer-center p-10 bg-base-200 text-base-content">
      <aside>
        <p class="text-4xl">🐱</p>
        <p class="font-bold">
          CATAAS - Cat as a Service
        </p>
        <p>基于 <a href="https://cataas.com" target="_blank" class="link link-primary">CATAAS API</a> 构建</p>
        <p>使用 <a href="https://nuxt.com" target="_blank" class="link link-secondary">Nuxt</a> + <a href="https://daisyui.com" target="_blank" class="link link-accent">DaisyUI</a></p>
      </aside>
    </footer>
  </div>
</template>
