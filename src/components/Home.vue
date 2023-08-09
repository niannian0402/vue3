<template>
    <div class="homeBox">
        <el-container>
            <el-header>
                <div class="left">
                    <img @click="goHome" src="../assets/img/logo.png" alt="">
                    <span>郑州大学附属郑州市中心医院医务管理系统</span>
                </div>
                <div class="right">

                    <el-popconfirm title="确定退出登录吗?" @confirm="loginOut">
                        <template #reference>
                            <el-icon>
                                <SwitchButton />
                            </el-icon>
                        </template>
                    </el-popconfirm>

                </div>
            </el-header>
            <el-container>
                <el-aside :width="isCollapse ? '64px' : '200px'">
                    <el-menu active-color="#409eff" background-color="#056f89" default-active="0" text-color="#fff"
                        unique-opened :collapse="isCollapse" :collapse-transition="false" router>
                        <div class="ttop" @click="collapse">|||</div>
                        <el-sub-menu :index="item.id + ''" v-for="(item, index) in menuList" :key="item.id">
                            <template #title>
                                <el-icon>
                                    <component :is="icons[index]"></component>
                                </el-icon>
                                <span>{{ item.authName }}</span>
                            </template>
                            <el-menu-item :index="childItem.id + ''" :route="`/home/${childItem.path}`"
                                v-for="childItem in item.children" :key="childItem.id"
                                @click="savePush(item.authName, childItem.authName)">
                                <el-icon>
                                    <component is="Menu"></component>
                                </el-icon>
                                <span>{{ childItem.authName }}</span>
                            </el-menu-item>
                        </el-sub-menu>

                    </el-menu>
                </el-aside>
                <el-main>

                    <router-view></router-view>
                </el-main>
            </el-container>
        </el-container>
    </div>
</template>

<script lang="ts" setup>

import { ref, reactive, getCurrentInstance, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { ArrowRight } from '@element-plus/icons-vue'
let { proxy }: any = getCurrentInstance()
const router = useRouter()
let loginOut = () => {
    sessionStorage.removeItem('token')
    router.push('/login')
    ElMessage({
        message: '退出登录成功',
        type: 'success',
        duration: 500,
        showClose: true,
    })
}
let menuList = ref([])
//侧边栏图标数组
const icons = reactive([
    'User',
    'SwitchFilled',
    'Document',
    'MessageBox',
    'Location',
    'Shop',
    'Orange',
    'Suitcase',
    'VideoPlay'
])
// 请求左侧菜单数据
let getMenuList = async () => {
    let { data: res } = await proxy.$http.get('menu/list')
    if (res.code !== 200) {
    } else {
        // console.log(res.data);
        menuList.value = res.data
    }

}
// 菜单的折叠
let isCollapse = ref(false)
let collapse = () => {
    isCollapse.value = !isCollapse.value
}

// 点击logo去home
let goHome = () => {
    router.push('/home')
}

let savePush = (name1: any, name2: any) => {
    // console.log(name1, name2);
    sessionStorage.setItem("name1", name1);
    sessionStorage.setItem("name2", name2);
}
onMounted(() => {
    getMenuList()

})
</script>

<style lang="less" scoped>
.homeBox {
    width: 100%;
    height: 100%;

    .el-container {
        .el-header {
            width: 100%;
            height: 60px;
            background-color: #13b9c9;
            display: flex;
            justify-content: space-between;
            align-items: center;

            .left {
                display: flex;
                align-items: center;
                color: #fff;

                img {
                    width: 50px;
                    height: 50px;
                    margin-right: 30px;

                    &:hover {
                        cursor: pointer;
                    }
                }

                span {
                    letter-spacing: 5px;
                    font-size: 20px;
                }
            }

            .right {
                font-size: 35px;
                color: #fff;
                margin-right: 50px;

                &:hover {
                    cursor: pointer;
                }
            }
        }
    }
}

.el-container {
    height: 100%;

    .el-aside {
        height: 100%;

        .ttop {
            height: 35px;
            color: #fff;
            letter-spacing: 3px;
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #056f89;

            &:hover {
                background-color: #0b6177;
                cursor: pointer;
            }
        }

        .el-menu {
            height: 100%;
            border: none;
        }
    }
}

.el-main {
    background: url(../assets/img/mainBj.jpg) no-repeat !important;
    background-size: cover !important;
}
</style>