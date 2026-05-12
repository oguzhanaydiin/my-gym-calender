<script setup lang="ts">
type DayTag = 'gym' | 'cardio' | 'rest'
type MuscleGroup = 'legs' | 'ass' | 'shoulders' | 'chest' | 'back' | 'biceps' | 'triceps'

type DayLog = {
  tags: DayTag[]
  muscles: MuscleGroup[]
  note: string
}

const MUSCLE_OPTIONS: Array<{ key: MuscleGroup, label: string }> = [
  { key: 'legs', label: 'Legs' },
  { key: 'ass', label: 'Ass' },
  { key: 'shoulders', label: 'Shoulders' },
  { key: 'chest', label: 'Chest' },
  { key: 'back', label: 'Back' },
  { key: 'biceps', label: 'Biceps' },
  { key: 'triceps', label: 'Triceps' }
]

const STORAGE_KEY = 'gym-calendar-logs-v1'
const WEEKDAYS = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat']

const today = new Date()
const viewedMonth = ref(new Date(today.getFullYear(), today.getMonth(), 1))
const selectedDateKey = ref(toDateKey(today.getFullYear(), today.getMonth(), today.getDate()))
const noteInput = ref('')
const logs = ref<Record<string, DayLog>>({})
const isModalOpen = ref(false)

const monthTitle = computed(() => {
  return new Intl.DateTimeFormat('en-US', {
    month: 'long',
    year: 'numeric'
  }).format(viewedMonth.value)
})

const daysInViewedMonth = computed(() => {
  const year = viewedMonth.value.getFullYear()
  const month = viewedMonth.value.getMonth()
  return new Date(year, month + 1, 0).getDate()
})

const firstWeekday = computed(() => {
  const year = viewedMonth.value.getFullYear()
  const month = viewedMonth.value.getMonth()
  return new Date(year, month, 1).getDay()
})

const calendarDays = computed(() => {
  const cells: Array<number | null> = []

  for (let i = 0; i < firstWeekday.value; i += 1) {
    cells.push(null)
  }

  for (let day = 1; day <= daysInViewedMonth.value; day += 1) {
    cells.push(day)
  }

  while (cells.length % 7 !== 0) {
    cells.push(null)
  }

  return cells
})

const monthSummary = computed(() => {
  const year = viewedMonth.value.getFullYear()
  const month = viewedMonth.value.getMonth()

  return Array.from({ length: daysInViewedMonth.value }, (_, index) => index + 1).reduce(
    (acc, day) => {
      const key = toDateKey(year, month, day)
      const tags = logs.value[key]?.tags ?? []

      if (tags.includes('gym')) acc.gym += 1
      if (tags.includes('cardio')) acc.cardio += 1
      if (tags.includes('rest')) acc.rest += 1

      return acc
    },
    { gym: 0, cardio: 0, rest: 0 }
  )
})

const selectedDateLabel = computed(() => {
  const [year = today.getFullYear(), month = today.getMonth() + 1, day = today.getDate()] = selectedDateKey.value
    .split('-')
    .map(Number)

  return new Intl.DateTimeFormat('en-US', {
    weekday: 'long',
    month: 'long',
    day: 'numeric',
    year: 'numeric'
  }).format(new Date(year, month - 1, day))
})

const selectedLog = computed<DayLog>(() => {
  return logs.value[selectedDateKey.value] ?? { tags: [], muscles: [], note: '' }
})

onMounted(() => {
  const raw = localStorage.getItem(STORAGE_KEY)

  if (raw) {
    try {
      const parsed = JSON.parse(raw) as Record<string, Partial<DayLog> & { status?: DayTag }>
      const normalized: Record<string, DayLog> = {}

      for (const [key, value] of Object.entries(parsed)) {
        const tags = Array.isArray(value.tags) ? value.tags : (value.status ? [value.status] : [])
        const muscles = Array.isArray(value.muscles) ? value.muscles : []
        const note = typeof value.note === 'string' ? value.note : ''

        normalized[key] = {
          tags: tags.filter((tag): tag is DayTag => ['gym', 'cardio', 'rest'].includes(tag)),
          muscles: muscles.filter((muscle): muscle is MuscleGroup => MUSCLE_OPTIONS.some(option => option.key === muscle)),
          note
        }
      }

      logs.value = normalized
    } catch {
      logs.value = {}
    }
  }

  noteInput.value = selectedLog.value.note
})

watch(logs, (value) => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(value))
}, { deep: true })

watch(selectedDateKey, () => {
  noteInput.value = selectedLog.value.note
})

