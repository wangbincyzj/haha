<template>
  <div id="app">
    <my-scroll  @pullTopEvent="handlePullTop"  @pullDownEvent="handlePullDown" container-height="100vh">
      <test-page/>
    </my-scroll>
  </div>
</template>

<script>
import TestPage from "@/components/test/TestPage";
import MyScroll from "@/components/my-scroll";

export default {
  name: 'App',
  components: {
    MyScroll,
    TestPage,
  },
  methods: {
    handlePullTop(ck) {
      setTimeout(() => {
        ck()
      }, 3000)
    },
    handlePullDown(ck) {
      setTimeout(() => {
        console.log(111111111)
        ck()
      }, 3000)
    },
  }
}
</script>

<style lang="scss">
* {
  padding: 0;
  margin: 0;
}
</style>
