<template>
  <div class="my-scroll" ref="container" :style="{height: containerHeight}">
    <div class="content" ref="content">
      <slot/>
    </div>
    <loading class="loading" :height="pullTopDistance" ref="loading"/>
  </div>
</template>

<script>
import Loading from "@/components/loading";
const debounce = (fn, time) => {
  let timer = null
  return (...args) => {
    if (timer) {
      clearTimeout(timer)
    }
    timer = setTimeout(() => {
      fn.apply(this, args)
    }, time)
  }
}


export default {
  name: "my-scroll",
  components: {Loading},
  events: ["pullTopEvent", "pullDownEvent"],
  props: {
    containerHeight: {
      type: String,
      required: true
    },
    pullTopDistance: {
      type: Number,
      default: 80
    },
    pullDownDistance: {
      type: Number,
      default: 80
    }

  },
  data() {
    return {
      height: 100,
    }
  },
  mounted() {
    this.initEvent()
  },
  beforeDestroy() {
    this.$refs.container.removeEventListener("touchmove", this.pullTopEventListener)
    this.$refs.container.removeEventListener("touchmove", this.pullDownEventListener)
  },
  watch: {

  },
  methods: {
    initEvent() {
      const container = this.$refs.container
      const content = this.$refs.content
      const loading = this.$refs.loading
      let startY, calculating, dy, lastY, fetching

      this.resetPullTopStatus = () => {
        fetching = false
        calculating = false
        content.style.top = "0"
        loading.stop()
      }
      this.pullTopEventListener = ev => {
        if (fetching) {
          return
        }
        window.addEventListener("touchend", this.windowEndEvent)
        const e = ev.touches[0]
        const scrollY = container.scrollTop

        // 达到了上拉边界, startY开始工作统计上拉距离
        if (scrollY === 0 && !calculating) {
          calculating = true
          startY = e.clientY

          // 不在上拉边界, 如果之前是上拉状态重置为否
        } else if (scrollY !== 0) {
          calculating && (calculating = false)
        }

        // 如果在上拉状态而且不在fetching状态, 随时判断是否达到fetching标准
        if (calculating && !fetching) {
          const isAdd = e.clientY - lastY > 0
          const deg = isAdd ? 10 : -10
          loading.rotate(deg)

          // dy是拉动的距离
          dy = e.clientY - startY
          content.style.top = dy + "px"
          if (dy >= this.pullTopDistance) { // 达到了刷新的条件
            calculating = false
            fetching = true
            loading.start()
            this.pullTopEvent()
          }
        }
        lastY = e.clientY
      }
      this.windowEndEvent = ev => {
        if (calculating && !fetching) {
          // 未达到刷新条件, 执行清理工作
          this.resetPullTopStatus()
        }
        window.removeEventListener("touchend", this.windowEndEvent)
      }


      this.resetPullDownStatus = () => {
        fetching = false
      }
      this.pullDownEventListener = debounce(ev => {
        if (fetching) {
          return
        }
        const e = ev.touches[0]
        const scrollY = container.scrollTop
        const ret = container.offsetHeight + scrollY + this.pullDownDistance >= content.scrollHeight
        if (ret) {
          fetching = true
          this.pullDownEvent()
        }
      }, 10)
      this.cancelPullDownEvent = () => {
        container.removeEventListener("touchmove", this.pullDownEventListener)
      }

      if (this.$listeners.pullTopEvent){
        container.addEventListener("touchmove", this.pullTopEventListener)
      }
      if (this.$listeners.pullDownEvent) {
        container.addEventListener("touchmove", this.pullDownEventListener)
      }
    },
    pullTopEvent() {
      const callback = () => {
        this.resetPullTopStatus()
      }
      this.$emit("pullTopEvent", callback)
    },
    pullDownEvent() {
      const callback = (isFinish) => {
        this.resetPullDownStatus()
        isFinish && this.cancelPullDownEvent()
      }
      this.$emit("pullDownEvent", callback)
    },
  }
}
</script>

<style scoped>
.my-scroll {
  position: relative;
  overflow: auto;
}

.content {
  background-color: rgba(200, 200, 200, 1);

  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
}

.loading {
  position: absolute;
  z-index: -1;
}
</style>
