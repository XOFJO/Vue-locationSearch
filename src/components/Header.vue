<script setup>
import { Icon } from '@iconify/vue';
</script>

<template>
  <footer className="footer items-center p-4 bg-neutral text-neutral-content">
    <div className="items-center grid-flow-col">
      <h1 class="text-xl font-bold">Location Search App</h1>
    </div>
    <div
      className="grid-flow-col gap-4 md:place-self-center md:justify-self-end"
    >
      <a
        class="cursor-pointer"
        href="https://github.com/XOFJO/Vue-locationSearch"
        target="_blank"
        ><Icon icon="mdi:github" width="35" />
      </a>
    </div>
  </footer>
</template>

<style lang="scss" scoped></style>
