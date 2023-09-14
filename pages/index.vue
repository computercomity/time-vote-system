<template>
  <div class="text-center pt-15">
    <h1>CUHKSZ 计算机协会管理系统</h1>
    <el-button type="primary" @click="open('/vote')">前往投票</el-button>
    <el-button type="primary" @click="open('/join')">招新加入</el-button>
    <el-button type="primary" @click="open('https://computercomity.com')">计协官网</el-button>
  </div>
</template>


<script lang="ts" setup>
const router = useRouter()

const open = (href: string) => {
  if (href.startsWith('http')) {
    window.open(href)
    return
  }
  router.push(href)
}

</script>