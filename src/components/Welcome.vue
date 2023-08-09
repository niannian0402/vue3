<template>
    <div class="welcomeBox">

        <img src="../assets/img/zhengzhouHospital.2a5b1f5d.svg" alt="">
    </div>
</template>

<script lang="ts" setup>
import { ref, reactive } from 'vue'

</script>

<style lang="less" scoped>
.welcomeBox {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    background: url(../assets/img/mainBj.jpg) no-repeat;
    background-size: cover;

    img {
        width: 1200px;
    }
}
</style>