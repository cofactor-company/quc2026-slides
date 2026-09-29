<!-- Small Cofactor wordmark in the bottom-right corner of every slide except the cover (which shows a large one).
     Turn it off for a deck with `cofactorFooter: false` in its headmatter, or for one slide in its frontmatter.
     The bottom-left corner links to the current slide on the published site, so screenshots can be traced back to it.
     The deck's folder on the site is its `exportFilename`. Turn it off the same way with `urlFooter: false`. -->
<script setup lang="ts">
import { computed } from 'vue'
import { useSlideContext } from '@slidev/client'

const { $page, $frontmatter, $slidev } = useSlideContext()

const slideUrl = computed(() => `cofactor-company.github.io/quc2026-slides/${$slidev.configs.exportFilename}/#/${$page.value}`)
</script>

<template>
  <CofactorLogo
    v-if="$slidev.configs.cofactorFooter !== false && $frontmatter.cofactorFooter !== false && $page !== 1 && $frontmatter.layout !== 'cover'"
    class="absolute bottom-3 right-6 text-sm opacity-70"
  />
  <a
    v-if="$slidev.configs.urlFooter !== false && $frontmatter.urlFooter !== false && $page !== 1 && $frontmatter.layout !== 'cover'"
    :href="`https://${slideUrl}`"
    class="url-footer absolute bottom-3 left-6 text-[10px] leading-none opacity-50"
  >
    {{ slideUrl }}
  </a>
</template>
