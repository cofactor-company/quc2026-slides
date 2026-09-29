<!-- Cofactor wordmark, as on https://cofactor.fi/: "Cofactor" with a cyan full stop, linking to the website. Size it with a text-* class. -->
<template>
  <a class="cofactor-logo" href="https://cofactor.fi/" target="_blank" rel="noopener">Cofactor<span class="cofactor-logo-dot">.</span></a>
</template>

<style scoped>
/* Undo the global link styling (accent colour, dashed underline) */
.cofactor-logo,
.cofactor-logo:hover {
  font-weight: 700;
  letter-spacing: -0.01em;
  color: var(--cofactor-fg);
  border-bottom: none;
  text-decoration: none;
  white-space: nowrap;
}

.cofactor-logo-dot {
  color: var(--cofactor-cyan);
}
</style>
