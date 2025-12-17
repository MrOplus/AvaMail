<script setup lang="ts">
import { onMounted, computed, ref } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useEmailsStore } from '@/stores/emails';
import api from '@/api/client';

const route = useRoute();
const router = useRouter();
const emailsStore = useEmailsStore();
const showDeleteConfirm = ref(false);

const email = computed(() => emailsStore.currentEmail);
const isSent = computed(() => route.name === 'sent-email');

onMounted(async () => {
  const id = route.params.id as string;
  await emailsStore.fetchEmail(id);
});

function formatDate(dateStr: string): string {
  return new Date(dateStr).toLocaleString();
}

function goBack() {
  router.push({ name: isSent.value ? 'sent' : 'inbox' });
}

async function toggleStar() {
  if (email.value) {
    await emailsStore.toggleStar(email.value.id);
  }
}

async function toggleRead() {
  if (email.value) {
    if (email.value.is_read) {
      await emailsStore.markAsUnread(email.value.id);
    } else {
      await emailsStore.markAsRead(email.value.id);
    }
  }
}

async function deleteEmail() {
  if (email.value) {
    await emailsStore.deleteEmail(email.value.id);
    goBack();
  }
}

function replyToEmail() {
  if (email.value) {
    router.push({
      name: 'compose',
      query: {
        replyTo: email.value.id,
        to: email.value.from_address,
        subject: `Re: ${email.value.subject || ''}`,
      },
    });
  }
}

function formatFileSize(bytes: number): string {
  if (bytes < 1024) return bytes + ' B';
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + ' KB';
  return (bytes / (1024 * 1024)).toFixed(1) + ' MB';
}

function downloadAttachment(attachmentId: string, filename: string) {
  if (!email.value) return;
  const url = api.getAttachmentUrl(email.value.id, attachmentId);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.target = '_blank';
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
}

function getFileIcon(contentType: string): string {
  if (contentType.startsWith('image/')) return 'image';
  if (contentType.startsWith('video/')) return 'video';
  if (contentType.startsWith('audio/')) return 'audio';
  if (contentType.includes('pdf')) return 'pdf';
  if (contentType.includes('word') || contentType.includes('document')) return 'doc';
  if (contentType.includes('sheet') || contentType.includes('excel')) return 'sheet';
  if (contentType.includes('zip') || contentType.includes('archive') || contentType.includes('compressed')) return 'archive';
  return 'file';
}
</script>

