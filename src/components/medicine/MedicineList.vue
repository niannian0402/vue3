<template>
    <div class="box">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button color="#13b9c9" class="white" @click="add_">添加药品</el-button>
                </div>
            </template>
            <el-table :data="medicineList" border style="width: 100%" :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }">
                <el-table-column type="index" />
                <el-table-column prop="title" label="药名" width="270px" />
                <el-table-column prop="type" label="类型" />
                <el-table-column prop="sale_price" label="原价" />
                <el-table-column prop="market_price" label="活动价" />
                <el-table-column prop="company" label="生产厂家" width="400px" />
                <el-table-column prop="" label="操作" width="300px">
                    <template #default="scope">
                        <el-button color="#13b9c9" class="white" :icon="Edit" size="small"
                            @click="edit_(scope.row)">修改</el-button>
                        <el-button color="#13b9c9" class="white" :icon="MessageBox" size="small"
                            @click="detail_(scope.row.id)">详情</el-button>

                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button color="#13b9c9" class="white" :icon="Delete" size="small">删除</el-button>
                            </template>
                        </el-popconfirm>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页器 -->
            <el-pagination v-model:current-page="pno" v-model:page-size="count" :page-sizes="[10]"
                layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange"
                @current-change="handleCurrentChange" />
        </el-card>

        <!-- dialog 的对话框 -->
        <el-dialog v-model="dialogVisible" :title="dialogTitle" width="50%" @close="dialogClose">
            <el-form ref="dialogFormRef" :model="dialogForm" :rules="dialogFormRules" label-width="120px"
                class="demo-ruleForm" status-icon>
                <el-form-item label="药品名称:" prop="title">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.title" />
                    <span v-else>{{ dialogForm.title }}</span>
                </el-form-item>
                <el-form-item label="原价:" prop="sale_price">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.sale_price" />
                    <span v-else>{{ dialogForm.sale_price }}</span>
                </el-form-item>
                <el-form-item label="活动价:" prop="market_price">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.market_price" />
                    <span v-else>{{ dialogForm.market_price }}</span>
                </el-form-item>
                <el-form-item label="类型:" prop="type">
                    <el-select v-if="dialogTitle !== '药品详情'" v-model="dialogForm.type" placeholder="请选择药品类型">
                        <el-option label="中成药" value="中成药" />
                        <el-option label="西药" value="西药" />
                        <el-option label="中药" value="中药" />
                        <el-option label="其他" value="其他" />
                    </el-select>
                    <span v-else>{{ dialogForm.type }}</span>
                </el-form-item>
                <el-form-item label="规格:" prop="unit">
                    <el-select v-if="dialogTitle !== '药品详情'" v-model="dialogForm.unit" placeholder="请选择药品类型">
                        <el-option label="片" value="片" />
                        <el-option label="粒" value="粒" />
                        <el-option label="盒" value="盒" />
                        <el-option label="贴" value="贴" />
                    </el-select>
                    <span v-else>{{ dialogForm.unit }}</span>
                </el-form-item>
                <el-form-item label="图片:" prop="pic">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.pic" />
                    <img v-else class="pic" :src="dialogForm.pic" alt="">
                </el-form-item>
                <el-form-item label="功能主治:" prop="fun">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.fun" />
                    <span v-else>{{ dialogForm.fun }}</span>
                </el-form-item>
                <el-form-item label="不良反应:" prop="bad">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.bad" />
                    <span v-else>{{ dialogForm.bad }}</span>
                </el-form-item>
                <el-form-item label="禁忌:" prop="need">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.need" />
                    <span v-else>{{ dialogForm.need }}</span>
                </el-form-item>
                <el-form-item label="生产企业:" prop="company">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.company" />
                    <span v-else>{{ dialogForm.company }}</span>
                </el-form-item>
                <el-form-item label="批准文号:" prop="num">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.num" />
                    <span v-else>{{ dialogForm.num }}</span>
                </el-form-item>
                <el-form-item label="详细介绍:" prop="detail">
                    <el-input v-if="dialogTitle !== '药品详情'" v-model="dialogForm.detail" type="textarea" />
                    <span v-else v-html="dialogForm.detail"></span>
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
import { ref, reactive, getCurrentInstance, onMounted, watch } from 'vue'
// 引入图标
import { Delete, MessageBox, Edit } from '@element-plus/icons-vue'
import type { FormRules } from 'element-plus'
import { ElMessage } from 'element-plus'
let { proxy }: any = getCurrentInstance()
let medicineList = ref([])
let total = ref()
const count = ref(10)
const pno = ref(1)

