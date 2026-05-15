<script setup lang="ts">
const props = defineProps<{
  name: string
}>()

const ICON_PATHS: Record<string, string> = {
  gym: '/icons/status/gym.svg',
  cardio: '/icons/status/cardio.svg',
  rest: '/icons/status/rest.svg'
}

const normalizedName = computed(() =>
  props.name.replace(/([a-z0-9])([A-Z])/g, '$1-$2').toLowerCase()
)

const src = computed(() => ICON_PATHS[normalizedName.value] ?? `/icons/${normalizedName.value}.svg`)
</script>

<template>
  <img
    :src="src"
    :alt="props.name"
    loading="lazy"
    decoding="async"
    draggable="false"
  >
</template>