function toDateKey(year: number, monthIndex: number, day: number) {
  const month = String(monthIndex + 1).padStart(2, '0')
  const date = String(day).padStart(2, '0')
  return `${year}-${month}-${date}`
}

function previousMonth() {
  viewedMonth.value = new Date(viewedMonth.value.getFullYear(), viewedMonth.value.getMonth() - 1, 1)
}

function nextMonth() {
  viewedMonth.value = new Date(viewedMonth.value.getFullYear(), viewedMonth.value.getMonth() + 1, 1)
}

function jumpToToday() {
  viewedMonth.value = new Date(today.getFullYear(), today.getMonth(), 1)
  selectedDateKey.value = toDateKey(today.getFullYear(), today.getMonth(), today.getDate())
}

function pickDay(day: number) {
  const year = viewedMonth.value.getFullYear()
  const month = viewedMonth.value.getMonth()
  selectedDateKey.value = toDateKey(year, month, day)
  isModalOpen.value = true
}

function saveNote() {
  const existing = logs.value[selectedDateKey.value] ?? { tags: [], muscles: [], note: '' }
  logs.value[selectedDateKey.value] = {
    ...existing,
    note: noteInput.value.trim()
  }
}

function hasTag(tag: DayTag) {
  return selectedLog.value.tags.includes(tag)
}

function toggleTag(tag: DayTag) {
  const existing = logs.value[selectedDateKey.value] ?? { tags: [], muscles: [], note: '' }
  const tags = new Set(existing.tags)

  if (tags.has(tag)) {
    tags.delete(tag)
  } else {
    tags.add(tag)
    if (tag === 'rest') {
      tags.delete('gym')
      tags.delete('cardio')
    }
    if (tag === 'gym' || tag === 'cardio') {
      tags.delete('rest')
    }
  }

  logs.value[selectedDateKey.value] = {
    ...existing,
    tags: Array.from(tags)
  }
}

function hasMuscle(muscle: MuscleGroup) {
  return selectedLog.value.muscles.includes(muscle)
}

function toggleMuscle(muscle: MuscleGroup) {
  const existing = logs.value[selectedDateKey.value] ?? { tags: [], muscles: [], note: '' }
  const muscles = new Set(existing.muscles)

  if (muscles.has(muscle)) {
    muscles.delete(muscle)
  } else {
    muscles.add(muscle)
  }

  logs.value[selectedDateKey.value] = {
    ...existing,
    muscles: Array.from(muscles)
  }
}

function isToday(day: number) {
  return (
    today.getFullYear() === viewedMonth.value.getFullYear()
    && today.getMonth() === viewedMonth.value.getMonth()
    && today.getDate() === day
  )
}

function isSelected(day: number) {
  const key = toDateKey(viewedMonth.value.getFullYear(), viewedMonth.value.getMonth(), day)
  return selectedDateKey.value === key
}

function tagIcon(tag: DayTag) {
  if (tag === 'gym') return 'gym'
  if (tag === 'cardio') return 'cardio'
  return 'rest'
}

function tagTone(tag: DayTag) {
  if (tag === 'gym') return 'bg-emerald-100 text-emerald-900 border-emerald-300'
  if (tag === 'cardio') return 'bg-sky-100 text-sky-900 border-sky-300'
  return 'bg-amber-100 text-amber-900 border-amber-300'
}

function dayTags(day: number) {
  const key = toDateKey(viewedMonth.value.getFullYear(), viewedMonth.value.getMonth(), day)
  return logs.value[key]?.tags ?? []
}

function dayMuscles(day: number) {
  const key = toDateKey(viewedMonth.value.getFullYear(), viewedMonth.value.getMonth(), day)
  return logs.value[key]?.muscles ?? []
}

function dayNote(day: number) {
  const key = toDateKey(viewedMonth.value.getFullYear(), viewedMonth.value.getMonth(), day)
  return logs.value[key]?.note?.trim() ?? ''
}

function muscleLabel(muscle: MuscleGroup) {
  return MUSCLE_OPTIONS.find(o => o.key === muscle)?.label ?? muscle
}

function closeModal() {
  isModalOpen.value = false
}

function clearSelectedDay() {
  logs.value[selectedDateKey.value] = {
    tags: [],
    muscles: [],
    note: ''
  }
  noteInput.value = ''
}

definePageMeta({
  title: 'Gym Calendar'
})
</script>

