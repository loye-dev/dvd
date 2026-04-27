<script setup lang="ts">
import NumberFlow from '@number-flow/vue'

const props = defineProps<{
  speed: number
  rainbow: boolean
  bounces: number
  corners: number
}>()

const emit = defineEmits<{
  'decrease-speed': []
  'increase-speed': []
  'toggle-rainbow': []
  'set-theme': [theme: 'dark' | 'light']
}>()

const colorMode = useColorMode()
const isDark = computed(() => colorMode.value === 'dark')

const isStatsStacked = computed(() => props.bounces >= 1000 || props.corners >= 1000)

const setTheme = (theme: 'dark' | 'light') => emit('set-theme', theme)
</script>

<template>
  <UPopover placement="bottom-end" class="fixed top-4 right-4">
    <template #default="{ open }">
      <UButton
        variant="ghost"
        color="neutral"
        icon="tabler:settings"
        class="transition-opacity"
        :class="open ? 'opacity-80' : 'opacity-15 hover:opacity-80'"
      />
    </template>
    <template #content>
      <div class="w-48 p-4 flex flex-col gap-3">
        <div class="flex flex-col gap-1.5">
          <span
            class="font-bold text-[10px] leading-none font-mono tracking-widest uppercase opacity-30"
            >SPEED</span
          >
          <div class="flex gap-1.5 items-center">
            <UButton
              variant="ghost"
              size="xs"
              icon="tabler:minus"
              class="flex-1 justify-center"
              @click="emit('decrease-speed')"
            />
            <NumberFlow
              :value="speed"
              suffix="×"
              class="w-10 text-center font-mono text-[13px] opacity-70 leading-none"
            />
            <UButton
              variant="ghost"
              size="xs"
              icon="tabler:plus"
              class="flex-1 justify-center"
              @click="emit('increase-speed')"
            />
          </div>
        </div>

        <div class="flex flex-col gap-1.5">
          <span
            class="font-bold text-[10px] leading-none font-mono tracking-widest uppercase opacity-30"
            >COLOR</span
          >
          <UButton
            :variant="rainbow ? 'soft' : 'ghost'"
            size="sm"
            :icon="rainbow ? 'tabler:rainbow' : 'tabler:rainbow-off'"
            block
            @click="emit('toggle-rainbow')"
          >
            Rainbow
          </UButton>
        </div>

        <div class="flex flex-col gap-1.5">
          <span
            class="font-bold text-[10px] leading-none font-mono tracking-widest uppercase opacity-30"
            >THEME</span
          >
          <div class="flex gap-1.5">
            <UButton
              :variant="isDark ? 'soft' : 'ghost'"
              size="sm"
              icon="tabler:moon"
              class="flex-1"
              @click="setTheme('dark')"
              >Dark</UButton
            >
            <UButton
              :variant="!isDark ? 'soft' : 'ghost'"
              size="sm"
              icon="tabler:sun"
              class="flex-1"
              @click="setTheme('light')"
              >Light</UButton
            >
          </div>
        </div>

        <UDivider />

        <div class="flex flex-col gap-1.5">
          <span class="font-bold text-[10px] font-mono tracking-widest uppercase opacity-30"
            >STATS</span
          >
          <div class="grid gap-2" :class="isStatsStacked ? 'grid-cols-1' : 'grid-cols-2'">
            <div class="rounded-lg py-2.5 px-1.5 text-center bg-black/5 dark:bg-white/5">
              <NumberFlow :value="bounces" class="font-mono font-bold text-2xl leading-none mb-1" />
              <div class="font-mono text-[9px] uppercase tracking-widest opacity-40">BOUNCES</div>
            </div>
            <div class="rounded-lg py-2.5 px-1.5 text-center bg-black/5 dark:bg-white/5">
              <NumberFlow :value="corners" class="font-mono font-bold text-2xl leading-none mb-1" />
              <div class="font-mono text-[9px] uppercase tracking-widest opacity-40">CORNERS</div>
            </div>
          </div>
        </div>

        <UButton
          block
          variant="link"
          size="sm"
          icon="simple-icons:github"
          href="https://github.com/loye-dev/dvd"
          target="_blank"
          rel="noopener noreferrer"
          class="opacity-30 hover:opacity-90"
        >
          GitHub
        </UButton>
      </div>
    </template>
  </UPopover>
</template>
