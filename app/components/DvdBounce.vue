<script setup lang="ts">
import confetti from 'canvas-confetti'

const colorMode = useColorMode()

const MAX_LOGO_WIDTH = 220
// Aspect ratio derived from the SVG viewBox dimensions (465.84 / 1058.4)
const LOGO_ASPECT = 465.84 / 1058.4
const BASE_SPEED = 2.8

// Scales with the viewport, capped at MAX_LOGO_WIDTH
const logoWidth = ref(MAX_LOGO_WIDTH)
const logoHeight = computed(() => Math.round(logoWidth.value * LOGO_ASPECT))

const x = ref(120)
const y = ref(80)
// Non-45° angle prevents the logo from bouncing in a visually repetitive straight diagonal
let velocityX = BASE_SPEED
let velocityY = BASE_SPEED * 0.72

const speed = ref(1)
const rainbow = ref(false)
let rainbowHue = 0
const logoColor = ref('#ffffff')
const bounces = ref(0)
const corners = ref(0)
const showCornerFlash = ref(false)

let cornerFlashTimer: ReturnType<typeof setTimeout> | null = null
let animationFrameId: number | null = null
let viewportWidth = 0
let viewportHeight = 0

// Rescales velocity to match the current speed multiplier while preserving direction
function normalizeVelocity() {
  const magnitude = Math.sqrt(velocityX * velocityX + velocityY * velocityY)
  if (magnitude === 0) return
  const target = BASE_SPEED * speed.value
  velocityX = (velocityX / magnitude) * target
  velocityY = (velocityY / magnitude) * target
}

// Only recalculate when speed changes, not on every animation frame
watch(speed, normalizeVelocity)

function fireConfetti() {
  const defaults = {
    spread: 360,
    ticks: 50,
    gravity: 0,
    decay: 0.94,
    startVelocity: 30,
    origin: { x: x.value <= 0 ? 0.05 : 0.95, y: y.value <= 0 ? 0.05 : 0.95 },
    colors: ['#FFE400', '#FFBD00', '#E89400', '#FFCA6C', '#FDFFB8'],
  }

  const shoot = () => {
    confetti({ ...defaults, particleCount: 40, scalar: 1.2, shapes: ['star'] })
    confetti({ ...defaults, particleCount: 10, scalar: 0.75, shapes: ['circle'] })
  }

  setTimeout(shoot, 0)
  setTimeout(shoot, 100)
  setTimeout(shoot, 200)
}

function tick() {
  let hitHorizontal = false
  let hitVertical = false

  x.value += velocityX
  y.value += velocityY

  if (x.value <= 0) {
    x.value = 0
    velocityX = Math.abs(velocityX)
    hitHorizontal = true
  } else if (x.value >= viewportWidth - logoWidth.value) {
    x.value = viewportWidth - logoWidth.value
    velocityX = -Math.abs(velocityX)
    hitHorizontal = true
  }

  if (y.value <= 0) {
    y.value = 0
    velocityY = Math.abs(velocityY)
    hitVertical = true
  } else if (y.value >= viewportHeight - logoHeight.value) {
    y.value = viewportHeight - logoHeight.value
    velocityY = -Math.abs(velocityY)
    hitVertical = true
  }

  if (hitHorizontal || hitVertical) {
    bounces.value++
    // Larger hue jump on bounce makes the color shift more noticeable than the continuous drift
    if (rainbow.value) rainbowHue = (rainbowHue + 55) % 360
  }

  // Corner hit: both walls touched in the same frame
  if (hitHorizontal && hitVertical) {
    corners.value++
    showCornerFlash.value = true
    if (cornerFlashTimer) clearTimeout(cornerFlashTimer)
    cornerFlashTimer = setTimeout(() => {
      showCornerFlash.value = false
    }, 1500)
    fireConfetti()
  }

  if (rainbow.value) {
    if (!hitHorizontal && !hitVertical) rainbowHue = (rainbowHue + 0.45) % 360
    logoColor.value = `hsl(${rainbowHue}deg 85% 65%)`
  }

  animationFrameId = requestAnimationFrame(tick)
}

function updateViewport() {
  viewportWidth = window.innerWidth
  viewportHeight = window.innerHeight
  logoWidth.value = Math.min(MAX_LOGO_WIDTH, Math.round(viewportWidth * 0.35))
  x.value = Math.min(x.value, viewportWidth - logoWidth.value)
  y.value = Math.min(y.value, viewportHeight - logoHeight.value)
}

function getDefaultColor() {
  return colorMode.value === 'dark' ? '#ffffff' : '#111111'
}

onMounted(() => {
  updateViewport()
  logoColor.value = getDefaultColor()
  animationFrameId = requestAnimationFrame(tick)
  window.addEventListener('resize', updateViewport)
})

onUnmounted(() => {
  if (animationFrameId) cancelAnimationFrame(animationFrameId)
  window.removeEventListener('resize', updateViewport)
  if (cornerFlashTimer) clearTimeout(cornerFlashTimer)
})

watch(
  () => colorMode.value,
  () => {
    if (!rainbow.value) logoColor.value = getDefaultColor()
  },
)

function handleDecreaseSpeed() {
  if (speed.value > 0.25) speed.value = Math.round((speed.value - 0.25) * 100) / 100
}

function handleIncreaseSpeed() {
  if (speed.value < 10) speed.value = Math.round((speed.value + 0.25) * 100) / 100
}

function handleToggleRainbow() {
  rainbow.value = !rainbow.value
  if (!rainbow.value) logoColor.value = getDefaultColor()
}

function handleSetTheme(theme: 'dark' | 'light') {
  // Set color directly instead of relying on the colorMode watcher, as
  // colorMode.value may not update synchronously after changing the preference
  colorMode.preference = theme
  if (!rainbow.value) {
    logoColor.value = theme === 'dark' ? '#ffffff' : '#111111'
  }
}
</script>

<template>
  <!-- Uses transform instead of left/top to avoid layout recalculations on every frame -->
  <div
    class="absolute top-0 left-0"
    style="will-change: transform"
    :style="{ transform: `translate(${x}px, ${y}px)`, color: logoColor, width: `${logoWidth}px` }"
  >
    <DvdLogo />
  </div>

  <!-- Corner flash -->
  <Transition name="fade">
    <div
      v-if="showCornerFlash"
      class="fixed inset-0 flex items-center justify-center gap-3 font-mono font-bold text-4xl tracking-widest pointer-events-none text-[#111] dark:text-white"
    >
      <UIcon name="tabler:star" class="size-8" />
      CORNER!
      <UIcon name="tabler:star" class="size-8" />
    </div>
  </Transition>

  <!-- Settings -->
  <SettingsPopover
    :speed="speed"
    :rainbow="rainbow"
    :bounces="bounces"
    :corners="corners"
    @decrease-speed="handleDecreaseSpeed"
    @increase-speed="handleIncreaseSpeed"
    @toggle-rainbow="handleToggleRainbow"
    @set-theme="handleSetTheme"
  />
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
