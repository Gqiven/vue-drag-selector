<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import { Coord } from './type'

type DefaultProps = {
  boxColor?: string
}

const props = defineProps<DefaultProps>()

const selectedNodeKeys = ref<string[]>([])
const preSelectedNodeKeys = ref<string[]>([])
const containerWrapRef = ref()
const containerRef = ref()
const startCoord = ref<Coord>({
  x: 0,
  y: 0
})
const endCoord = ref<Coord>({
  x: 0,
  y: 0
})
const selectingBoxRef = ref()
const containerRect = computed(() => containerRef.value?.getBoundingClientRect())
const childNodes = computed<Element[]>(() => [...containerRef.value.childNodes])
const boxColor = computed(() => props.boxColor || 'rgba(0, 184, 92, .5)')
// const childNodeKeyMap = computed(() => {
//   return ([...childNodes.value]).reduce((res, node) => {
//     if(node && node.getAttribute) {
//       res[node.getAttribute('data-key') as string] = node
//     }
    
//     return res
//   }, {} as {[key: string]: Element})
// })

const getCoords = (e: MouseEvent) => ({
  x: e.clientX - containerRect.value.left,
  y: e.clientY - containerRect.value.top
});

const getDimensions = (startCoord: Coord, endCoord: Coord) => {
  return {
    width: Math.abs(startCoord.x - endCoord.x),
    height: Math.abs(startCoord.y - endCoord.y)
  }
}

const collisionCheck = (node1: DOMRect, node2: DOMRect) => {
  return node1.left < node2.left + node2.width &&
  node1.left + node1.width > node2.left &&
  node1.top < node2.top + node2.height &&
  node1.top + node1.height > node2.top;
}

const getDiffNodeKeys = (preNodeKeys: string[], curNodeKeys: string[]) => {
  const resultNodeKeys = [] as string[]
  const nodeKeys = preNodeKeys.concat(curNodeKeys)
  const map = new Map<string, boolean>([])
  nodeKeys.forEach((key) => {
    if(map.has(key)) {
      map.set(key, !map.get(key))
    }else {
      map.set(key, true)
    }
  })
  const entries = [...map.entries()]
  entries.forEach(([key, value]) => {
    !!value && resultNodeKeys.push(key)
  })
  return resultNodeKeys
}

const renderSelectedNodes = (selectedNodeKeys: string[]) => {
  for(let i = 0, len = childNodes.value.length; i < len; i ++) {
    const node = childNodes.value[i] as any
    const key = node && node.getAttribute && node.getAttribute('data-key')
    if(key !== void 0) {
      if(selectedNodeKeys.includes(key)) {
        node.style.backgroundColor = 'rgba(0, 184, 92, .7)'
        node.style.color = '#fff'
      }else {
        node.style.backgroundColor = 'transparent'
        node.style.color = '#213547'
      }
    }
    
  }
}

const intersection = () => {
  const boxRect = selectingBoxRef.value.getBoundingClientRect()
  const intersectionNodeKeys = [] as string[]
  childNodes.value.forEach(node => {
    if(node.getBoundingClientRect && collisionCheck(boxRect, node.getBoundingClientRect())) {
      intersectionNodeKeys.push(node.getAttribute('data-key') as string)
    }
  })
  selectedNodeKeys.value = getDiffNodeKeys(preSelectedNodeKeys.value, intersectionNodeKeys)
  renderSelectedNodes(selectedNodeKeys.value)
}

// mouse
const handleStartDrag = (e: MouseEvent) => {
  startCoord.value = getCoords(e)
  selectingBoxRef.value.style.top = `${startCoord.value.y}px`
  selectingBoxRef.value.style.left = `${startCoord.value.x}px`

  document.addEventListener('mousemove', handleDrag)
  // document.addEventListener('touch', handleTouchMove)

}

const handleDrag = (e: MouseEvent) => {
  endCoord.value = getCoords(e)
  const dimensions = getDimensions(startCoord.value, endCoord.value)

  if(endCoord.value.x < startCoord.value.x) {
    selectingBoxRef.value.style.left = `${endCoord.value.x}px`
  }

  if(endCoord.value.y < startCoord.value.y) {
    selectingBoxRef.value.style.top = `${endCoord.value.y}px`
  }
  selectingBoxRef.value.style.width = dimensions.width + "px";
  selectingBoxRef.value.style.height = dimensions.height + "px";

  // intersection()
}

// touch
const handleTouchStart = (e: TouchEvent) =>{}
const handleTouchMove = () => {}

const handleEndDrag = () => {
  intersection()
  startCoord.value = {
    x: 0,
    y: 0
  }
  endCoord.value = {
    x: 0,
    y: 0
  }
  selectingBoxRef.value.style.width = 0
  selectingBoxRef.value.style.height = 0

  preSelectedNodeKeys.value = JSON.parse(JSON.stringify(selectedNodeKeys.value))

  document.removeEventListener("mousemove", handleDrag);
  document.removeEventListener("touchmove", handleTouchMove);
}



onMounted(() => {
  containerRef.value = document.getElementById('x-drag-selector')
  containerRef.value.addEventListener("mousedown", handleStartDrag);
  containerRef.value.addEventListener("touchstart", handleTouchStart);


  document.addEventListener("mouseup", handleEndDrag);
  document.addEventListener("touchend", handleEndDrag);
})
</script>

<template>
  <div class="container-wrap" id="drag-selector-wrap" ref="containerWrapRef">
    <div class="selector" id="x-drag-selector">
        <slot>

        </slot>
      
    </div>
    <div ref="selectingBoxRef"
      class="selecting-box"
      :style="{
        backgroundColor: boxColor
      }"
    ></div>
  </div>
</template>

<style scoped>
.container-wrap {
  position: relative;
}
.selecting-box {
  position: absolute;
  /* left: 0;
  right: 0;
  top: 0;
  bottom: 0; */
}
.selector {
  user-select: none;
  position: relative;
}
/* .selector div, .selector span, .selector p {
  user-select: none;
} */
.selector .selected-item {
  background-color: aqua;
}
</style>