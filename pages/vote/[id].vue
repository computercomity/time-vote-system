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

const tableBox: Ref<HTMLElement | null> = ref(null)

const move = (date: string, time: string, click = false) => {
  const _move = (date: string, time: string) => {
    if (selectedTime.value.find((item) => item.date === date && item.time === time)) {
      selectedTime.value = selectedTime.value.filter((item) => item.date !== date || item.time !== time)
    } else {
      selectedTime.value.push({ date, time })
    }
  }

  // 判断是否是手机，如果是手机，调用点击事件
  if (isMouseDown.value === true) {
    if (lastPostion.value.date === date && lastPostion.value.time === time) {
      return
    }
    lastPostion.value = { date, time }
    _move(date, time)
    return
  }
}

const resetLocation = () => {
  lastPostion.value = { date: '', time: '' }
  return true
}
const mouseDown = () => {
  isMouseDown.value = true
}
const mouseUp = () => {
  isMouseDown.value = false
}
const mouseMove = (e: MouseEvent, date: string, time: string) => {
  // 手机端不触发
  if (/Android|webOS|iPhone|iPod|BlackBerry/i.test(e.view?.window.navigator.userAgent ?? '')) {
    return
  }
  move(date, time)
}
const touchMove = (e: TouchEvent) => {
  const myLocation = e.changedTouches[0];
  const realTarget = e.view?.window.document.elementFromPoint(myLocation.clientX, myLocation.clientY);
  if (realTarget?.tagName === 'TD') {
    const target = (realTarget as HTMLElement)
    const date = target.dataset.date
    const time = target.dataset.time
    date && time && move(date, time)
  }

  autoScroll(myLocation.clientX)
}
const autoScroll = (clientX: number) => {
  // 当拖动到右侧边缘时，自动滚动
  if (!tableBox?.value) return

  const clientWidth = tableBox.value.clientWidth ?? 0 // 不变
  const scrollLeft = tableBox.value.scrollLeft ?? 0

  const right = clientWidth - clientX

  if (right < 50 && right > 0) {
    tableBox.value.scrollLeft = scrollLeft + (50 - right) * 0.4
    return
  }
  const left = clientX
  if (left < 50 && left > 0) {
    tableBox.value.scrollLeft = scrollLeft - (50 - left) * 0.4
    return
  }
}

const reset = () => {
  selectedTime.value = []
  ElNotification.success({
    title: '成功',
    message: '已重置时间选择',
  })
}

const submit = () => {
  if (selectedTime.value.length === 0) {
    ElNotification.error({
      title: '错误',
      message: '请至少选择一个时间',
    })
    return
  }
  console.log(selectedTime.value)
}
</script>
  
<template>
  <div class="text-center py-12">
    <h2 class="text-gray-800">{{ data.name }}</h2>
    <span class="text-gray-400">投票ID - {{ name }}</span>
  </div>
  <!-- https://stackoverflow.com/questions/3918842/how-to-find-out-the-actual-event-target-of-touchmove-javascript-event -->
  <!-- 单击(当前元素按下，当前元素弹起，和按下移动冲突) -> 选中/取消
  移动 -> 无效果
  按下移动 -> 如果当前位置有，取消，如果当前位置没有，选中
  电脑端  mousedown -> (mousemove) -> mouseup -> click
  手机端  mousemove -> mousedown -> mouseup -> click
  -->
  <div class="my-4 p-5 bg-[#f5f5f7] rounded-lg mxa max-w-[1000px]">
    <div class="flex inline-block flex-col select-none" @mouseleave="mouseUp" @touchend="mouseUp" @touchcancel="mouseUp"
      @touchstart="() => resetLocation() && mouseDown()">
      <div class="flex flex-col my-2">
        <h3 class="mx-auto text-[#017bff] my-2">请 拖拽 或 点选 你空闲的时间</h3>
        <h5 class="mx-auto text-[#017bff] my-1">Click or drag the time you are free.</h5>
      </div>
      <div style="overflow-x: scroll;" class="my-5 p-2 table-box" ref="tableBox">
        <table @touchmove.prevent="touchMove"
          class="select-time w-full min-w-[800px] table-fixed text-center outline outline-0.5 outline-[#6e6e73] border-spacing-0 bg-white shadow">
          <tr v-for="date in ['TIME', ...data.dateRange]">
            <td class="bg-white py2 text-gray-800 font-bold border-solid border-0.1 border-[#6e6e73]">{{ date }}</td>
            <td class="bg-white py2 text-gray-800 font-bold border-solid border-0.1 border-[#6e6e73]"
              v-if="date === 'TIME'" v-for="time in data.timeRange">{{ time }}</td>
            <td v-else v-for="time in data.timeRange" @mousedown="() => {
              resetLocation()
              mouseDown()
              move(date, time)
            }" @mouseup="mouseUp" @mousemove="(e) => mouseMove(e, date, time)" :data-date="date" :data-time="time"
              :class="['border-solid border-0.1 border-[#6e6e73] cursor-pointer', selectedTime.find((item) => item.date === date && item.time === time) ? 'bg-[#68cc45]' : 'bg-white']">
            </td>
          </tr>
        </table>
        <div
          class="inline-block flex flex-col md:hidden py-4 w-full min-w-[800px] outline-dashed outline-0.1 outline-[#6e6e73]">
          <span class="ml-4 color-[#6e6e73] font-300 font-italic animate-fade-in-right">*拖动我查看右侧更多选择 >>></span>
          <span class="ml-6 color-[#6e6e73] font-300 font-italic animate-fade-in-right">Drag here to see more >>></span>
        </div>

      </div>
      <div class="flex flex-row">
        <el-button class="mr-auto" type="default" @click="reset">重置</el-button>
        <el-button class="ml-auto" type="primary" @click="submit">提交</el-button>
      </div>
    </div>
  </div>
</template>

<style>
.select-time {
  font-family: 'Courier New', Courier, monospace;
}

html {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}
</style>