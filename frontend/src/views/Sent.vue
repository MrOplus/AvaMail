<script setup lang="ts">
import { onMounted, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useEmailsStore } from '@/stores/emails';

const router = useRouter();
const emailsStore = useEmailsStore();

onMounted(async () => {
  await emailsStore.fetchEmails('sent');
});

const emails = computed(() => emailsStore.sent);

function formatDate(dateStr: string): string {
  const date = new Date(dateStr);
  const now = new Date();
  const diff = now.getTime() - date.getTime();
  const days = Math.floor(diff / (1000 * 60 * 60 * 24));

  if (days === 0) {
    return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  } else if (days === 1) {
    return 'Yesterday';
  } else if (days < 7) {
    return date.toLocaleDateString([], { weekday: 'short' });
  } else {
    return date.toLocaleDateString([], { month: 'short', day: 'numeric' });
  }
}

function openEmail(id: string) {
  router.push({ name: 'sent-email', params: { id } });
}
</script>

<template>
  <div class="p-8">
    <div class="flex items-center justify-between mb-6">
      <h1 class="text-2xl font-bold text-gray-900">Sent</h1>
      <button
        @click="emailsStore.fetchEmails('sent')"
        class="flex items-center px-3 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50"
      >
        <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
        </svg>
        Refresh
      </button>
    </div>

    <!-- Loading state -->
    <div v-if="emailsStore.isLoading" class="flex justify-center py-12">
      <div class="spinner"></div>
    </div>

    <!-- Empty state -->
    <div v-else-if="emails.length === 0" class="text-center py-12">
      <svg class="mx-auto h-12 w-12 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8" />
      </svg>
      <h3 class="mt-2 text-sm font-medium text-gray-900">No sent emails</h3>
      <p class="mt-1 text-sm text-gray-500">You haven't sent any emails yet.</p>
      <router-link
        :to="{ name: 'compose' }"
        class="mt-4 inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-lg shadow-sm text-white bg-primary-600 hover:bg-primary-700"
      >
        Compose Email
      </router-link>
    </div>

    <!-- Email list -->
    <div v-else class="bg-white rounded-lg shadow overflow-hidden">
      <ul class="divide-y divide-gray-200">
        <li
          v-for="email in emails"
          :key="email.id"
          @click="openEmail(email.id)"
          class="hover:bg-gray-50 cursor-pointer"
        >
          <div class="px-4 py-4 flex items-center">
            <!-- Email content -->
            <div class="flex-1 min-w-0">
              <div class="flex items-center justify-between">
                <p class="text-sm text-gray-600 truncate">
                  To: {{ email.to_address }}
                </p>
                <p class="text-xs text-gray-500 ml-2 flex-shrink-0">
                  {{ formatDate(email.created_at) }}
                </p>
              </div>
              <p class="text-sm font-medium text-gray-900 truncate">
                {{ email.subject || '(No Subject)' }}
              </p>
              <p class="text-sm text-gray-500 truncate">
                {{ email.body_text?.substring(0, 100) || 'No preview available' }}
              </p>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>
