<template>
  <el-menu :default-active="activeIndex" class="el-menu" mode="horizontal" :ellipsis="false" @select="handleSelect">
    
    <div class="mt-2.4 mb-0.8 ml-2.5 mr-2">
      <ElImage class="w-10 h-10" src="/logo.png">Logo</ElImage>
    </div>
    <el-menu-item v-for="link in menu" :index="link.index" :class="link.index === 'gap' && 'flex-grow'">
      {{ link && link?.title }}
    </el-menu-item>
  </el-menu>
</template>

<script lang="ts" setup>
const menu = [
  {
    index: '0',
    title: '首页',
    src: '/'
  },
  {
    index: '1',
    title: '投票',
    src: '/vote'
  },
  {
    index: 'gap',
  },
  {
    index: '2',
    title: '关于',
    src: '/about'
  }
]

const router = useRouter()

const calCulateActiveIndex = () => {
  const index = menu.findIndex((e) => e.src === router.currentRoute.value.path)
  return index === -1 ? '-1' : index.toString()
}

const activeIndex = ref(calCulateActiveIndex())
const handleSelect = (key: string, keyPath: string[]) => {
  router.push(menu.find((e) => e.index === key)!.src ?? '/')
}
</script>