<template>
  <div class="h-screen overflow-hidden bg-linear-to-br from-rose-100 via-orange-50 to-cyan-100 px-2 py-2 sm:px-4 sm:py-4 lg:px-6 lg:py-6">
    <div class="mx-auto flex h-full w-full max-w-7xl flex-col rounded-3xl border border-white/70 bg-white/80 p-3 shadow-xl backdrop-blur sm:p-5 lg:p-6">
      <div class="mb-2 flex flex-wrap items-center justify-between gap-2">
        <h1 class="text-2xl font-black tracking-tight text-zinc-900 sm:text-3xl lg:text-4xl">
          Gym Calendar
        </h1>

        <div class="flex items-center gap-2">
          <span class="rounded-lg bg-emerald-100 px-2 py-1 text-xs font-bold text-emerald-800">
            {{ monthSummary.gym }} Gym
          </span>
          <span class="rounded-lg bg-sky-100 px-2 py-1 text-xs font-bold text-sky-800">
            {{ monthSummary.cardio }} Cardio
          </span>
          <span class="rounded-lg bg-amber-100 px-2 py-1 text-xs font-bold text-amber-800">
            {{ monthSummary.rest }} Rest
          </span>

          <div class="ml-1 flex items-center gap-1.5">
            <button
              type="button"
              class="inline-flex h-8 w-8 items-center justify-center rounded-xl border border-zinc-200 bg-white/90 transition hover:bg-rose-50"
              @click="previousMonth"
            >
              <CuteIcon
                name="chevronLeft"
                class="size-5"
              />
            </button>
            <UButton
              color="neutral"
              variant="soft"
              size="sm"
              @click="jumpToToday"
            >
              Today
            </UButton>
            <button
              type="button"
              class="inline-flex h-8 w-8 items-center justify-center rounded-xl border border-zinc-200 bg-white/90 transition hover:bg-rose-50"
              @click="nextMonth"
            >
              <CuteIcon
                name="chevronRight"
                class="size-5"
              />
            </button>
          </div>
        </div>
      </div>

      <div class="mb-2 rounded-xl bg-zinc-900 px-4 py-1.5 text-center">
        <p class="text-sm font-bold text-white sm:text-base">
          {{ monthTitle }}
        </p>
      </div>

      <div class="mb-1 grid grid-cols-7 gap-1 sm:gap-1.5">
        <span
          v-for="dayName in WEEKDAYS"
          :key="dayName"
          class="rounded-lg bg-zinc-100 px-1 py-1 text-center text-[10px] font-bold uppercase tracking-wide text-zinc-600 sm:text-[11px]"
        >
          {{ dayName }}
        </span>
      </div>

      <div class="grid min-h-0 flex-1 auto-rows-fr grid-cols-7 gap-1 sm:gap-1.5">
        <template
          v-for="(day, index) in calendarDays"
          :key="`${day}-${index}`"
        >
          <div
            v-if="day === null"
            class="rounded-2xl"
          />

          <button
            v-else
            type="button"
            class="flex h-full flex-col gap-0.5 rounded-2xl border p-1.5 text-left transition hover:-translate-y-0.5 hover:shadow-md sm:p-2"
            :class="[
              isSelected(day)
                ? 'border-rose-400 bg-linear-to-b from-rose-50 to-white ring-2 ring-rose-300 shadow-sm'
                : 'border-zinc-100 bg-white hover:border-zinc-300 hover:bg-zinc-50/60',
              isToday(day) && !isSelected(day) ? 'border-emerald-300 bg-emerald-50/40' : ''
            ]"
            @click="pickDay(day)"
          >
            <span
              class="text-xs font-black leading-none sm:text-sm"
              :class="isToday(day) ? 'text-emerald-600' : 'text-zinc-800'"
            >
              {{ day }}
            </span>

            <div class="flex flex-wrap gap-0.5">
              <span
                v-for="tag in dayTags(day)"
                :key="tag"
                class="inline-flex items-center gap-0.5 rounded-full border px-1 py-0.5 text-[9px] font-bold leading-none sm:text-[10px]"
                :class="tagTone(tag)"
              >
                <CuteIcon
                  :name="tagIcon(tag)"
                  class="size-3 sm:size-3.5"
                />
                {{ tag }}
              </span>
            </div>

            <div
              v-if="dayMuscles(day).length"
              class="flex flex-wrap gap-0.5"
            >
              <span
                v-for="muscle in dayMuscles(day).slice(0, 3)"
                :key="muscle"
                class="inline-flex items-center rounded-full border border-violet-300 bg-violet-100 px-1 py-0.5 text-[9px] font-bold leading-none text-violet-900 sm:text-[10px]"
              >
                {{ muscleLabel(muscle) }}
              </span>
              <span
                v-if="dayMuscles(day).length > 3"
                class="inline-flex items-center rounded-full border border-zinc-300 bg-zinc-100 px-1 py-0.5 text-[9px] font-bold leading-none text-zinc-600 sm:text-[10px]"
              >
                +{{ dayMuscles(day).length - 3 }}
              </span>
            </div>

            <p
              v-if="dayNote(day)"
              class="mt-auto truncate text-[8px] italic leading-none text-zinc-400 sm:text-[9px]"
            >
              {{ dayNote(day) }}
            </p>
          </button>
        </template>
      </div>
    </div>

    <div
      v-if="isModalOpen"
      class="fixed inset-0 z-50 flex items-end justify-center bg-zinc-950/45 p-3 sm:items-center sm:p-6"
      @click.self="closeModal"
    >
      <div class="max-h-[92vh] w-full max-w-2xl overflow-auto rounded-2xl border border-zinc-200 bg-white p-4 shadow-2xl sm:p-6">
        <div class="mb-4 flex items-start justify-between gap-3">
          <div>
            <h2 class="text-lg font-black text-zinc-900 sm:text-xl">
              {{ selectedDateLabel }}
            </h2>
            <p class="text-sm text-zinc-600">
              Add badges and notes for this day.
            </p>
          </div>
          <UButton
            color="neutral"
            variant="ghost"
            @click="closeModal"
          >
            <CuteIcon
              name="close"
              class="size-6"
            />
          </UButton>
        </div>

        <div class="mb-4 grid grid-cols-1 gap-2 sm:grid-cols-3">
          <button
            type="button"
            class="inline-flex items-center justify-center gap-2 rounded-xl border px-3 py-2.5 text-sm font-bold transition"
            :class="hasTag('gym') ? 'border-emerald-500 bg-emerald-50 text-emerald-900' : 'border-zinc-200 text-zinc-700 hover:border-zinc-400'"
            @click="toggleTag('gym')"
          >
            <CuteIcon
              name="gym"
              class="size-6"
            />
            Gym
          </button>
          <button
            type="button"
            class="inline-flex items-center justify-center gap-2 rounded-xl border px-3 py-2.5 text-sm font-bold transition"
            :class="hasTag('cardio') ? 'border-sky-500 bg-sky-50 text-sky-900' : 'border-zinc-200 text-zinc-700 hover:border-zinc-400'"
            @click="toggleTag('cardio')"
          >
            <CuteIcon
              name="cardio"
              class="size-6"
            />
            Cardio
          </button>
          <button
            type="button"
            class="inline-flex items-center justify-center gap-2 rounded-xl border px-3 py-2.5 text-sm font-bold transition"
            :class="hasTag('rest') ? 'border-amber-500 bg-amber-50 text-amber-900' : 'border-zinc-200 text-zinc-700 hover:border-zinc-400'"
            @click="toggleTag('rest')"
          >
            <CuteIcon
              name="rest"
              class="size-6"
            />
            Rest
          </button>
        </div>

        <p class="mb-2 text-sm font-bold text-zinc-700">
          Muscle Focus
        </p>
        <div class="mb-4 grid grid-cols-2 gap-2 sm:grid-cols-4">
          <button
            v-for="muscle in MUSCLE_OPTIONS"
            :key="muscle.key"
            type="button"
            class="inline-flex items-center justify-center gap-2 rounded-xl border px-2 py-2.5 text-xs font-bold transition sm:text-sm"
            :class="hasMuscle(muscle.key) ? 'border-violet-500 bg-violet-50 text-violet-900' : 'border-zinc-200 text-zinc-700 hover:border-zinc-400'"
            @click="toggleMuscle(muscle.key)"
          >
            {{ muscle.label }}
          </button>
        </div>

        <label
          class="mb-2 block text-sm font-bold text-zinc-700"
          for="day-note"
        >
          Notes
        </label>
        <textarea
          id="day-note"
          v-model="noteInput"
          class="min-h-32 w-full rounded-xl border border-zinc-300 bg-white px-3 py-2 text-sm text-zinc-900 outline-none transition focus:border-rose-400 focus:ring-4 focus:ring-rose-100"
          placeholder="Leg day, PR, how you felt, quick diary..."
          @input="saveNote"
        />

        <div class="mt-4 flex flex-wrap justify-between gap-2">
          <UButton
            color="error"
            variant="soft"
            @click="clearSelectedDay"
          >
            Clear Day
          </UButton>
          <UButton
            color="primary"
            @click="closeModal"
          >
            Done
          </UButton>
        </div>
      </div>
    </div>
  </div>
</template>
