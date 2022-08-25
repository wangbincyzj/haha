<template>
  <canvas ref="canvas"></canvas>
</template>

<script>
export default {
  name: "loading",
  props: {
    height: {
      type: Number,
      default: 100
    }
  },
  data() {
    return {
      width: "100%"
    }
  },
  mounted() {
    this.initInstance()
    this.initSize()
    this.initStatus()
  },
  methods: {
    initInstance() {
      this.canvas = this.$refs.canvas
    },
    initSize() {
      this.canvas.width = window.innerWidth
      this.canvas.height = this.height
    },
    initStatus() {
      const canvas = this.canvas
      const {width, height} = this.canvas
      const ctx = canvas.getContext("2d")
      const gap = 1 / 360 * 2 * Math.PI
      ctx.lineWidth = 2
      let v = 0
      let running = false
      const render = () => {
        for (let i = 0; i < 360; i++) {
          const deg = (i + v) * gap
          ctx.save()
          ctx.translate(width / 2, height / 2)
          ctx.strokeStyle = `rgb(${i * .5},${256 - i * .6}, 200)`
          ctx.beginPath()
          ctx.rotate(deg)
          ctx.arc(0, 0, 10, 0, gap)
          ctx.stroke()
          ctx.closePath()
          ctx.restore()
        }
      }
      const clear = () => {
        ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
      }
      const start = () => {
        running = true
        v += 10
        requestAnimationFrame(() => {
          clear()
          render()
          if (running) {
            start()
          }
        })
      }
      const stop = () => running = false

      // 提供外部调用
      this.clear = clear
      this.stop = stop
      this.start = start
      this.rotate = (deg) => {
        v += deg
        render()
      }
    },
  }
}
</script>

<style scoped>
</style>
