<template>
  <!-- Page number on every slide but the title. `$nav.total` counts only
       visible slides, so a slide carrying `hide: true` is excluded from
       both the number and the total. -->
  <footer
    v-if="$nav.currentPage > 1"
    class="absolute bottom-3 right-4 text-xs opacity-40"
  >
    {{ $nav.currentPage }} / {{ $nav.total }}
  </footer>
</template>
