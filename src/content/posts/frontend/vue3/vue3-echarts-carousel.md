---
title: 基于Vue3的ECharts自动轮播
published: 2025-11-17
image: https://images.unsplash.com/photo-1763037152119-f86924827076?fm=jpg&q=60&w=3000&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
tags: [ "Vue3", "ECharts", "可视化" ]
category: 前端
---

本文章实现的是ECharts数据自动轮播，其中`options`为`computed`计算项，请自行配置。

本文章使用了`useECharts` hook，请参考[基于Vue3的ECharts hook封装](/posts/frontend/vue3/vue3-echarts-hook/)。

```vue
<script setup lang="ts">
import type { ChartCountData } from '@/api/home'
import type { EChartsOptions } from '@/plugins/echarts'
import { HomeApi } from '@/api/home'

const chartRef = useTemplateRef('chartRef')

// 从接口获取的数据列表缓存
const originDataList = ref<ChartCountData[]>([])
// echarts显示的数据列表
const dataList = ref<ChartCountData[]>([])

// echarts配置，包含dataList
const options = computed<EChartsOptions>(() => ({
  ...options
}))

// useECharts hook
const { setOption } = useECharts(chartRef, options.value, {
  defaultInit: false,
})
// loading状态
const loading = ref(false)
// 轮播定时器
const timeout = ref<NodeJS.Timeout | null>(null)
// 轮播步数，用于计算echarts显示的数据列表
const step = ref(0)
// 图表显示数据数量
const chartCount = ref(6)

// 获取数据
async function getData() {
  try {
    loading.value = true
    // 从接口获取数据
    const { data } = await HomeApi.policeCount({
      codeValue: userStore.addressCode,
      days: durationType.value,
    })
    // 缓存
    originDataList.value = data
    // 重置步数
    step.value = 0
    await nextTick()
    // 重置定时器
    if (timeout.value) {
      clearInterval(timeout.value)
    }
    // 数据量小于等于图表显示数量时，直接赋值
    if (originDataList.value.length <= chartCount.value) {
      dataList.value = data
      setOption(options.value)
    } else {
      // 数据量大于图表显示数量时，按图表显示数量滑动
      dataList.value = originDataList.value.slice(step.value, chartCount.value)
      setOption(options.value)
      timeout.value = setInterval(() => {
        step.value++
        // 小于原始数据长度时直接切割
        if (step.value + chartCount.value <= originDataList.value?.length) {
          dataList.value = originDataList.value.slice(
            step.value,
            step.value + chartCount.value,
          )
        } else {
          // 否则补全前面的数据
          dataList.value = [
            ...originDataList.value.slice(step.value),
            ...originDataList.value.slice(
              0,
              chartCount.value - (originDataList.value.length - step.value),
            ),
          ]
          if (step.value === originDataList.value.length) {
            step.value = 0
          }
        }
        setOption(options.value)
      }, 3000)
    }
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  getData()
})

onBeforeUnmount(() => {
  if (timeout.value) {
    clearInterval(timeout.value)
  }
})
</script>

<template>
  <div class="relative h-238px w-full">
    <div
      ref="chartRef"
      :class="{
        'op-0': !dataList?.length,
      }"
      class="size-full"
    ></div>
    <span
      v-if="!dataList?.length"
      class="absolute left-1/2 top-1/2 translate--1/2 text-14px"
    >
      暂无数据
    </span>
  </div>
</template>
```
