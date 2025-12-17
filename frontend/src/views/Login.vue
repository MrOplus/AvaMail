<script setup lang="ts">
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';
import { useAuthStore } from '@/stores/auth';

const router = useRouter();
const authStore = useAuthStore();

const password = ref('');
const confirmPassword = ref('');
const error = ref('');
const isLoading = ref(false);

const isSetupMode = computed(() => authStore.needsSetup);

async function handleSubmit() {
  error.value = '';

  if (!password.value) {
    error.value = 'Password is required';
    return;
  }

  if (isSetupMode.value) {
    if (password.value.length < 8) {
      error.value = 'Password must be at least 8 characters';
      return;
    }
    if (password.value !== confirmPassword.value) {
      error.value = 'Passwords do not match';
      return;
    }
  }

  isLoading.value = true;

  try {
    if (isSetupMode.value) {
      await authStore.setup(password.value);
      router.push({ name: 'setup' });
    } else {
      await authStore.login(password.value);
      router.push({ name: 'dashboard' });
    }
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Authentication failed';
  } finally {
    isLoading.value = false;
  }
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-50 py-12 px-4 sm:px-6 lg:px-8">
    <div class="max-w-md w-full space-y-8">
      <div>
        <div class="mx-auto h-16 w-16 bg-primary-500 rounded-2xl flex items-center justify-center">
          <svg class="h-10 w-10 text-white" fill="currentColor" viewBox="0 0 24 24">
            <path d="M20 4H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z"/>
          </svg>
        </div>
        <h2 class="mt-6 text-center text-3xl font-extrabold text-gray-900">
          {{ isSetupMode ? 'Welcome to AvaMail' : 'Sign in to AvaMail' }}
        </h2>
        <p class="mt-2 text-center text-sm text-gray-600">
          {{ isSetupMode ? 'Set up your admin password to get started' : 'Enter your password to continue' }}
        </p>
      </div>

      <form class="mt-8 space-y-6" @submit.prevent="handleSubmit">
        <div class="rounded-md shadow-sm space-y-4">
          <div>
            <label for="password" class="block text-sm font-medium text-gray-700">Password</label>
            <input
              id="password"
              v-model="password"
              type="password"
              required
              :minlength="isSetupMode ? 8 : undefined"
              class="mt-1 appearance-none relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 rounded-lg focus:outline-none focus:ring-primary-500 focus:border-primary-500 focus:z-10 sm:text-sm"
              :placeholder="isSetupMode ? 'Create a password (min 8 characters)' : 'Enter your password'"
            />
          </div>

          <div v-if="isSetupMode">
            <label for="confirmPassword" class="block text-sm font-medium text-gray-700">Confirm Password</label>
            <input
              id="confirmPassword"
              v-model="confirmPassword"
              type="password"
              required
              class="mt-1 appearance-none relative block w-full px-3 py-2 border border-gray-300 placeholder-gray-500 text-gray-900 rounded-lg focus:outline-none focus:ring-primary-500 focus:border-primary-500 focus:z-10 sm:text-sm"
              placeholder="Confirm your password"
            />
          </div>
        </div>

        <div v-if="error" class="rounded-md bg-red-50 p-4">
          <div class="flex">
            <div class="flex-shrink-0">
              <svg class="h-5 w-5 text-red-400" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
              </svg>
            </div>
            <div class="ml-3">
              <p class="text-sm font-medium text-red-800">{{ error }}</p>
            </div>
          </div>
        </div>

        <div>
          <button
            type="submit"
            :disabled="isLoading"
            class="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-lg text-white bg-primary-600 hover:bg-primary-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-primary-500 disabled:opacity-50 disabled:cursor-not-allowed"
          >
            <span v-if="isLoading" class="absolute left-0 inset-y-0 flex items-center pl-3">
              <div class="h-5 w-5 border-2 border-white border-t-transparent rounded-full animate-spin"></div>
            </span>
            {{ isLoading ? 'Please wait...' : (isSetupMode ? 'Create Account' : 'Sign In') }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>