let getMedicineList = async () => {
    let { data: res } = await proxy.$http.get(`/medicine/list?count=${count.value}&pno=${pno.value}`)
    // console.log(res);

    if (res.code !== 200) {
        return
    } else {
        medicineList.value = res.data
        total.value = res.total

    }
}
const handleSizeChange = (newpages: number) => {
    // console.log(newpages);
    count.value = newpages;
    getMedicineList()

}
const handleCurrentChange = (newpno: number) => {
    // console.log(newpno);
    pno.value = newpno;
    getMedicineList()
}
const dialogVisible = ref(false)
const dialogTitle = ref('')
// 药品数据
let dialogForm = reactive({
    title: '',
    sale_price: '',
    market_price: '',
    type: '',
    unit: '',
    pic: '',
    fun: '',
    bad: '',
    need: '',
    company: '',
    num: '',
    detail: ''
})
// 验证规则
const dialogFormRef = ref()
const dialogFormRules = reactive<FormRules>({
    title: [{ required: true, message: '请输入药品名称', trigger: 'blur' }],
    sale_price: [{ required: true, message: '请输入药品原价', trigger: 'blur' }],
    market_price: [{ required: true, message: '请输入药品的活动价', trigger: 'blur' }],
    type: [{ required: true, message: '请输入药品类型', trigger: 'blur' }],
    unit: [{ required: true, message: '请输入药品规格', trigger: 'blur' }],
    pic: [{ required: true, message: '请输入图片路径', trigger: 'blur' }],
    fun: [{ required: true, message: '请输入功能主治', trigger: 'blur' }],
    bad: [{ required: true, message: '请输入不良反应', trigger: 'blur' }],
    need: [{ required: true, message: '请输入禁忌', trigger: 'blur' }],
    company: [{ required: true, message: '请输入生产企业', trigger: 'blur' }],
    num: [{ required: true, message: '请输入批号', trigger: 'blur' }],
    detail: [{ required: true, message: '请填写详细介绍', trigger: 'blur' }],
})

// 添加药品
const add_ = () => {
    dialogTitle.value = '添加药品'
    dialogVisible.value = true
    //所有的数据 清空
    Object.keys(dialogForm).forEach(key => {
        dialogForm[key] = ''
    })
}
// 修改按钮
const edit_ = (row) => {
    // console.log(row);
    dialogForm = row
    dialogTitle.value = '修改药品'
    dialogVisible.value = true
}
// 修改按钮时提交确定
const editMedicine = async () => {
    let { data: res } = await proxy.$http.put('medicine/update', dialogForm)
    if (res.code !== 200) {
        ElMessage({
            message: '修改药品失败',
            type: 'error',
            duration: 500,
        })
    } else {
        ElMessage({
            message: '修改药品成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                dialogVisible.value = false
                getMedicineList()
            }
        })
    }

}
// 关闭对话框时
const dialogClose = () => {
    getMedicineList()
    dialogFormRef.value.resetFields()
    Object.keys(dialogForm).forEach(item => {
        dialogForm[item] = ''
    })
}
// 发起添加药品请求
const addMedicine = async () => {
    let { data: res } = await proxy.$http.post('medicine/add', dialogForm)
    if (res.code !== 200) {
        ElMessage({
            message: '添加药品失败',
            type: 'error',
            duration: 500,
        })
    } else {
        ElMessage({
            message: '添加药品成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                dialogVisible.value = false
                getMedicineList()
            }
        })
    }
}
// 详情药品
let detail_ = async (id: any) => {
    let { data: res } = await proxy.$http.get(`medicine/detail?id=${id}`)
    if (res.code !== 200) {
        ElMessage({
            message: '药品详情获取失败',
            type: 'error',
            duration: 500,
        })
    } else {
        dialogForm = res.data[0]
        dialogTitle.value = '药品详情'
        dialogVisible.value = true
    }
}

// 确定按钮时
const dialogEnter = () => {
    dialogFormRef.value.validate((valid: boolean) => {
        if (!valid) {
            ElMessage({
                message: '请填写表单内容',
                type: 'error',
                duration: 500,
            })
        }
        else {
            if (dialogTitle.value == '添加药品') {
                addMedicine()
            } if (dialogTitle.value == '修改药品') {
                editMedicine()
            }


        }
    })
}

// 删除
const delete_ = async (id: any) => {
    let { data: res } = await proxy.$http.delete(`medicine?id=${id}`)
    if (res.code !== 200) {
        ElMessage({
            message: '删除药品失败',
            type: 'error',
            duration: 500,
        })
    } else {
        ElMessage({
            message: '删除药品成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                getMedicineList()
            }
        })
    }

}
onMounted(() => {
    getMedicineList()
})
</script>

<style lang="less" scoped>
.pic {
    width: 300px;
}
</style>