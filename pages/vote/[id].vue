<script setup lang="ts">
import { ref } from 'vue';

const route = useRoute()
const name = route.params.id

const generateTimeRange = (start: number, end: number) => {
  const timeRange = []
  for (let i = start; i < end; i++) {
    timeRange.push(`${i}:00`)
  }
  return timeRange
}

const data = {
  id: 1,
  name: '计协管理层第一次会议',
  dateRange: ['8/27', '8/28'],
  timeRange: generateTimeRange(9, 22)
}

const isMouseDown: Ref<boolean> = ref(false)

type position = {
  date: string,
  time: string
}

const lastPostion: Ref<position> = ref({ date: '', time: '' })
const selectedTime: Ref<position[]> = ref([])

const log: Ref<string[]> = ref([])

const move = (date: string, time: string) => {
  log.value.push('move', date, time)
  if (isMouseDown.value === true) {
    log.value.push('move and down', date, time)
    if (lastPostion.value.date === date && lastPostion.value.time === time) {
      log.value.push('move and down returned', date, time)
      return
    }
    lastPostion.value = { date, time }
    if (selectedTime.value.find((item) => item.date === date && item.time === time)) {
      selectedTime.value = selectedTime.value.filter((item) => item.date !== date || item.time !== time)
    } else {
      selectedTime.value.push({ date, time })
    }
  }
}

</script>
  
<template>
  <div class="page-index">
    <h2>{{ data.name }}</h2>
    <span class="text-gray-400">投票ID - {{ name }}</span>
  </div>
  <div class="flex flex-col my-5 rounded p5 w-[800px] min-w-[800px] mxa bg-gray-50 select-none"
    @touchstart="() => isMouseDown = true" @touchend="() => isMouseDown = false" @touchcancel="() => isMouseDown = false"
    @mousedown="() => isMouseDown = true" @mouseup="() => isMouseDown = false" @mouseleave="() => isMouseDown = false">
    <div class="flex flex-row">
      <h4 class="mx-auto text-blue-500 my-2">请 拖拽 或 点选 你空闲的时间</h4>
    </div>
    <div>
      <table
        class="select-time my-5 w-full table-fixed text-center outline outline-1 outline-gray border-spacing-0 bg-white">
        <tr v-for="date in ['TIME', ...data.dateRange]">
          <td class="border-gray-200 bg-white py2 text-gray-5 font-bold border-solid border-1">{{ date }}</td>
          <td class="border-gray-200 bg-white py2 text-gray-5 font-bold border-solid border-1" v-if="date === 'TIME'"
            v-for="time in data.timeRange">{{ time }}</td>
          <td v-else v-for="time in data.timeRange" @touchmove="(e) => move(date, time)"
            @mousemove="(e) => move(date, time)" @click="(e) => move(date, time)"
            :class="['border-gray-200 border-solid border-1', selectedTime.find((item) => item.date === date && item.time === time) ? 'bg-green-100' : 'bg-white']">
          </td>
        </tr>
      </table>
    </div>
    <div class="flex flex-row">
      <el-button class="mr-auto" type="default" @click="() => (selectedTime = []) && (log = [])">重置</el-button>
      <el-button class="ml-auto" type="primary" @click="() => router.push('/vote/1')">提交</el-button>
    </div>
    {{ log.reverse().map((item) => item) }}
  </div>
</template>

<style>
.page-index {
  padding-top: 60px;
  text-align: center;
}

.select-time {
  font-family: 'Courier New', Courier, monospace;
}

html {
  touch-action: none;
}
</style>