<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { useRouter } from 'vue-router';
import { useEmailsStore } from '@/stores/emails';
import api from '@/api/client';
import type { SetupStatus } from '@/api/types';

const router = useRouter();
const emailsStore = useEmailsStore();
const setupStatus = ref<SetupStatus | null>(null);
const isLoading = ref(true);

onMounted(async () => {
  try {
    await Promise.all([
      emailsStore.fetchStats(),
      loadSetupStatus(),
    ]);
  } finally {
    isLoading.value = false;
  }
});

async function loadSetupStatus() {
  setupStatus.value = await api.getSetupStatus();
}

const stats = [
  { name: 'Inbox', key: 'total_received', icon: 'inbox', route: 'inbox', color: 'bg-blue-500' },
  { name: 'Sent', key: 'total_sent', icon: 'send', route: 'sent', color: 'bg-green-500' },
  { name: 'Unread', key: 'unread', icon: 'mail', route: 'inbox', color: 'bg-yellow-500' },
  { name: 'Starred', key: 'starred', icon: 'star', route: 'inbox', color: 'bg-purple-500' },
];

function getStatValue(key: string): number {
  return emailsStore.stats[key as keyof typeof emailsStore.stats] || 0;
}

function navigateTo(route: string) {
  router.push({ name: route });
}
</script>

<template>
  <div class="p-8">
    <h1 class="text-2xl font-bold text-gray-900 mb-8">Dashboard</h1>

    <!-- Loading state -->
    <div v-if="isLoading" class="flex justify-center py-12">
      <div class="spinner"></div>
    </div>

    <template v-else>
      <!-- Setup warning -->
      <div v-if="!setupStatus?.setupCompleted" class="mb-8 bg-yellow-50 border border-yellow-200 rounded-lg p-4">
        <div class="flex">
          <svg class="h-5 w-5 text-yellow-400" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M8.257 3.099c.765-1.36 2.722-1.36 3.486 0l5.58 9.92c.75 1.334-.213 2.98-1.742 2.98H4.42c-1.53 0-2.493-1.646-1.743-2.98l5.58-9.92zM11 13a1 1 0 11-2 0 1 1 0 012 0zm-1-8a1 1 0 00-1 1v3a1 1 0 002 0V6a1 1 0 00-1-1z" clip-rule="evenodd" />
          </svg>
          <div class="ml-3">
            <h3 class="text-sm font-medium text-yellow-800">Setup Required</h3>
            <p class="mt-1 text-sm text-yellow-700">
              Configure Cloudflare and Brevo to start receiving and sending emails.
            </p>
            <router-link
              :to="{ name: 'settings' }"
              class="mt-2 inline-flex items-center text-sm font-medium text-yellow-800 hover:text-yellow-900"
            >
              Go to Settings
              <svg class="ml-1 h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
              </svg>
            </router-link>
          </div>
        </div>
      </div>

      <!-- Stats grid -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
        <button
          v-for="stat in stats"
          :key="stat.key"
          @click="navigateTo(stat.route)"
          class="bg-white rounded-lg shadow p-6 hover:shadow-md transition-shadow text-left"
        >
          <div class="flex items-center">
            <div :class="[stat.color, 'rounded-lg p-3']">
              <svg v-if="stat.icon === 'inbox'" class="h-6 w-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-2.586a1 1 0 00-.707.293l-2.414 2.414a1 1 0 01-.707.293h-3.172a1 1 0 01-.707-.293l-2.414-2.414A1 1 0 006.586 13H4" />
              </svg>
              <svg v-if="stat.icon === 'send'" class="h-6 w-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
              </svg>
              <svg v-if="stat.icon === 'mail'" class="h-6 w-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
              </svg>
              <svg v-if="stat.icon === 'star'" class="h-6 w-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z" />
              </svg>
            </div>
            <div class="ml-4">
              <p class="text-sm font-medium text-gray-500">{{ stat.name }}</p>
              <p class="text-2xl font-semibold text-gray-900">{{ getStatValue(stat.key) }}</p>
            </div>
          </div>
        </button>
      </div>

      <!-- Setup status -->
      <div v-if="setupStatus" class="bg-white rounded-lg shadow p-6">
        <h2 class="text-lg font-semibold text-gray-900 mb-4">Configuration Status</h2>
        <div class="space-y-4">
          <div class="flex items-center justify-between">
            <div class="flex items-center">
              <div :class="[setupStatus.cloudflare.configured ? 'bg-green-100' : 'bg-gray-100', 'rounded-full p-2']">
                <svg :class="[setupStatus.cloudflare.configured ? 'text-green-600' : 'text-gray-400', 'h-5 w-5']" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path v-if="setupStatus.cloudflare.configured" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                  <path v-else stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </div>
              <span class="ml-3 text-sm font-medium text-gray-900">Cloudflare Email Routing</span>
            </div>
            <span :class="[setupStatus.cloudflare.configured ? 'text-green-600' : 'text-gray-500', 'text-sm']">
              {{ setupStatus.cloudflare.configured ? 'Configured' : 'Not configured' }}
            </span>
          </div>

          <div class="flex items-center justify-between">
            <div class="flex items-center">
              <div :class="[setupStatus.brevo.configured ? 'bg-green-100' : 'bg-gray-100', 'rounded-full p-2']">
                <svg :class="[setupStatus.brevo.configured ? 'text-green-600' : 'text-gray-400', 'h-5 w-5']" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path v-if="setupStatus.brevo.configured" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                  <path v-else stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </div>
              <span class="ml-3 text-sm font-medium text-gray-900">Brevo (Sending)</span>
            </div>
            <span :class="[setupStatus.brevo.configured ? 'text-green-600' : 'text-gray-500', 'text-sm']">
              {{ setupStatus.brevo.configured ? 'Configured' : 'Not configured' }}
            </span>
          </div>

          <div v-if="setupStatus.domain" class="pt-4 border-t border-gray-200">
            <p class="text-sm text-gray-500">Domain</p>
            <p class="text-sm font-medium text-gray-900">{{ setupStatus.domain }}</p>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>
