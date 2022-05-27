<script setup lang="ts">
import { computed, ref, toRefs, defineExpose, watch } from 'vue'
import SvgIcon from './SvgIcon.vue'
interface IStep {
  element: string,
  parentElement?: string,
  popover: {
    description: string
  },
  onNext?: (() => void) | (() => Promise<boolean>)
}
const props = defineProps<{ steps: Array<IStep> }>()
const { steps } = toRefs(props)
const currentStepIndex = ref(-1)
const currentStep = computed(() => steps.value[currentStepIndex.value])
const teleportParent = computed(() => currentStep.value?.parentElement || 'body')
const showDriver = ref(false)

let replaceElement: Node | null = null
let overlayElement: Node | null = null
let currentElement: HTMLElement | null = null
const positionInfo = computed(() => {
  if (!currentStep.value) return null
  currentElement = document.querySelector(currentStep.value.element)
  if (currentElement) {
    // 高亮元素会脱离文档，所以原来的位置需要添加一个占位元素防止布局漂移
    replaceElement = currentElement.cloneNode(true);
    (replaceElement as HTMLElement).removeAttribute('id')
    
    // 计算选中元素相对于屏幕的位置和元素大小
    const { width, height, top, left } = currentElement.getBoundingClientRect()
    currentElement.dataset.oldWidth = currentElement.style.width
    currentElement.dataset.oldHeight = currentElement.style.height
    currentElement.dataset.oldTop = currentElement.style.top
    currentElement.dataset.oldLeft = currentElement.style.left
    currentElement.dataset.oldRight = currentElement.style.right
    currentElement.dataset.oldBottom = currentElement.style.bottom
    currentElement.dataset.oldBoxSizing = currentElement.style.boxSizing

    // 选中元素上添加透明遮罩，防止触发事件
    overlayElement = document.createElement('div');
    (overlayElement as HTMLElement).classList.add('driver-select-item-overlay');
    (overlayElement as HTMLElement).style.width = `${width}px`;
    (overlayElement as HTMLElement).style.height = `${height}px`;
    (overlayElement as HTMLElement).style.top = `${top}px`;
    (overlayElement as HTMLElement).style.left = `${left}px`;

    // 选中元素改变样式
    currentElement.setAttribute('driver-type', 'select-item')
    currentElement.style.top = `${top}px`
    currentElement.style.left = `${left}px`
    currentElement.style.width = `${width}px`
    currentElement.style.height = `${height}px`
    currentElement.style.boxSizing = 'border-box';

    // 插入占位元素和透明遮罩
    (currentElement.parentNode || document.body).insertBefore(replaceElement, currentElement);
    (currentElement.parentNode || document.body).insertBefore(overlayElement, currentElement);
    return {
      width,
      height,
      top: top,
      left: left
    }
  } else {
    close()
    return null
  }
})
const stageStyle = computed(() => {
  if (!positionInfo.value) return {}
  const { width, height, top, left } = positionInfo.value
  return {
      width: `${width}px`,
      height: `${height}px`,
      top: `${top}px`,
      left: `${left}px`
    }
})
const popoverStyle = computed(() => {
  if (!positionInfo.value) return {}
  const { width, height, top, left } = positionInfo.value
  return {
      top: `${top + height + 20}px`,
      left: `${left + width + 20}px`
    }
})
const isLastStep = computed(() => currentStepIndex.value === steps.value.length - 1)
const handleToNextStep = async () => {
  await currentStep.value.onNext?.()
  initEle()
  if (!isLastStep.value) {
    currentStepIndex.value ++
  } else {
    close()
  }
}
const close = () => {
  initEle()
  showDriver.value = false
}

// 初始化 dom
const initEle = () => {
  if (currentElement && replaceElement && overlayElement) {
    const { oldWidth = '', oldHeight = '', oldTop = '', oldLeft = '', oldRight = '', oldBottom = '', oldBoxSizing = '' } = currentElement.dataset
    currentElement.removeAttribute('driver-type');
    currentElement.style.top = oldTop
    currentElement.style.right = oldRight 
    currentElement.style.bottom = oldBottom
    currentElement.style.left = oldLeft
    currentElement.style.width = oldWidth
    currentElement.style.height = oldHeight
    currentElement.style.boxSizing = oldBoxSizing
    delete currentElement.dataset.oldTop
    delete currentElement.dataset.oldRight
    delete currentElement.dataset.oldBottom
    delete currentElement.dataset.oldLeft
    delete currentElement.dataset.oldWidth
    delete currentElement.dataset.oldHeight
    delete currentElement.dataset.boxSizing
    currentElement?.parentNode?.removeChild(replaceElement) // 移除 replaceDom
    currentElement?.parentNode?.removeChild(overlayElement) // 移动透明遮罩
  }
}
const show = () => {
  currentStepIndex.value = 0
  showDriver.value = true
}
defineExpose({
  close,
  show
})
</script>
<template>
  <teleport :to="teleportParent">
    <div v-show="showDriver" class="driver-page-overlay fixed opacity-75 transition-all ease-in-out duration-100 inset-0 w-full h-full bg-black"></div>
  </teleport>
  <teleport :to="teleportParent">
    <div v-show="showDriver" class="driver-highlighted-element-stage rounded fixed block bg-white transition-all ease-in duration-100" :style="stageStyle"></div>
  </teleport>
  <teleport :to="teleportParent">
    <div v-show="showDriver" class="driver-popover fixed bg-B-01 rounded p-4 transition-all ease-in duration-100" :style="popoverStyle">
      <SvgIcon
        class="absolute -top-10 -left-8 text-white z-10"
        icon-class="arrow"
        width="30"
        height="50"
      />
      <div class="text-xs text-white opacity-50">小提示 {{ `${currentStepIndex + 1}/${steps.length}` }}</div>
      <div class="mt-1 text-white">{{ currentStep && currentStep.popover && currentStep.popover.description || '' }}</div>
      <div class="mt-4 flex">
        <div class="text-xs flex-1 py-1.5 text-center rounded text-white cursor-pointer" style="background: rgba(255, 255, 255, .1);" @click="close">跳过教程</div>
        <div v-if="!isLastStep" class="text-xs flex-1 ml-2.5 py-1.5 text-center rounded bg-white text-B-01 cursor-pointer" @click="handleToNextStep">下一步</div>
        <div v-else class="text-xs flex-1 ml-2.5 py-1.5 text-center rounded bg-white text-B-01 cursor-pointer" @click="close">完成</div>
      </div>
    </div>
  </teleport>
</template>
<style lang="scss">
.driver-page-overlay {
  z-index: 10000;
}
.driver-highlighted-element-stage {
  z-index: 10001;
}
[driver-type="select-item"] {
  z-index: 10002 !important;
  position: fixed !important;
  margin: 0 !important;
  box-sizing: border-box !important;
}
.driver-select-item-overlay {
  z-index: 10003;
  position: fixed;
  opacity: 0;
}
.driver-popover {
  z-index: 10004 !important;
  min-width: 208px;
}
</style>
