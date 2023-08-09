<template>
    <div class="loginBox">

        <!-- 表单验证 -->
        <el-form ref="ruleFormRef" :rules="loginFromRule" :model="loginFrom" class="loginFrom">
            <el-form-item prop="uname">
                <el-input v-model="loginFrom.uname" :prefix-icon="User" />
            </el-form-item>
            <el-form-item prop="upwd">
                <el-input v-model="loginFrom.upwd" type="password" show-password :prefix-icon="Lock" />
            </el-form-item>

            <div class="btn">
                <el-button color="#54b9c9" plain @click="submit">登录</el-button>
                <el-button color="#54b9c9" plain @click="setFrom">重置</el-button>
            </div>

        </el-form>
    </div>
</template>

<script lang="ts" setup>
import { ref, reactive, getCurrentInstance, onMounted } from 'vue'
import type { FormRules } from 'element-plus'
import { ElMessage } from 'element-plus'
import { useRouter } from 'vue-router'
const router = useRouter()

import { User, Lock } from '@element-plus/icons-vue'
let { proxy }: any = getCurrentInstance()
// 表单数据
let loginFrom = reactive({
    uname: 'admin',
    upwd: '123456'
})
// 表单验证
const loginFromRule = reactive<FormRules>({
    uname: [{ required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 6, message: '用户名3-6位', trigger: 'blur' }],
    upwd: [{ required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 12, message: '密码6-12位', trigger: 'blur' },]
})
// 重置按钮
let ruleFormRef = ref()
let setFrom = () => {
    ruleFormRef.value.resetFields()
    sessionStorage.removeItem('token')
    ElMessage({
        message: '登录成功',
        type: 'success',
        duration: 500,
        showClose: true,
    })
}
// const submit = () => {
//     // 获取到真正的表单元素
//     ruleFormRef.value.validate((isValid, invalidFields) => {
//         if (isValid) {
//             // 表单所有元素验证通过，可以提交了
//             console.log('验证通过')
//         } else {
//             console.log(invalidFields)
//             console.log('验证不通过,不能提交,请检查')
//         }
//     })
// }
const submit = async () => {
    // 获取到真正的表单元素
    let result = await ruleFormRef.value.validate((isValid) => {
        if (!isValid) {
            // console.log('验证不通过,不能提交,请检查')
            sessionStorage.removeItem('token')
            ElMessage({
                message: '请输入合法的用户名和密码',
                type: 'error',
                duration: 500,
                showClose: true
            })
        } else {
            // 表单所有元素验证通过，可以提交了
            // console.log('验证通过')
            login()
        }
    })
}
// 登录请求
const login = async () => {
    let { data: res } = await proxy.$http.post('users/login', loginFrom)
    // console.log(res);
    if (res.code !== 200) {
        // console.log('请求失败');

    } else {
        sessionStorage.setItem('token', res.token)
        ElMessage({
            message: '登录成功',
            type: 'success',
            duration: 500,
            showClose: true,
            onClose: () => {
                router.push('/home')
            }
        })
    }

}

onMounted(() => {

})
</script>

<style lang="less" scoped>
.loginBox {
    width: 100%;
    height: 100%;
    background: url(../assets/img/background.png) no-repeat;
    background-size: cover;
    position: relative;

    .loginFrom {
        width: 20%;

        padding: 30px 20px;
        box-sizing: border-box;
        position: absolute;
        right: 0;
        top: 50%;
        margin-right: 170px;
        border: 1px solid #00bccc;
        border-radius: 7px;


        .el-form-item {
            width: 100%;

            .el-input {
                width: 100%;
            }
        }

        .btn {
            width: 100%;
            display: flex;
            justify-content: space-evenly;

        }
    }
}
</style>