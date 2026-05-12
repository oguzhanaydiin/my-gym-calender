<script setup lang="ts">
const props = defineProps<{
  name: string
}>()

const failed = ref(false)
const currentPathIndex = ref(0)

const normalizedName = computed(() => {
  return props.name
    .replace(/([a-z0-9])([A-Z])/g, '$1-$2')
    .toLowerCase()
})

const candidatePaths = computed(() => {
  return [
    `/icons/ui/${normalizedName.value}.svg`,
    `/icons/status/${normalizedName.value}.svg`,
    `/icons/${normalizedName.value}.svg`,
  ]
})

const src = computed(() => candidatePaths.value[currentPathIndex.value])

watch(() => props.name, () => {
  currentPathIndex.value = 0
  failed.value = false
})

function onImageError() {
  if (currentPathIndex.value < candidatePaths.value.length - 1) {
    currentPathIndex.value += 1
  } else {
    failed.value = true
  }
}
</script>

<template>
  <img
    v-if="!failed"
    :src="src"
    :alt="props.name"
    loading="lazy"
    decoding="async"
    draggable="false"
    @error="onImageError"
  >
</template>
