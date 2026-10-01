<script setup lang="ts">
// Overrides Slidev's error layout, shown when a slide chunk fails to load.
// On GitHub Pages this happens mostly after a redeploy: a page loaded earlier
// (or a cached index.html, Pages caches for 10 min) asks for chunk hashes that
// no longer exist. A cache-busting reload fetches the current build.
import { onMounted } from 'vue'

const RELOAD_KEY = 'slide-error-reloaded-at'
const hasServer = __SLIDEV_HAS_SERVER__

function reload() {
  const url = new URL(location.href)
  url.searchParams.set('r', Date.now().toString())
  location.replace(url.toString())
}

onMounted(() => {
  // Offline, a reload would replace the deck with the browser's error page
  if (hasServer || !navigator.onLine)
    return
  try {
    // Reload once per 30 s, so a persistent failure doesn't loop
    const last = Number(sessionStorage.getItem(RELOAD_KEY) ?? 0)
    if (Date.now() - last < 30_000)
      return
    sessionStorage.setItem(RELOAD_KEY, Date.now().toString())
  }
  catch {
    return
  }
  reload()
})
</script>

<template>
  <div class="slidev-layout h-full flex flex-col items-center justify-center gap-4 text-center">
    <template v-if="hasServer">
      <div class="text-red-500 font-bold font-mono">
        An error occurred on this slide. Check the terminal for more information.
      </div>
    </template>
    <template v-else>
      <div class="opacity-80">
        Couldn't load this slide. The slides may have been updated, or the connection dropped.
      </div>
      <button class="reload" @click="reload">
        Reload slides
      </button>
    </template>
  </div>
</template>

<style scoped>
.reload {
  background: var(--cofactor-bg-dark);
  color: var(--cofactor-accent);
  border: 1px solid var(--cofactor-accent);
  border-radius: 0.5rem;
  padding: 0.5rem 1.25rem;
  cursor: pointer;
}
</style>
