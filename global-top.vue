<!-- Pace timer for rehearsing and giving a talk. Only in `slidev` dev mode, never in the built site or exports.
     Turn it on for a deck with `paceTimer: true` in its headmatter.
     It starts when you leave the first slide and survives page reloads.
     Plan the talk in sections: `pace: <minutes>` in a slide's frontmatter starts a section that lasts that long,
     until the next slide with `pace`. Sections follow each other, so changing one length moves only the ones after it.
     `paceAt: <minutes from start>` pins a section to a fixed time (e.g. a break), so the sections after it don't move,
     and the gap before it is spare time. Within a section the plan is spread evenly over its slides.
     The timer shows how far ahead (green) or behind (amber/red) you are, and how long until the next section (hover it for the section's title).
     Click the time to pause or resume. The ↺ button resets it, and it starts again on the next slide change.
     Scroll the mouse wheel over it to jump the time by a minute (5 with shift), for testing or catching up after a restart. -->
<script setup lang="ts">
import { computed, onBeforeUnmount, ref, watch } from 'vue'
import { configs, useNav } from '@slidev/client'

interface PaceState { start: number, pausedAt: number | null, pausedTotal: number }

const { slides, currentSlideNo, isPrintMode, isEmbedded } = useNav()

const enabled = import.meta.env.DEV && (configs as any).paceTimer === true
const storageKey = `pace-timer:${configs.exportFilename ?? configs.title}`

function load(): PaceState | null {
  try {
    const raw = localStorage.getItem(storageKey)
    return raw ? JSON.parse(raw) : null
  }
  catch {
    return null
  }
}

const state = ref<PaceState | null>(load())
watch(state, (s) => {
  try {
    if (s)
      localStorage.setItem(storageKey, JSON.stringify(s))
    else
      localStorage.removeItem(storageKey)
  }
  catch {}
}, { deep: true })

watch(currentSlideNo, (no: number) => {
  if (enabled && !state.value && no > 1)
    state.value = { start: Date.now(), pausedAt: null, pausedTotal: 0 }
})

const now = ref(Date.now())
const tick = setInterval(() => now.value = Date.now(), 1000)
onBeforeUnmount(() => clearInterval(tick))

const elapsedMin = computed(() => {
  const s = state.value
  if (!s)
    return 0
  const end = s.pausedAt ?? now.value
  return (end - s.start - s.pausedTotal) / 60000
})

interface Section { no: number, min: number, length: number, title: string }

// Sections in slide order, each starting where the previous one ends unless pinned with `paceAt`
const checkpoints = computed(() => {
  const sections: Section[] = []
  let at = 0
  for (const r of slides.value) {
    const fm = r.meta?.slide?.frontmatter ?? {}
    if (typeof fm.pace !== 'number' && typeof fm.paceAt !== 'number')
      continue
    if (typeof fm.paceAt === 'number') {
      if (import.meta.env.DEV && fm.paceAt < at)
        console.warn(`[pace-timer] sections before slide ${r.no} overrun its paceAt: ${fm.paceAt} by ${at - fm.paceAt} min`)
      at = fm.paceAt
    }
    const length = typeof fm.pace === 'number' ? fm.pace : 0
    sections.push({ no: r.no, min: at, length, title: r.meta.slide.title ?? `Slide ${r.no}` })
    at += length
  }
  return sections
})

const plan = computed(() => {
  const no = currentSlideNo.value
  const points = checkpoints.value
  const prev = [...points].reverse().find(p => p.no <= no)
  const next = points.find(p => p.no > no)
  if (!prev)
    return { planned: 0, next }
  const slidesInSection = (next?.no ?? slides.value.length + 1) - prev.no
  return { planned: prev.min + prev.length * (no - prev.no) / slidesInSection, next }
})

const delta = computed(() => elapsedMin.value - plan.value.planned)
const untilNext = computed(() => Math.round((plan.value.next?.min ?? 0) - elapsedMin.value))

const status = computed(() => {
  if (delta.value > 5)
    return 'late'
  if (delta.value > 2)
    return 'slipping'
  return 'ok'
})

function clock(min: number) {
  const total = Math.max(0, Math.floor(min * 60))
  const h = Math.floor(total / 3600)
  const m = Math.floor(total / 60) % 60
  const s = total % 60
  const mm = String(m).padStart(h ? 2 : 1, '0')
  return `${h ? `${h}:` : ''}${mm}:${String(s).padStart(2, '0')}`
}

function onWheel(e: WheelEvent) {
  // Shift+wheel scrolls sideways in most browsers
  const delta = e.deltaY || e.deltaX
  if (!delta)
    return
  const step = (e.shiftKey ? 5 : 1) * 60000 * (delta < 0 ? 1 : -1)
  const s = state.value ?? (state.value = { start: Date.now(), pausedAt: null, pausedTotal: 0 })
  // Moving the start back adds elapsed time, but never below zero
  s.start = Math.min(s.start - step, (s.pausedAt ?? Date.now()) - s.pausedTotal)
}

function togglePause() {
  const s = state.value
  if (!s)
    return
  if (s.pausedAt) {
    s.pausedTotal += Date.now() - s.pausedAt
    s.pausedAt = null
  }
  else {
    s.pausedAt = Date.now()
  }
}
</script>

<template>
  <div
    v-if="enabled && !isPrintMode && !isEmbedded"
    class="pace-timer"
    :class="[status, { paused: state?.pausedAt }]"
    @wheel.prevent.stop="onWheel"
  >
    <template v-if="state">
      <span class="time" title="Pause/resume" @click="togglePause">{{ clock(elapsedMin) }}</span>
      <span v-if="checkpoints.length" class="delta">{{ Math.round(delta) > 0 ? '+' : Math.round(delta) < 0 ? '−' : '±' }}{{ Math.abs(Math.round(delta)) }}m</span>
      <span v-if="plan.next" class="next" :title="plan.next.title">{{ untilNext >= 0 ? `next in ${untilNext}m` : `next ${-untilNext}m overdue` }}</span>
      <button class="reset" title="Reset timer" @click="state = null">↺</button>
    </template>
    <span v-else>timer starts on next slide</span>
  </div>
</template>

<style scoped>
.pace-timer {
  position: fixed; top: 0.4rem; right: 0.6rem; z-index: 100;
  display: flex; gap: 0.6rem; align-items: baseline;
  font-family: var(--slidev-code-font-family, monospace); font-size: 11px; font-variant-numeric: tabular-nums;
  padding: 0.15rem 0.5rem; border-radius: 0.3rem; background: rgba(0, 0, 0, 0.35);
  opacity: 0.55; user-select: none;
}
.pace-timer:hover { opacity: 1; }
.time { cursor: pointer; }
.reset { cursor: pointer; opacity: 0.7; padding: 0 0.15rem; }
.reset:hover { opacity: 1; color: var(--cofactor-accent); }
.delta { font-weight: 700; color: #4ade80; }
.slipping .delta { color: #fbbf24; }
.late .delta { color: #f87171; }
.next { opacity: 0.7; }
.paused { animation: blink 1s steps(2) infinite; }
@keyframes blink { 50% { opacity: 0.2; } }
</style>
