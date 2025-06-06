<script setup lang="ts">
import {
  computed,
  getCurrentInstance,
  nextTick,
  onBeforeUnmount,
  onMounted,
  ref,
  toRef,
  watch,
  watchEffect,
} from 'vue'

import { emitEvent, useNameHelper, useProps } from '@vexip-ui/config'
import { useManualRef, useRtl, useSetTimeout } from '@vexip-ui/hooks'
import { USE_TOUCH, boundRange, isDefined, throttle } from '@vexip-ui/utils'
import { scrollbarProps } from './props'
import { useTrack } from './hooks'
import { ScrollbarType, scrollbarPlacements } from './symbol'

defineOptions({ name: 'Scrollbar' })

const _props = defineProps(scrollbarProps)
const props = useProps('scrollbar', _props, {
  placement: {
    default: 'right',
    validator: value => scrollbarPlacements.includes(value),
  },
  scroll: {
    default: 0,
    validator: value => value >= 0 && value <= 100,
    static: true,
  },
  barLength: {
    default: 35,
    validator: value => value > 0 && value < 100,
  },
  width: null,
  appear: false,
  fade: 1500,
  barColor: null,
  trackColor: null,
  disabled: false,
  wrapper: null,
  duration: null,
  useTrack: false,
  trackSpeed: {
    default: 2,
    validator: value => value > 0 && value < 10,
  },
})

const nh = useNameHelper('scrollbar')
const active = ref(false)
const scrolling = ref(false)

const { manualRef, triggerUpdate } = useManualRef()

const { isRtl } = useRtl()

const currentScroll = manualRef(props.scroll)

const container = ref<HTMLElement>()
const bar = ref<HTMLElement>()
const track = ref<HTMLElement>()

const { timer } = useSetTimeout()

const type = computed(() => {
  return props.placement === 'right' || props.placement === 'left'
    ? ScrollbarType.VERTICAL
    : ScrollbarType.HORIZONTAL
})

const { tracking, handleMouseDown: handleTrackMouseDown } = useTrack({
  currentScroll,
  track,
  bar,
  type,
  trackSpeed: toRef(props, 'trackSpeed'),
  barLength: toRef(props, 'barLength'),
  disabled: toRef(props, 'disabled'),
  onDown: scroll => {
    clearTimeout(timer.fade)
    emitEvent(props.onScrollStart, scroll)
  },
  // onMove: () => clearTimeout(timer.fade),
  onUp: scroll => {
    setScrollbarFade()
    triggerUpdate()
    emitEvent(props.onScrollEnd, scroll)
  },
  onScroll: scroll => {
    triggerUpdate()
    emitEvent(props.onScroll, scroll)
  },
})

const className = computed(() => {
  return [
    nh.b(),
    nh.bs('vars'),
    nh.bm(props.placement),
    {
      [nh.bm('inherit')]: props.inherit,
      [nh.bm('fade')]: props.fade,
      [nh.bm('scrolling')]: scrolling.value,
      [nh.bm('tracking')]: tracking.value,
      [nh.bm('active')]: active.value,
      [nh.bm('disabled')]: props.disabled,
    },
  ]
})
const style = computed<Record<string, string>>(() => {
  return {
    [nh.cv('bar-bg-color')]: props.barColor,
    [nh.cv('track-bg-color')]: props.trackColor,
    [nh.cv('width')]: props.width ? `${props.width}px` : null!,
  }
})

watch(
  () => props.scroll,
  value => {
    currentScroll.value = value
    triggerUpdate()
  },
)
watchEffect(() => {
  if (!bar.value) return

  const position = `${((100 - props.barLength) * currentScroll.value) / props.barLength}%`
  const length = `${props.barLength}%`

  if (type.value === ScrollbarType.VERTICAL) {
    bar.value.style.height = length
    bar.value.style.transform = `translate3d(0, ${position}, 0)`
  } else {
    bar.value.style.width = length
    bar.value.style.transform = `translate3d(${isRtl.value ? '-' : ''}${position}, 0, 0)`
  }
})
watchEffect(() => {
  if (!bar.value) return

  bar.value.style.transitionDuration =
    isDefined(props.duration) && props.duration >= 0 ? `${props.duration}ms` : ''
})

if (props.appear) {
  watch(currentScroll, () => {
    clearTimeout(timer.fade)
    active.value = true

    if (!scrolling.value && !tracking.value) {
      setScrollbarFade()
    }
  })
}