<template>
  <div class="p-8">
    <!-- Header -->
    <div class="flex items-center justify-between mb-6">
      <button
        @click="goBack"
        class="flex items-center text-gray-600 hover:text-gray-900"
      >
        <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
        </svg>
        Back to {{ isSent ? 'Sent' : 'Inbox' }}
      </button>

      <div v-if="email" class="flex items-center space-x-2">
        <button
          v-if="!isSent"
          @click="replyToEmail"
          class="flex items-center px-3 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50"
        >
          <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h10a8 8 0 018 8v2M3 10l6 6m-6-6l6-6" />
          </svg>
          Reply
        </button>

        <button
          v-if="!isSent"
          @click="toggleStar"
          class="p-2 text-gray-400 hover:text-yellow-500 rounded-lg hover:bg-gray-100"
          :title="email.is_starred ? 'Remove star' : 'Add star'"
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

        <button
          v-if="!isSent"
          @click="toggleRead"
          class="p-2 text-gray-400 hover:text-gray-600 rounded-lg hover:bg-gray-100"
          :title="email.is_read ? 'Mark as unread' : 'Mark as read'"
        >
          <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
          </svg>
        </button>

        <button
          @click="showDeleteConfirm = true"
          class="p-2 text-gray-400 hover:text-red-500 rounded-lg hover:bg-gray-100"
          title="Delete"
        >
          <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
          </svg>
        </button>
      </div>
    </div>

    <!-- Loading state -->
    <div v-if="emailsStore.isLoading" class="flex justify-center py-12">
      <div class="spinner"></div>
    </div>

    <!-- Email content -->
    <div v-else-if="email" class="bg-white rounded-lg shadow">
      <!-- Email header -->
      <div class="p-6 border-b border-gray-200">
        <h1 class="text-xl font-semibold text-gray-900 mb-4">
          {{ email.subject || '(No Subject)' }}
        </h1>

        <div class="flex items-start justify-between">
          <div>
            <p class="text-sm font-medium text-gray-900">
              {{ isSent ? 'To' : 'From' }}: {{ isSent ? email.to_address : email.from_address }}
            </p>
            <p v-if="!isSent" class="text-sm text-gray-500">
              To: {{ email.to_address }}
            </p>
            <p v-if="email.cc" class="text-sm text-gray-500">
              CC: {{ email.cc }}
            </p>
          </div>
          <p class="text-sm text-gray-500">
            {{ formatDate(email.created_at) }}
          </p>
        </div>
      </div>

      <!-- Email body -->
      <div class="p-6">
        <div
          v-if="email.body_html"
          class="email-content"
          v-html="email.body_html"
        ></div>
        <pre
          v-else-if="email.body_text"
          class="whitespace-pre-wrap font-sans text-sm text-gray-700"
        >{{ email.body_text }}</pre>
        <p v-else class="text-gray-500 italic">No content</p>
      </div>

      <!-- Attachments -->
      <div v-if="email.attachments && email.attachments.length > 0" class="px-6 pb-6">
        <div class="border-t border-gray-200 pt-4">
          <h3 class="text-sm font-medium text-gray-900 mb-3">
            Attachments ({{ email.attachments.length }})
          </h3>
          <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3">
            <button
              v-for="attachment in email.attachments"
              :key="attachment.id"
              @click="downloadAttachment(attachment.id, attachment.filename)"
              class="flex items-center p-3 bg-gray-50 rounded-lg hover:bg-gray-100 transition-colors text-left"
            >
              <!-- File icon -->
              <div class="flex-shrink-0 w-10 h-10 bg-primary-100 rounded-lg flex items-center justify-center">
                <svg v-if="getFileIcon(attachment.content_type) === 'image'" class="w-5 h-5 text-primary-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
                </svg>
                <svg v-else-if="getFileIcon(attachment.content_type) === 'pdf'" class="w-5 h-5 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21h10a2 2 0 002-2V9.414a1 1 0 00-.293-.707l-5.414-5.414A1 1 0 0012.586 3H7a2 2 0 00-2 2v14a2 2 0 002 2z" />
                </svg>
                <svg v-else class="w-5 h-5 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.172 7l-6.586 6.586a2 2 0 102.828 2.828l6.414-6.586a4 4 0 00-5.656-5.656l-6.415 6.585a6 6 0 108.486 8.486L20.5 13" />
                </svg>
              </div>
              <div class="ml-3 min-w-0 flex-1">
                <p class="text-sm font-medium text-gray-900 truncate">{{ attachment.filename }}</p>
                <p class="text-xs text-gray-500">{{ formatFileSize(attachment.size) }}</p>
              </div>
              <svg class="w-5 h-5 text-gray-400 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
              </svg>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Not found -->
    <div v-else class="text-center py-12">
      <p class="text-gray-500">Email not found</p>
    </div>

    <!-- Delete confirmation modal -->
    <div
      v-if="showDeleteConfirm"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
      @click.self="showDeleteConfirm = false"
    >
      <div class="bg-white rounded-lg shadow-xl p-6 max-w-sm w-full mx-4">
        <h3 class="text-lg font-medium text-gray-900 mb-4">Delete Email</h3>
        <p class="text-sm text-gray-500 mb-6">
          Are you sure you want to delete this email? This action cannot be undone.
        </p>
        <div class="flex justify-end space-x-3">
          <button
            @click="showDeleteConfirm = false"
            class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50"
          >
            Cancel
          </button>
          <button
            @click="deleteEmail"
            class="px-4 py-2 text-sm font-medium text-white bg-red-600 rounded-lg hover:bg-red-700"
          >
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
