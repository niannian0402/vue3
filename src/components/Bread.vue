<template>
    <div class="">
        <el-breadcrumb :separator-icon="ArrowRight" v-if="$route.path != '/welcome'">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>{{ name1M }}</el-breadcrumb-item>
            <el-breadcrumb-item>{{ name2M }}</el-breadcrumb-item>

        </el-breadcrumb>
    </div>
</template>

<script lang="ts" setup>
import { ArrowRight } from '@element-plus/icons-vue'
import { ref, reactive, onMounted } from 'vue'
const name1M = ref('');
const name2M = ref('');

onMounted(() => {

    name1M.value = sessionStorage.getItem("name1")
    name2M.value = sessionStorage.getItem("name2")
})
</script>

<style lang="less" scoped></style>