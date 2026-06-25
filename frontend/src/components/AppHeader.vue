<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessageBox } from 'element-plus'
import { useAuthStore } from '@/stores/auth'

const route = useRoute()
const router = useRouter()
const authStore = useAuthStore()
const avatarTimestamp = ref(Date.now())
const userMenuVisible = ref(false)
const activeWorkspace = computed(() => {
  if (route.path.startsWith('/interview')) return 'interview'
  if (route.path.startsWith('/my-questions')) return 'questions'
  if (route.path.startsWith('/profile')) return 'profile'
  return 'chat'
})

watch(
  () => authStore.user?.avatar_url,
  () => {
    avatarTimestamp.value = Date.now()
  }
)

function getAvatarUrl(): string {
  const url = authStore.currentUser?.avatar_url
  if (!url) return ''
  return url.includes('?') ? `${url}&t=${avatarTimestamp.value}` : `${url}?t=${avatarTimestamp.value}`
}

function goToProfile() {
  userMenuVisible.value = false
  router.push('/profile')
}

function goToMyQuestions() {
  userMenuVisible.value = false
  router.push('/my-questions')
}

function goToChat() {
  router.push('/')
}

function goToInterview() {
  router.push('/interview')
}

function toggleUserMenu() {
  userMenuVisible.value = !userMenuVisible.value
}

function closeUserMenu() {
  userMenuVisible.value = false
}

function handleLogout() {
  userMenuVisible.value = false
  ElMessageBox.confirm('确定要退出登录吗？', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning',
  })
    .then(() => authStore.logout())
    .catch(() => {})
}

onMounted(() => {
  document.addEventListener('click', closeUserMenu)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', closeUserMenu)
})
</script>

<template>
  <header class="app-header">
    <div class="header-left">
      <img class="app-logo" src="/logo.png" alt="logo" />
      <span class="app-title">智能面试助手</span>
    </div>

    <nav class="workspace-tabs" aria-label="工作区切换">
      <button
        class="workspace-tab"
        :class="{ active: activeWorkspace === 'chat' }"
        type="button"
        @click="goToChat"
      >
        智能对话
      </button>
      <button
        class="workspace-tab"
        :class="{ active: activeWorkspace === 'interview' }"
        type="button"
        @click="goToInterview"
      >
        AI 面试
      </button>
      <button
        class="workspace-tab"
        :class="{ active: activeWorkspace === 'questions' }"
        type="button"
        @click="goToMyQuestions"
      >
        我的题库
      </button>
      <button
        class="workspace-tab"
        :class="{ active: activeWorkspace === 'profile' }"
        type="button"
        @click="goToProfile"
      >
        个人中心
      </button>
    </nav>

    <div class="header-right">
      <div class="user-menu-wrap" @click.stop>
        <div class="user-info" @click="toggleUserMenu">
          <el-avatar :size="32" :src="getAvatarUrl()" :key="avatarTimestamp" :style="{ backgroundColor: '#534AB7' }">
            {{ authStore.currentUser?.username?.charAt(0)?.toUpperCase() || 'U' }}
          </el-avatar>
          <span class="user-name">{{ authStore.currentUser?.username || '用户' }}</span>
          <el-icon :size="12"><ArrowDown /></el-icon>
        </div>
        <div v-if="userMenuVisible" class="user-menu">
          <button class="user-menu-item" @click="handleLogout">
            <el-icon><SwitchButton /></el-icon>
            <span>退出登录</span>
          </button>
        </div>
      </div>
    </div>
  </header>
</template>

<style scoped>
.app-header {
  height: var(--header-height);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 var(--spacing-lg) 0;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.94), rgba(250, 250, 250, 0.82)),
    var(--color-bg);
  border-bottom: 1px solid rgba(9, 9, 11, 0.045);
  box-shadow: 0 12px 34px rgba(9, 9, 11, 0.018);
  backdrop-filter: blur(16px);
  flex-shrink: 0;
  position: relative;
  z-index: 5;
}

.app-header::after {
  content: "";
  position: absolute;
  left: var(--sidebar-width);
  right: 0;
  bottom: -1px;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(79, 70, 229, 0.08), transparent);
  pointer-events: none;
}

.header-left {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
}

.app-logo {
  width: 32px;
  height: 32px;
  border-radius: var(--radius-sm);
  object-fit: cover;
  box-shadow: 0 8px 20px rgba(6, 182, 212, 0.16);
}

.app-title {
  font-size: var(--font-lg);
  font-weight: 700;
  color: var(--color-text);
  letter-spacing: 0;
}

.workspace-tabs {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 56px;
  position: absolute;
  left: 50%;
  bottom: 0;
  transform: translateX(-50%);
  height: var(--header-height);
  padding: 0;
}

.workspace-tab {
  position: relative;
  min-width: 104px;
  height: var(--header-height);
  padding: 0 6px;
  color: var(--color-text-secondary);
  font-size: 16px;
  font-weight: 650;
  letter-spacing: 0;
  transition: color 0.18s ease;
}

.workspace-tab::after {
  content: "";
  position: absolute;
  left: 14px;
  right: 14px;
  bottom: 0;
  height: 3px;
  border-radius: 999px;
  background: transparent;
  transform: scaleX(0.5);
  opacity: 0;
  transition: background 0.18s ease, opacity 0.18s ease, transform 0.18s ease;
}

.workspace-tab:hover {
  color: var(--color-primary);
}

.workspace-tab.active {
  color: var(--color-primary);
  font-weight: 750;
}

