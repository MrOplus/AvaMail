<script setup lang="ts">
import { onMounted, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useEmailsStore } from '@/stores/emails';

const router = useRouter();
const emailsStore = useEmailsStore();

onMounted(async () => {
  await emailsStore.fetchEmails('received');
});

const emails = computed(() => emailsStore.inbox);

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
  router.push({ name: 'email', params: { id } });
}

async function toggleStar(e: Event, id: string) {
  e.stopPropagation();
  await emailsStore.toggleStar(id);
}
</script>

<template>
  <div class="p-8">
    <div class="flex items-center justify-between mb-6">
      <h1 class="text-2xl font-bold text-gray-900">Inbox</h1>
      <button
        @click="emailsStore.fetchEmails('received')"
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
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 13V6a2 2 0 00-2-2H6a2 2 0 00-2 2v7m16 0v5a2 2 0 01-2 2H6a2 2 0 01-2-2v-5m16 0h-2.586a1 1 0 00-.707.293l-2.414 2.414a1 1 0 01-.707.293h-3.172a1 1 0 01-.707-.293l-2.414-2.414A1 1 0 006.586 13H4" />
      </svg>
      <h3 class="mt-2 text-sm font-medium text-gray-900">No emails</h3>
      <p class="mt-1 text-sm text-gray-500">Your inbox is empty.</p>
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
            <!-- Star button -->
            <button
              @click="(e) => toggleStar(e, email.id)"
              class="mr-3 text-gray-400 hover:text-yellow-500"
            >
              <svg
                class="h-5 w-5"
                :class="email.is_starred ? 'text-yellow-500 fill-current' : ''"
                :fill="email.is_starred ? 'currentColor' : 'none'"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z" />
              </svg>
            </button>

            <!-- Unread indicator -->
            <div class="w-2 mr-3">
              <div
                v-if="!email.is_read"
                class="w-2 h-2 bg-primary-500 rounded-full"
              ></div>
            </div>

            <!-- Email content -->
            <div class="flex-1 min-w-0">
              <div class="flex items-center justify-between">
                <p
                  class="text-sm truncate"
                  :class="email.is_read ? 'text-gray-600' : 'font-semibold text-gray-900'"
                >
                  {{ email.from_address }}
                </p>
                <p class="text-xs text-gray-500 ml-2 flex-shrink-0">
                  {{ formatDate(email.created_at) }}
                </p>
              </div>
              <p
                class="text-sm truncate"
                :class="email.is_read ? 'text-gray-500' : 'font-medium text-gray-900'"
              >
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
