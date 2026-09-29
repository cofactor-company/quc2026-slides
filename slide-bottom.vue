<!-- Small Cofactor wordmark in the bottom-right corner of every slide except the cover (which shows a large one).
     Turn it off for a deck with `cofactorFooter: false` in its headmatter, or for one slide in its frontmatter. -->
<script setup lang="ts">
import { useSlideContext } from '@slidev/client'

const { $page, $frontmatter, $slidev } = useSlideContext()
</script>

<template>
  <CofactorLogo
    v-if="$slidev.configs.cofactorFooter !== false && $frontmatter.cofactorFooter !== false && $page !== 1 && $frontmatter.layout !== 'cover'"
    class="absolute bottom-4 right-6 text-sm opacity-70"
  />
</template>
