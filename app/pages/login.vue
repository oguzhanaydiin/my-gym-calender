<script setup lang="ts">
const email = ref('')
const password = ref('')

function handleLogin() {
  // TODO: implement login logic
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-950 px-4">
    <div class="w-full max-w-sm">
      <div class="text-center mb-8">
        <h1 class="text-3xl font-bold text-white">
          Welcome back
        </h1>
      </div>

      <div class="bg-gray-900 border border-gray-800 rounded-xl p-6 space-y-4">
        <UFormField label="Email">
          <UInput
            v-model="email"
            type="email"
            placeholder="you@example.com"
            class="w-full"
          />
        </UFormField>

        <UFormField label="Password">
          <UInput
            v-model="password"
            type="password"
            placeholder="••••••••"
            class="w-full"
          />
        </UFormField>

        <div class="flex items-center justify-between text-sm">
          <NuxtLink
            to="/register"
            class="text-gray-400 hover:text-white transition-colors"
          >
            Don't have an account?
          </NuxtLink>
          <a
            href="#"
            class="text-primary-400 hover:text-primary-300 transition-colors"
          >
            Forgot password?
          </a>
        </div>

        <UButton
          color="primary"
          class="w-full justify-center"
          @click="handleLogin"
        >
          Sign in
        </UButton>
      </div>
    </div>
  </div>
</template>
