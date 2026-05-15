<script setup lang="ts">
const name = ref('')
const email = ref('')
const password = ref('')
const confirmPassword = ref('')

function handleRegister() {
  // TODO: implement register logic
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gray-950 px-4">
    <div class="w-full max-w-sm">
      <div class="text-center mb-8">
        <h1 class="text-3xl font-bold text-white">
          Create account
        </h1>
      </div>

      <div class="bg-gray-900 border border-gray-800 rounded-xl p-6 space-y-4">
        <UFormField label="Name">
          <UInput
            v-model="name"
            type="text"
            placeholder="Your name"
            class="w-full"
          />
        </UFormField>

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

        <UFormField label="Confirm Password">
          <UInput
            v-model="confirmPassword"
            type="password"
            placeholder="••••••••"
            class="w-full"
          />
        </UFormField>

        <div class="text-sm">
          <NuxtLink
            to="/login"
            class="text-gray-400 hover:text-white transition-colors"
          >
            Already have an account?
          </NuxtLink>
        </div>

        <UButton
          color="primary"
          class="w-full justify-center"
          @click="handleRegister"
        >
          Create account
        </UButton>
      </div>
    </div>
  </div>
</template>
