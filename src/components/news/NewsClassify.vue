<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button color="#13b9c9" class="white" @click="add_">添加分类</el-button>
                </div>
            </template>
            <!-- 表格区域 -->
            <el-table :data="newsList" stripe style="width: 100%" border :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column prop="id" label="分类ID" width="90" />
                <el-table-column prop="cname" label="分类名称" width="600" />
                <el-table-column label="分类操作">
                    <template #default="scope">
                        <el-button color="#13b9c9" class="white" @click="edit_(scope.row)">修改分类</el-button>
                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button color="#13b9c9" class="white">删除分类</el-button>
                            </template>
                        </el-popconfirm>

                    </template>
                </el-table-column>
            </el-table>
        </el-card>

        <!-- 对话框 -->
        <el-dialog v-model="dialogVisible" :title="dialogTitle" width="40%" @close="dialogClose">
            <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="120px" status-icon>
                <el-form-item label="分类名称" prop="cname">
                    <el-input v-model="ruleForm.cname" />
                </el-form-item>
            </el-form>

            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="dialogVisible = false">取消</el-button>
                    <el-button type="primary" @click="dialogEnter">
                        确定
                    </el-button>
                </span>
            </template>
        </el-dialog>
    </div>
</template>

<script lang="ts" setup>
import Bread from '../Bread.vue'
import { ref, reactive, getCurrentInstance, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import type { FormRules } from 'element-plus'
let { proxy }: any = getCurrentInstance()
let newsList = ref([])
let getNewsList = async () => {
    let { data: res } = await proxy.$http.get('news/cat/list')
    if (res.code !== 200) {
        ElMessage({
            message: "获取新闻列表失败",
            type: 'error',
            duration: 500
        })
    } else {
        newsList.value = res.data
    }

}
let dialogTitle = ref('')
// 对话框的显示和隐藏
let dialogVisible = ref(false)
// 表单数据
let ruleForm = reactive({
    cname: '',

})
const rules = reactive<FormRules>({
    cname: [
        { required: true, message: '请输入分类名称', trigger: 'blur' }
    ],
})
// 点击添加分类的按钮
function add_() {
    dialogTitle.value = '添加新闻分类'
    dialogVisible.value = true;
    Object.keys(ruleForm).forEach(key => {
        ruleForm[key] = ''
    })
}
const ruleFormRef = ref()
// 对话框的关闭事件
let dialogClose = () => {
    ruleFormRef.value.resetFields()
}
// 添加的请求
let addNews = async () => {
    let { data: res } = await proxy.$http.post('news/cat/add',
        { cname: ruleForm.cname })
    if (res.code !== 200) {
        ElMessage({
            message: "添加新闻分类失败",
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: "添加成功",
            type: 'success',
            duration: 500,
            onClose: () => {
                dialogVisible.value = false;
                getNewsList()
            }
        })
    }

}
// 编辑的按钮
let idd = ''
let edit_ = (row: any) => {
    // console.log(row.id);
    idd = row.id;
    ruleForm.cname = row.cname
    dialogTitle.value = '修改分类'
    dialogVisible.value = true
}
// 修改的请求
let editNews = async () => {
    let { data: res } = await proxy.$http.put('news/cat/update', {
        cname: ruleForm.cname,
        id: idd
    })
    if (res.code !== 200) {
        ElMessage({
            message: "修改失败",
            type: 'error',
            duration: 500
        })
    } else {
        dialogVisible.value = false
        ElMessage({
            message: "修改成功",
            type: 'success',
            duration: 500,
            onClose: () => {
                getNewsList()
            }
        })
    }
}
// 删除按钮
let delete_ = async (id: any) => {
    let { data: res } = await proxy.$http.delete(`news/cat?id=${id}`)
    if (res.code !== 200) {
        ElMessage({
            message: "删除失败",
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: "删除成功",
            type: 'success',
            duration: 500,
            onClose: () => {
                getNewsList()
            }
        })
    }


}
// 确定按钮
let dialogEnter = () => {
    ruleFormRef.value.validate((valid: boolean) => {
        if (!valid) {
            ElMessage({
                message: "请输入表单内容",
                type: 'error',
                duration: 500
            })
        } else {
            if (dialogTitle.value == '添加新闻分类') {
                addNews()
            }
            if (dialogTitle.value == '修改分类') {
                editNews()
            }

        }
    })

}
onMounted(() => {
    getNewsList()
})
</script>

<style lang="less" scoped></style>