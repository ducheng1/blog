---
title: 基于Vue3的ECharts hook封装
published: 2025-11-17
image: https://images.unsplash.com/photo-1761839257658-23502c67f6d5?fm=jpg&q=60&w=3000&ixlib=rb-4.1.0&ixid=M3wxMjA3fDF8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
tags: [ "Vue3", "ECharts", "Hooks", "可视化" ]
category: 前端
---

## 引入ECharts

推荐按需引入，如果不考虑包体积则可以全量引入

### 按需引入

```typescript
// 系列类型
import type {
  BarSeriesOption,
  LineSeriesOption,
  PieSeriesOption,
} from 'echarts/charts'
// 组件类型
import type {
  DatasetComponentOption,
  DataZoomComponentOption,
  GridComponentOption,
  LegendComponentOption,
  TitleComponentOption,
  TooltipComponentOption,
} from 'echarts/components'
// 类型工具
import type { ComposeOption } from 'echarts/core'
// 图表
import { BarChart, LineChart, PieChart } from 'echarts/charts'
// 组件
import {
  DatasetComponent,
  DataZoomComponent,
  GridComponent,
  LegendComponent,
  TitleComponent,
  TooltipComponent,
  // 内置数据转换器组件 (filter, sort)
  TransformComponent,
} from 'echarts/components'
import * as echarts from 'echarts/core'
// 功能
import { LabelLayout, UniversalTransition } from 'echarts/features'
// 渲染器
import { CanvasRenderer } from 'echarts/renderers'

export type EChartsOptions = ComposeOption<
  | BarSeriesOption
  | LineSeriesOption
  | TitleComponentOption
  | TooltipComponentOption
  | GridComponentOption
  | DatasetComponentOption
  | PieSeriesOption
  | LegendComponentOption
  | DataZoomComponentOption
>

// 注册必须的组件
echarts.use([
  // 组件
  TitleComponent,
  TooltipComponent,
  GridComponent,
  DatasetComponent,
  TransformComponent,
  LegendComponent,
  DataZoomComponent,
  // 图表
  BarChart,
  LineChart,
  PieChart,
  // 功能
  LabelLayout,
  UniversalTransition,
  // 渲染器
  CanvasRenderer,
])

export default echarts
```

## Hook封装

这里实现了一些基本功能，如自动调整尺寸防抖、销毁页面的同时销毁实例等

```typescript
import type { ECharts } from 'echarts/core'
import type { EChartsOptions } from '@/plugins/echarts'
import { debounce } from 'es-toolkit'
import echarts from '@/plugins/echarts'

export interface UseEChartsOptions {
  /**
   * echarts初始化选项
   */
  initOptions?: echarts.EChartsInitOpts
  /**
   * 是否自动调整尺寸
   */
  autoResize?: boolean
  /**
   * 是否开启动画
   */
  animation?: boolean
  /**
   * resize防抖时间
   */
  resizeDebounce?: number
  /**
   * 是否默认初始化options
   */
  defaultInit?: boolean
}

export function useECharts(
  domRef: Ref<HTMLElement | null>,
  chartOptions: EChartsOptions,
  options?: UseEChartsOptions,
) {
  const {
    initOptions,
    autoResize = true,
    animation = true,
    resizeDebounce = 300,
    defaultInit = true,
  } = options ?? {}

  let chartInst: ECharts | null

  // 初始化
  function init() {
    if (!domRef.value) {
      return
    }
    chartInst = echarts.init(domRef.value, undefined, initOptions)
  }

  // 设置选项
  function setOption(
    options: EChartsOptions,
    notMerge?: boolean,
    lazyUpdate?: boolean,
  ) {
    if (!chartInst) {
      return
    }
    nextTick(() => {
      // @ts-expect-error initialized
      chartInst.setOption(options, {
        notMerge,
        lazyUpdate,
        silent: !animation,
      })
    })
  }

  // 自动调整尺寸函数
  const resize = debounce(
    () => {
      if (!chartInst) {
        return
      }
      chartInst.resize({
        animation: animation
          ? {
              duration: 300,
            }
          : undefined,
      })
    },
    resizeDebounce,
    { edges: ['leading', 'trailing'] },
  )

  // 自动调整尺寸
  if (autoResize) {
    window.addEventListener('resize', resize)
  }

  // 销毁实例
  function destroy() {
    if (chartInst) {
      chartInst?.dispose()
      chartInst = null
    }
    window.removeEventListener('resize', resize)
  }

  onMounted(() => {
    nextTick(() => {
      init()
      if (defaultInit) {
        setOption(chartOptions)
      }
    })
  })

  onUnmounted(() => {
    destroy()
  })

  return {
    init,
    // @ts-expect-error initialized
    chartInst,
    setOption,
    destroy,
    resize,
  }
}
```
