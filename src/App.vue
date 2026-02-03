<script setup>
import { ref } from 'vue'
import TopBar from './components/TopBar.vue'

const sidebarExpanded = ref(true)

const toggleSidebar = () => {
  sidebarExpanded.value = !sidebarExpanded.value
}
</script>

<template>
  <div class="app-layout">
    <!-- Sidebar -->
    <aside class="sidebar" :class="{ expanded: sidebarExpanded }">
      <div class="sidebar-header">
        <span class="logo">{{ sidebarExpanded ? 'CONNEXIA' : 'C' }}</span>
        <button class="toggle-btn" @click="toggleSidebar">
          <svg
            width="16"
            height="16"
            viewBox="0 0 16 16"
            fill="none"
            :class="{ rotated: !sidebarExpanded }"
          >
            <path d="M10 4l-4 4 4 4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
      </div>
    </aside>

    <!-- Main content -->
    <main class="main-content">
      <TopBar :compact-date="sidebarExpanded" />
      <div class="content-area">
        <!-- Contenu principal ici -->
      </div>
    </main>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --color-black: #000000;
  --color-white: #FFFFFF;
  --color-blue: #2A62EE;
  --color-background: #EFF5FC;
}

body {
  font-family: 'Roboto', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: var(--color-black);
}
</style>

<style scoped>
.app-layout {
  display: flex;
  min-height: 100vh;
  background: var(--color-background);
  padding: 10px 0 10px 10px;
}

.sidebar {
  width: 60px;
  background: var(--color-white);
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  transition: width 0.25s ease;
  height: calc(100vh - 20px);
  flex-shrink: 0;
}

.sidebar.expanded {
  width: 240px;
}

.sidebar-header {
  padding: 20px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.sidebar:not(.expanded) .sidebar-header {
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
}

.logo {
  font-size: 20px;
  font-weight: 700;
  color: var(--color-black);
  white-space: nowrap;
  overflow: hidden;
}

.toggle-btn {
  width: 28px;
  height: 28px;
  border: 1px solid #e0e0e0;
  border-radius: 6px;
  background: var(--color-white);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--color-black);
  flex-shrink: 0;
  transition: background-color 0.15s ease;
}

.toggle-btn:hover {
  background: #f5f5f5;
}

.toggle-btn svg {
  transition: transform 0.25s ease;
}

.toggle-btn svg.rotated {
  transform: rotate(180deg);
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.content-area {
  flex: 1;
  padding: 24px;
}
</style>
