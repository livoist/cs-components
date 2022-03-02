<template lang="pug">
.scratchCardEl
  canvas(
    ref="canvas"
    @mousedown="mouseDown"
    @mousemove="mouseMove"
    @touchstart="touchDown"
    @touchmove="touchMove"
  )
</template>

<script lang="ts">
import { defineComponent, ref, reactive, onMounted, nextTick, onUnmounted } from 'vue'
import debounce from 'lodash/debounce'

export default defineComponent({
  name: 'ScratchCard',
  props: {
    isClearScratch: {
      type: Boolean,
      default: false
    },
    brushOptions: {
      type: Object,
      default: () => ({
        size: 20,
        shape: 'round'
      })
    },
    hideOptions: {
      type: Object,
      default: () => ({
        type: 'color',
        value: '#dadada'
      })
    },
    getPercentageCleared: {
      type: Boolean,
      default: false
    },
    percentageStride: {
      type: Number,
      default: 150
    }
  },
  setup(props, context) {
    const {
      brushOptions,
      hideOptions,
      getPercentageCleared,
      percentageStride } = props

    const canvas: any = ref(null)
    const canvasContext: any = ref(null)
    const isPressed = ref(false)
    const observer: any = ref(null)
    const initFlag = ref(false)

    const offset = reactive({
      top: 0,
      left: 0
    })

    const position = reactive({
      currX: 0,
      currY: 0,
      lastX: 0,
      lastY: 0
    })

    const init = () => {
      initFlag.value = true
      setCanvasSizeAndContext()
      nextTick(() => fillArea())
    }

    const setCanvasSizeAndContext = () => {
      nextTick(() => {
        const canvasRect = canvas.value?.getBoundingClientRect()
        const { width, height } = canvasRect
        canvas.value.width = Math.ceil(width)
        canvas.value.height = Math.ceil(height)
        canvasContext.value = canvas.value.getContext('2d')
      })
    }

    const setOffsets = () => {
      const { top, left } = canvas.value?.getBoundingClientRect()
      offset.top = top + document.body.scrollTop
      offset.left = left + document.body.scrollLeft
    }

    
    const setFillStyle = () => {
      const {
        type,
        value = '',
        src = '',
        repeat = ''
      } = hideOptions
      canvasContext.value.globalCompositeOperation = 'source-over'

      if (type === 'color') return new Promise((resolve) => resolve(canvasContext.value.fillStyle = value))

      return new Promise((resolve, reject) => {
        const img = new Image()
        img.onload = () => {
          canvasContext.value.fillStyle = canvasContext.value.createPattern(img, repeat)
          resolve(img)
        }
        img.onerror = (err) => reject(err)
        img.src = src
      })
    }

    const fillArea = async () => {
      const { width, height } = canvas.value?.getBoundingClientRect()
      await setFillStyle()
        .catch((err) => {
          console.log(`
            failed
            Error: ${err.name}
            Message: ${err.message}
          `)
        })
      canvasContext.value.fillRect(0, 0, width, height)
      initFlag.value = false
    }

    const lastPositionHelper = (x: number, y: number) => {
      position.lastX = x
      position.lastY = y
    }

    const currentPositionHelper = (x: number, y: number) => {
      position.currX = x - offset.left
      position.currY = y - offset.top
    }

    const calcClearedArea = () => {
      if (!getPercentageCleared) return
      const { width, height } = canvasContext.value.canvas
      const { data, data: { length } } = canvasContext.value.getImageData(0, 0, width, height)
      const total = length / percentageStride
      let clearedCount = 0
      for (let i = clearedCount; i < length; i += percentageStride) {
        if (parseInt(data[i], 10) === 0) clearedCount += 1
      }
      context.emit('percentage-update', Math.round((clearedCount / total) * 100))
    }

    const draw = () => {
      const {
        currX,
        currY,
        lastX,
        lastY
      } = position

      canvasContext.value.beginPath()
      canvasContext.value.globalCompositeOperation = 'destination-out'
      canvasContext.value.lineWidth = brushOptions.size
      canvasContext.value.lineJoin = brushOptions.shape
      canvasContext.value.moveTo(lastX, lastY)

      if (isPressed.value) {
        canvasContext.value.lineTo(currX, currY)
      } else {
        canvasContext.value.lineTo(currX + 0.01, currY + 0.01)
      }

      canvasContext.value.closePath()
      canvasContext.value.stroke()

      lastPositionHelper(currX, currY)
      calcClearedArea()
    }

    const mouseMove = (event: any) => {
      const { clientX, clientY } = event
      if (!isPressed.value) return
      currentPositionHelper(clientX, clientY)
      draw()
    }

    const mouseDown = (event: any) => {
      const { clientX, clientY } = event
      setOffsets()
      currentPositionHelper(clientX, clientY)
      lastPositionHelper(position.currX, position.currY)
      draw()
      isPressed.value = true
    }

    const mouseUp = () => isPressed.value = false

    const touchDown = (event: any) => {
      const { targetTouches: [{ clientX, clientY }] } = event
      setOffsets()
      currentPositionHelper(clientX, clientY)
      lastPositionHelper(position.currX, position.currY)
      draw()
      isPressed.value = true
    }

    const touchMove = (event: any) => {
      event.preventDefault()
      const { targetTouches: [{ clientX, clientY }] } = event
      currentPositionHelper(clientX, clientY)
      draw()
    }

    const touchUp = () => isPressed.value = false

    onMounted(() => {
      const debounceInit = debounce(() => init(), 200)
      observer.value = new MutationObserver((mutations) => {
        mutations.forEach(({ attributeName }) => {
          if (initFlag.value || !attributeName) return
          debounceInit()
        })
      })
      observer.value.observe(canvas.value, {
        childList: true,
        attributes: true,
        characterData: true,
        subtree: true
      })

      window.addEventListener('mouseup', mouseUp, false)
      window.addEventListener('touchend', touchUp, false)
      window.addEventListener('touchcancel', touchUp, false)

      init()
      window.addEventListener('resize', debounce(init, 500))
    })

    onUnmounted(() => {
      window.removeEventListener('mouseup', mouseUp, false)
      window.removeEventListener('touchend', touchUp, false)
      window.removeEventListener('touchcancel', touchUp, false)
      window.removeEventListener('resize', debounce(init, 500))
      observer.disconnect()
    })

    return {
      canvas,
      touchDown,
      touchMove,
      mouseMove,
      mouseDown
    }
  },
})
</script>

<style lang="sass" scoped>
.scratchCardEl
  position: absolute
  width: 100%
  height: 100%
  border-radius: 8px
  overflow: hidden
  > *
      user-select: none
  &.removeScratch
    display: none

canvas
  width: 100%
  height: 100%
  position: relative
  top: 50%
  left: 50%
  transform: translate(-50%,-50%)

</style>