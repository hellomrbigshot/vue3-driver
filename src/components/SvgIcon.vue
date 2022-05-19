<script setup lang="ts">
import { computed } from 'vue'
interface Props {
  iconClass: string
  size?: string | number,
  width?: string | number,
  height?: string | number
}
const props = withDefaults(defineProps<Props>(), { size: 16, color: '' })
const iconName = computed(() => `#icon-${props.iconClass}`)
</script>
<template>
  <svg
    class="svg-icon" :style="{ width: `${width || size}px`, height: `${height || size}px` }"
    fill="currentColor"
    aria-hidden="true"
  >
    <use :xlink:href="iconName"/>
  </svg>
</template>
