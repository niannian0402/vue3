<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button color="#13b9c9" class="white" @click="add_">添加文章</el-button>
                </div>
            </template>
            <el-table :data="articleList" stripe style="width: 100%" border :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column prop="title" label="文章标题" width="380" />
                <el-table-column prop="author" label="作者" width="180" />
                <el-table-column prop="cat_id" label="分类" />
                <el-table-column label="分类操作">
                    <template #default="scope">
                        <el-button color="#13b9c9" class="white" @click="edit_(scope.row)">修改文章</el-button>
                        <el-button color="#13b9c9" class="white" @click="detail_(scope.row)">文章详情</el-button>
                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.aid)">
                            <template #reference>
                                <el-button color="#13b9c9" class="white">删除文章</el-button>
                            </template>
                        </el-popconfirm>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页器 -->
            <el-pagination v-model:current-page="pno" v-model:page-size="count" :page-sizes="[2, 3, 5, 10]"
                layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
                @current-change="handleCurrentChange" />
        </el-card>
        <!-- dialog对话框 -->
        <el-dialog v-model="dialogVisible" :title="dialogTitle" width="50%" @close="dialogClose">
            <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="120px" status-icon>
                <el-form-item label="标题:" prop="title">
                    <el-input v-if="dialogTitle !== '文章详情'" v-model="ruleForm.title" />
                    <span v-else>{{ ruleForm.title }}</span>
                </el-form-item>
                <el-form-item label="分类:" prop="cat_id">
                    <el-select v-if="dialogTitle !== '文章详情'" v-model="ruleForm.cat_id" clearable placeholder="请选择">
                        <el-option label="美女" value="美女" />
                        <el-option label="军事" value="军事" />
                        <el-option label="教育" value="教育" />
                        <el-option label="科学" value="科学" />
                        <el-option label="体育" value="体育" />
                        <el-option label="时尚" value="时尚" />
                        <el-option label="金融" value="金融" />
                    </el-select>
                    <span v-else>{{ ruleForm.cat_id }}</span>
                </el-form-item>
                <el-form-item label="图片:" prop="pic">
                    <el-input v-if="dialogTitle !== '文章详情'" v-model="ruleForm.pic" />
                    <span v-else><img class="pic" :src="ruleForm.pic" alt=""></span>
                </el-form-item>
                <el-form-item label="作者:" prop="author">
                    <el-input v-if="dialogTitle !== '文章详情'" v-model="ruleForm.author" />
                    <span v-else>{{ ruleForm.author }}</span>
                </el-form-item>
                <el-form-item label="文章详情:" prop="detail">
                    <el-input v-if="dialogTitle !== '文章详情'" v-model="ruleForm.detail" />
                    <span v-else v-html="ruleForm.detail"></span>
                </el-form-item>
                <el-form-item label="创建时间:" prop="ctime">
                    <el-input v-if="dialogTitle !== '文章详情'" v-model="ruleForm.ctime" />
                    <span v-else>{{ ruleForm.ctime }}</span>
                </el-form-item>
                <el-form-item label="is_rec:" prop="is_rec">
                    <el-select v-if="dialogTitle !== '文章详情'" v-model="ruleForm.is_rec" clearable placeholder="请选择">
                        <el-option label="0" value="0" />
                        <el-option label="1" value="1" />
                    </el-select>
                    <span v-else>{{ ruleForm.is_rec }}</span>
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
import { ElMessage } from 'element-plus'
import { ref, reactive, getCurrentInstance, onMounted } from 'vue'
import type { FormInstance, FormRules } from 'element-plus'

let { proxy }: any = getCurrentInstance()
let articleList = ref([])
let count = ref(10)
let pno: any = ref(1)
let total = ref(3)

const getArticleList = async () => {
    let { data: res } = await proxy.$http.get(`news/article/list?count=${count.value}&pno=${pno.value}`)
    // console.log(res);
    if (res.code !== 200) {
        // console.log('请求失败');
        ElMessage({
            message: "请求失败",
            type: 'error',
            duration: 500
        })

    } else {
        articleList.value = res.data
        pno.value = res.pno
        total.value = res.total
    }

}
// 分页器事件
const handleSizeChange = (newval: number) => {
    count.value = newval
    getArticleList()
}
const handleCurrentChange = (val: any) => {
    pno.value = val
    getArticleList()
}
// dialog
const dialogVisible = ref(false)
const dialogTitle = ref()
const ruleFormRef = ref<FormInstance>()
const dialogClose = () => {
    ruleFormRef.value.resetFields()
}
// 表单
let ruleForm = reactive({
    title: '',
    cat_id: '',
    pic: '',
    author: '',
    detail: '',
    ctime: '',
    is_rec: ''
})
const rules = reactive<FormRules>({
    title: [{ required: true, message: '请输入标题', trigger: 'blur' }],
    nacnameme: [{ required: true, message: '请输入分类', trigger: 'blur' }],
    pic: [{ required: true, message: '请输入图片路径', trigger: 'blur' }],
    author: [{ required: true, message: '请输入作者', trigger: 'blur' }],
    detail: [{ required: true, message: '请输入文章详情', trigger: 'blur' }],
    ctime: [{ required: true, message: '请输入创建时间', trigger: 'blur' }],
    is_rec: [{ required: true, message: '请输入创建时间', trigger: 'blur' }],
})
// 添加文章
const add_ = () => {
    getArticleList()
    dialogVisible.value = true
    dialogTitle.value = '添加文章'
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ""
    })
}
// 添加文章请求
const addArticle = async () => {
    let { data: res } = await proxy.$http.post('news/article/add', ruleForm)
    if (res.code !== 200) {
        ElMessage({
            message: '添加文章失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '添加文章成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                getArticleList()
                dialogVisible.value = false
            }
        })
    }
}

// 修改按钮
const edit_ = (row: any) => {
    ruleForm = row
    dialogVisible.value = true
    dialogTitle.value = '修改文章'
}
const editArticle = async () => {
    let { data: res } = await proxy.$http.put('news/article/update', ruleForm)
    if (res.code !== 200) {
        ElMessage({
            type: 'error',
            message: '修改文章成功',
            duration: 500
        })
    } else {
        ElMessage({
            type: 'success',
            message: '修改文章成功',
            duration: 500,
            onClose: () => {
                getArticleList();
                dialogVisible.value = false
            }
        })
    }

}

// 确定按钮
const dialogEnter = () => {
    ruleFormRef.value.validate((valid) => {
        if (!valid) {
            ElMessage({
                type: 'error',
                message: '请输入表单内容',
                duration: 500
            })
        } else {
            if (dialogTitle.value == '添加文章') {
                addArticle()
            } if (dialogTitle.value == '修改文章') {
                editArticle()
            }
        }
    })
}
// 文章详情按钮
const detail_ = (row: any) => {
    getArticleList()
    ruleForm = row
    dialogVisible.value = true
    dialogTitle.value = '文章详情'
}
// 删除
const delete_ = async (aid: any) => {
    let { data: res } = await proxy.$http.delete(`news/article?id=${aid}`)
    if (res.code !== 200) {
        ElMessage({
            message: '删除失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '删除成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                getArticleList()
            }
        })
    }

}
onMounted(() => {
    getArticleList()
})
</script>

<style lang="less" scoped></style>