const handleWrapperMouseMove = throttle(() => {
  clearTimeout(timer.fade)

  if (props.disabled) {
    active.value = false
  } else {
    active.value = true

    if (!scrolling.value && !tracking.value) {
      setScrollbarFade()
    }
  }
})

let wrapperElement: HTMLElement | null

onMounted(() => {
  let instance = getCurrentInstance()

  nextTick(() => {
    if (typeof props.wrapper === 'string') {
      wrapperElement = document.querySelector(props.wrapper)
    } else {
      wrapperElement = props.wrapper
    }

    if (!wrapperElement) {
      if (instance?.parent) {
        wrapperElement = instance.parent.proxy?.$el

        if (!wrapperElement) {
          wrapperElement = container.value?.parentElement ?? null
        }
      } else {
        wrapperElement = container.value?.parentElement ?? null
      }
    }

    if (wrapperElement && props.fade >= 300) {
      wrapperElement.addEventListener('mousemove', handleWrapperMouseMove)
    }

    instance = null

    if (!props.appear) {
      watch(currentScroll, () => {
        clearInterval(timer.fade)
        active.value = true
        setScrollbarFade()
      })
    }
  })
})

onBeforeUnmount(() => {
  if (wrapperElement) {
    wrapperElement.removeEventListener('mousemove', handleWrapperMouseMove)
  }

  wrapperElement = null
  clearTimeout(timer.fade)
})

defineExpose({
  currentScroll,
  container,
  bar,
  track,
  handleScroll,
})

let length: number
let startAt: number
let cursorAt: number

function handleMouseDown(event: PointerEvent) {
  if (event.button !== 0 || props.disabled) {
    return false
  }

  event.stopPropagation()
  event.preventDefault()

  if (!track.value || !bar.value) return false

  document.addEventListener('pointermove', handleMouseMove)
  document.addEventListener('pointerup', handleMouseUp)

  const rect = track.value.getBoundingClientRect()
  const barRect = bar.value.getBoundingClientRect()

  if (type.value === ScrollbarType.VERTICAL) {
    length = rect.height
    startAt = barRect.top - rect.top
    cursorAt = event.clientY
  } else {
    length = rect.width
    startAt = isRtl.value ? barRect.right - rect.right : barRect.left - rect.left
    cursorAt = event.clientX
  }

  clearTimeout(timer.fade)

  scrolling.value = true
  emitEvent(props.onScrollStart, currentScroll.value)
}

function handleMouseMove(event: PointerEvent) {
  event.stopPropagation()

  if (!USE_TOUCH) {
    event.preventDefault()
  }

  let position: number

  if (type.value === ScrollbarType.VERTICAL) {
    position = startAt + event.clientY - cursorAt
  } else {
    position = isRtl.value
      ? -(startAt + event.clientX - cursorAt)
      : startAt + event.clientX - cursorAt
  }

  // position / length * 100 === (100 - barLength) * currentScroll / 100
  currentScroll.value = (position / length / (100 - props.barLength)) * 1e4

  verifyScroll()
  triggerUpdate()
  emitEvent(props.onScroll, currentScroll.value)
}

function handleMouseUp(event: PointerEvent) {
  event.preventDefault()

  document.removeEventListener('pointermove', handleMouseMove)
  document.removeEventListener('pointerup', handleMouseUp)

  setScrollbarFade()

  scrolling.value = false
  emitEvent(props.onScrollEnd, currentScroll.value)
}

function verifyScroll() {
  currentScroll.value = Math.max(0, Math.min(currentScroll.value, 100))
}

function setScrollbarFade() {
  if (props.fade >= 300) {
    timer.fade = setTimeout(() => {
      active.value = false
    }, props.fade)
  }
}

function handleScroll(scroll: number) {
  if (Math.abs(currentScroll.value - scroll) < 0.0001) return

  currentScroll.value = boundRange(scroll, 0, 100)
  triggerUpdate()
}

function disableEvent<E extends Event>(event: E) {
  if (event.cancelable) {
    event.stopPropagation()
    event.preventDefault()
  }
}
</script>

<template>
  <div
    ref="container"
    :class="className"
    role="scrollbar"
    :style="style"
  >
    <div
      ref="track"
      :class="[nh.be('track'), props.useTrack ? null : nh.bem('track', 'disabled')]"
      @touchstart="disableEvent"
      @pointerdown="handleTrackMouseDown"
    ></div>
    <div
      ref="bar"
      :class="nh.be('bar')"
      @touchstart="disableEvent"
      @pointerdown="handleMouseDown"
    ></div>
  </div>
</template>
