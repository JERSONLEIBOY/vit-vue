<template>
  <div
    :class="{ wrapper: true, x: props.scrollX }"
    ref="wrapper"
  >
    <slot></slot>
  </div>
</template>

<script setup>
import { onMounted, ref, watch } from 'vue'

import BScroll from '@better-scroll/core'
import ObserveDOM from '@better-scroll/observe-dom'
import ObserveImage from '@better-scroll/observe-image'
import Pullup from '@better-scroll/pull-up'
import Pulldown from '@better-scroll/pull-down'
let bscroll = BScroll
const wrapper = ref(null)
BScroll.use(ObserveDOM)
BScroll.use(ObserveImage)
BScroll.use(Pullup)
BScroll.use(Pulldown)
const emit = defineEmits(['ckick', 'beforeScroll', 'afterScroll', 'scroll'])
const props = defineProps({
  /**
   * 1 滚动的时候会派发scroll事件，会截流。
   * 2 滚动的时候实时派发scroll事件，不会截流。
   * 3 除了实时派发scroll事件，在swipe的情况下仍然能实时派发scroll事件
   */
  probeType: {
    type: Number,
    required: false,
    default: 1
  },

  /**
   * 是否开启横向滚动，默认纵向滚动。
   * 开启横向滚动需要将传入元素设置为横向例如：display:inline-block
   */
  scrollX: {
    type: Boolean,
    required: false,
    default: false
  },

  /**
   * 点击列表是否派发click事件
   */
  click: {
    type: Boolean,
    default: true
  },
  /**
   * 是否派发滚动事件
   */
  listenScroll: {
    type: Boolean,
    default: false
  },
  /**
   * 是否派发滚动到底部的事件，用于上拉加载
   */
  pullup: {
    type: Function,
    default: null
  },
  /**
   * 是否派发顶部下拉的事件，用于下拉刷新
   */
  pulldown: {
    type: Function,
    default: null
  },
  /**
   * 是否派发列表滚动开始的事件
   */
  beforeScroll: {
    type: Boolean,
    default: false
  },
  /**
   * 是否派发列表滚动开始的事件
   */
  afterScroll: {
    type: Boolean,
    default: false
  },
  refreshScroll: {
    type: Boolean,
    default: false
  },
  data: {
    type: Array,
    default: []
  }
})
watch(() => props.data, () => {
  setTimeout(() => {
    refresh();
  }, 20)
})
const refresh = () => {
  bscroll && bscroll.refresh();
}
const scrollTo = (x, y, time) => {
  bscroll && bscroll.scrollTo(x, y, time)
}
const scrollToElement = (el, time) => {
  bscroll && bscroll.scrollToElement(el, time)
}
defineExpose({
  refresh,
  scrollTo,
  scrollToElement
})
onMounted(() => {
  bscroll = new BScroll(wrapper.value, {
    scrollX: props.scrollX,
    scrollY: true,
    probeType: props.probeType,
    click: props.click,
    observeDOM: true,
    observeImage: true,
    mouseWheel: true,
    pullDownRefresh: props.pulldown,
    pullUpLoad: props.pullup
  })
  /**
   * 如果开启了滚动前事件派发
   */
  if (props.beforeScroll) {
    bscroll.on('beforeScrollStart', () => {
      emit('beforeScroll')
    })
  }

  /**
   * 如果开启了滚动(滚动中)事件派发
   */
  if (props.listenScroll) {
    bscroll.on('scroll', (position) => {
      emit('scroll', position)
    })
  }

  /**
   * 如果开启了滚动结束事件派发
   */
  if (props.beforeScroll) {
    bscroll.on('scrollEnd', () => {
      emit('afterScroll')
    })
  }
  if (props.refreshScroll) {
    bscroll.refresh();
  }
  if (props.pullup !== null) {
    bscroll.on('pullingUp', () => {
      try {
        props.pullup().then(() => {
          bscroll.finishPullUp()
        })
      } catch (e) {
        // 传入非 Promise 函数
        bscroll.finishPullUp()
      }
    })
  }

  if (props.pulldown !== null) {
    bscroll.on('pullingDown', () => {
      try {
        props.pulldown().then(() => {
          bscroll.finishPullDown()
        })
      } catch (e) {
        // 传入非 Promise 函数
        bscroll.finishPullDown()
      }
    })
  }
})

</script>

<style scoped lang="scss">
.wrapper {
  overflow: hidden;
  .content {
    display: block;
  }
  // 开启横向滚动
  &.x {
    white-space: nowrap;
    .content {
      display: inline-block;
    }
  }
}
</style>