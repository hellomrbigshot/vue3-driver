<script setup lang="ts">
import { onMounted, ref, nextTick } from 'vue'
import Driver from './components/Driver.vue'
const driverRef = ref()
const showStep2 = ref(false)
const steps = [
  {
    element: '#step1',
    parentElement: '#stepParent',
    popover: {
      description: '点击显示菜单2',
    },
    onNext: async () => {
      return new Promise((resolve, _) => {
        toggleStep2()
        nextTick(() => {
          resolve(true)
        })
      })
    }
  },
  {
    element: '#step2',
    parentElement: '#stepParent',
    popover: {
      description: '后面还有一个菜单3'
    }
  },
  {
    element: '#step3',
    parentElement: '#stepParent',
    popover: {
      description: '提示结束了'
    }
  }
]
const toggleStep2 = () => {
  showStep2.value = !showStep2.value
}
onMounted(() => {
  nextTick(() => {
    driverRef.value?.show()
  })
})
</script>

<template>
  <div class="flex justify-center mt-10 mb-10">
    <img class="block" alt="Vue logo" src="./assets/logo.png" width="200" height="200"/>
  </div>
  <div id="stepParent" class="flex">
    <div id="step1" class="px-4 py-2 bg-blue-400 text-xs rounded text-white cursor-pointer" @click="toggleStep2">菜单1</div>
    <div v-show="showStep2" id="step2" class="px-4 py-2 ml-4 bg-red-400 text-xs rounded text-white">菜单2</div>
    <div id="step3" class="px-4 py-2 ml-4 bg-lime-400 text-xs rounded text-white">菜单3</div>
    <Driver ref="driverRef" :steps="steps"></Driver>
  </div>
</template>
