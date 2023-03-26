<template>
  <web-view :src="url">

  </web-view>
</template>

<script>
export default {
  name: "index",
  data() {
    return {
      url: 'https://www.baidu.com',
    }
  }
}
</script>

<style scoped>

</style>
