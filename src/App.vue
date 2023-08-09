<template>
  <div class="box">
    <router-view></router-view>
  </div>
</template>

<script lang="ts" setup>
import { ref, reactive } from 'vue'

</script>

<style lang="less">
.box {
  width: 100%;
  height: 100%;
}
</style>