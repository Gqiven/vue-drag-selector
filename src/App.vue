<script setup lang="ts">
import { Ref, ref } from 'vue';
import DragSelector from './components/drag-selector'

type NodeData = {
  value: any,
  index: number
}

const nodeValues = ref<NodeData[]>([])

const createNumberArr = (max: number) => {
  const arr = []
  let count = 0
  while(count < max) {
    arr.push({
      value: count,
      index: count
    })
    count++
  }
  return arr
}

nodeValues.value = createNumberArr(20)

</script>

<template>
  <div class="container">
    <drag-selector>
      <div v-for="({ value, index}) in nodeValues" :key="value" :data-key="index" class="item">
        {{ value }}
      </div>
    </drag-selector>
  </div>
</template>

<style scoped>
.container {
  width: 50%;
  margin: auto;
}
.item {
  width: 50px;
  height: 30px;
  text-align: center;
  line-height: 30px;
  border: 1px solid #ededed;
  border-radius: 4px;
  display: inline-block;
  margin: 5px 5px;
}
</style>
