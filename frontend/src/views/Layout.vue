<script setup lang="ts">
import { computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useAuthStore } from '@/stores/auth';
import { useEmailsStore } from '@/stores/emails';

const route = useRoute();
const router = useRouter();
const authStore = useAuthStore();
const emailsStore = useEmailsStore();

const navigation = [
  { name: 'Dashboard', route: 'dashboard', icon: 'home' },
  { name: 'Inbox', route: 'inbox', icon: 'inbox' },
  { name: 'Sent', route: 'sent', icon: 'send' },
  { name: 'Compose', route: 'compose', icon: 'edit' },
  { name: 'Settings', route: 'settings', icon: 'settings' },
];

const currentRoute = computed(() => route.name);

async function logout() {
  await authStore.logout();
  router.push({ name: 'login' });
}
</script>

<template>
  <div class="min-h-screen bg-gray-50">
    <!-- Sidebar -->
    <aside class="fixed inset-y-0 left-0 w-64 bg-white border-r border-gray-200">
      <!-- Logo -->
      <div class="flex items-center h-16 px-6 border-b border-gray-200">
        <div class="h-8 w-8 bg-primary-500 rounded-lg flex items-center justify-center">
          <svg class="h-5 w-5 text-white" fill="currentColor" viewBox="0 0 24 24">
            <path d="M20 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/>
          </svg>
        </div>
        <span class="ml-2 text-xl font-bold text-gray-900">AvaMail</span>
      </div>

      <!-- Navigation -->
      <nav class="flex-1 px-4 py-4 space-y-1">
        <router-link
          v-for="item in navigation"
          :key="item.route"
          :to="{ name: item.route }"
          class="flex items-center px-3 py-2 text-sm font-medium rounded-lg transition-colors"
          :class="currentRoute === item.route || (item.route === 'inbox' && currentRoute === 'email') || (item.route === 'sent' && currentRoute === 'sent-email')
            ? 'bg-primary-50 text-primary-700'
            : 'text-gray-700 hover:bg-gray-100'"
        >
          <!-- Icons -->
          <svg v-if="item.icon === 'home'" class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" />
          </svg>
          <svg v-if="item.icon === 'inbox'" class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-2.586a1 1 0 00-.707.293l-2.414 2.414a1 1 0 01-.707.293h-3.172a1 1 0 01-.707-.293l-2.414-2.414A1 1 0 006.586 13H4" />
          </svg>
          <svg v-if="item.icon === 'send'" class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
          </svg>
          <svg v-if="item.icon === 'edit'" class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
          </svg>
          <svg v-if="item.icon === 'settings'" class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
          </svg>

          <span>{{ item.name }}</span>

          <!-- Unread badge for inbox -->
          <span
            v-if="item.route === 'inbox' && emailsStore.stats.unread > 0"
            class="ml-auto bg-primary-500 text-white text-xs font-medium px-2 py-0.5 rounded-full"
          >
            {{ emailsStore.stats.unread }}
          </span>
        </router-link>
      </nav>

      <!-- User section -->
      <div class="absolute bottom-0 left-0 right-0 border-t border-gray-200">
        <div class="p-4">
          <button
            @click="logout"
            class="flex items-center w-full px-3 py-2 text-sm font-medium text-gray-700 rounded-lg hover:bg-gray-100"
          >
            <svg class="w-5 h-5 mr-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
            </svg>
            Sign Out
          </button>
        </div>
        <!-- Open Source Footer -->
        <div class="px-4 py-3 bg-gray-50 border-t border-gray-200">
          <a
            href="https://github.com/MrOplus/AvaMail"
            target="_blank"
            rel="noopener noreferrer"
            class="flex items-center justify-center text-xs text-gray-500 hover:text-gray-700 transition-colors"
          >
            <svg class="w-4 h-4 mr-1.5" fill="currentColor" viewBox="0 0 24 24">
              <path fill-rule="evenodd" clip-rule="evenodd" d="M12 2C6.477 2 2 6.477 2 12c0 4.42 2.865 8.17 6.839 9.49.5.092.682-.217.682-.482 0-.237-.008-.866-.013-1.7-2.782.604-3.369-1.34-3.369-1.34-.454-1.156-1.11-1.463-1.11-1.463-.908-.62.069-.608.069-.608 1.003.07 1.531 1.03 1.531 1.03.892 1.529 2.341 1.087 2.91.831.092-.646.35-1.086.636-1.336-2.22-.253-4.555-1.11-4.555-4.943 0-1.091.39-1.984 1.029-2.683-.103-.253-.446-1.27.098-2.647 0 0 .84-.269 2.75 1.025A9.578 9.578 0 0112 6.836c.85.004 1.705.114 2.504.336 1.909-1.294 2.747-1.025 2.747-1.025.546 1.377.203 2.394.1 2.647.64.699 1.028 1.592 1.028 2.683 0 3.842-2.339 4.687-4.566 4.935.359.309.678.919.678 1.852 0 1.336-.012 2.415-.012 2.743 0 .267.18.578.688.48C19.138 20.167 22 16.418 22 12c0-5.523-4.477-10-10-10z" />
            </svg>
            Open Source on GitHub
          </a>
        </div>
      </div>
    </aside>

    <!-- Main content -->
    <main class="ml-64 min-h-screen">
      <router-view />
    </main>
  </div>
</template>