.workspace-tab.active::after {
  background: linear-gradient(90deg, var(--color-primary), var(--color-accent));
  opacity: 1;
  transform: scaleX(1);
}

.header-right {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
}

.theme-toggle-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.theme-toggle-wrap::after {
  content: attr(data-tooltip);
  position: absolute;
  top: calc(100% + 8px);
  left: 50%;
  z-index: 120;
  min-width: 64px;
  padding: 5px 8px;
  border-radius: 7px;
  background: var(--color-text);
  color: var(--color-text-inverse);
  font-size: 12px;
  line-height: 1.2;
  text-align: center;
  white-space: nowrap;
  box-shadow: 0 10px 24px rgba(9, 9, 11, 0.14);
  opacity: 0;
  pointer-events: none;
  transform: translate(-50%, -2px);
  transition: opacity 0.16s ease, transform 0.16s ease;
}

.theme-toggle-wrap::before {
  content: "";
  position: absolute;
  top: calc(100% + 3px);
  left: 50%;
  z-index: 121;
  width: 8px;
  height: 8px;
  background: var(--color-text);
  opacity: 0;
  pointer-events: none;
  transform: translate(-50%, -2px) rotate(45deg);
  transition: opacity 0.16s ease, transform 0.16s ease;
}

.theme-toggle-wrap:hover::after,
.theme-toggle-wrap:hover::before {
  opacity: 1;
  transform: translate(-50%, 0) rotate(0deg);
}

.theme-toggle-wrap:hover::before {
  transform: translate(-50%, 0) rotate(45deg);
}

.user-info {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  padding: 5px 10px 5px 6px;
  border: 1px solid var(--color-subtle-line);
  border-radius: 999px;
  cursor: pointer;
  transition: background 0.2s, box-shadow 0.2s, transform 0.2s;
  background: var(--color-surface-glass-soft);
}

.user-menu-wrap {
  position: relative;
}

.user-menu {
  position: absolute;
  top: calc(100% + 10px);
  right: 0;
  width: 148px;
  padding: 6px;
  background: var(--color-card);
  border: 1px solid var(--color-subtle-line);
  border-radius: 10px;
  box-shadow: 0 16px 38px rgba(9, 9, 11, 0.12);
  z-index: 100;
}

.user-menu::before {
  content: "";
  position: absolute;
  top: -6px;
  right: 22px;
  width: 10px;
  height: 10px;
  background: var(--color-card);
  border-left: 1px solid var(--color-subtle-line);
  border-top: 1px solid var(--color-subtle-line);
  transform: rotate(45deg);
}

.user-menu-item {
  position: relative;
  z-index: 1;
  width: 100%;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 9px 10px;
  border-radius: 8px;
  color: var(--color-text-secondary);
  font-size: var(--font-sm);
  text-align: left;
  background: transparent;
}

.user-menu-item:hover {
  color: var(--color-primary);
  background: var(--color-primary-lighter);
}

.user-info:hover {
  background: var(--color-surface-glass-strong);
  box-shadow: 0 10px 24px rgba(36, 38, 66, 0.08);
  transform: translateY(-1px);
}

.user-name {
  font-size: var(--font-sm);
  color: var(--color-text);
  max-width: 120px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

@media (max-width: 1100px) {
  .workspace-tabs {
    gap: 28px;
  }

  .workspace-tab {
    min-width: 88px;
    font-size: 15px;
  }
}

@media (max-width: 768px) {
  .app-header {
    padding: 0 12px;
  }

  .app-title,
  .user-name {
    display: none;
  }

  .workspace-tabs {
    position: static;
    transform: none;
    flex: 1;
    justify-content: center;
    height: 36px;
    padding: 0;
    margin-left: var(--spacing-sm);
    gap: 10px;
    border: none;
    border-radius: 0;
    background: transparent;
    box-shadow: none;
    backdrop-filter: none;
  }

  .workspace-tab {
    min-width: auto;
    height: 36px;
    padding: 0 4px;
    border-radius: 10px;
    border-bottom: 1px solid transparent;
    font-size: 12px;
  }

  .workspace-tab::after {
    left: 10px;
    right: 10px;
  }
}

:global(.theme-dark) .app-header {
  background:
    linear-gradient(180deg, rgba(17, 19, 26, 0.94), rgba(9, 10, 15, 0.86)),
    var(--color-bg);
  border-bottom-color: rgba(255, 255, 255, 0.08);
}

:global(.theme-dark) .app-header::after {
  background: linear-gradient(90deg, transparent, rgba(99, 102, 241, 0.16), transparent);
}

:global(.theme-dark) .user-info {
  background: rgba(22, 24, 34, 0.72);
  border-color: rgba(255, 255, 255, 0.08);
}

:global(.theme-dark) .workspace-tab:hover {
  color: var(--color-primary-light);
}

:global(.theme-dark) .workspace-tabs {
  background: transparent;
  border-color: transparent;
  box-shadow: none;
}

:global(.theme-dark) .workspace-tab.active {
  color: var(--color-primary-light);
}

@media (max-width: 768px) {
  :global(.theme-dark) .workspace-tabs {
    background: transparent;
    border-color: transparent;
    box-shadow: none;
  }
}

:global(.theme-dark) .user-menu {
  background: #161822;
  border-color: rgba(255, 255, 255, 0.08);
  box-shadow: 0 18px 42px rgba(0, 0, 0, 0.42);
}

:global(.theme-dark) .user-menu::before {
  background: #161822;
  border-color: rgba(255, 255, 255, 0.08);
}
</style>
