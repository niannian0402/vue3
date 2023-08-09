<template>
    <div class="">
        <Bread></Bread>
        <el-card class="box-card">
            <template #header>
                <div class="card-header">
                    <el-button color="#13b9c9" class="white" @click="add_">医院添加</el-button>
                </div>
            </template>
            <!-- 表格区域 -->
            <el-table :data="hospitalList.datas" stripe style="width: 100%" border :header-cell-style="{
                backgroundColor: 'rgba(19, 185, 201, 0.7)',
                color: 'rgb(70, 69, 69)'
            }
                ">
                <el-table-column prop="title" label="医院名称" width="220" />
                <el-table-column prop="grade" label="医院级别" width="180">
                    <template #default="scope">
                        <span v-if="scope.row.grade == 1">三甲医院</span>
                        <span v-else-if="scope.row.grade == 2">三乙医院</span>
                        <span v-else-if="scope.row.grade == 3">二甲医院</span>
                        <span v-else>一甲医院</span>
                    </template>
                </el-table-column>
                <el-table-column prop="province" label="省分(直辖市)" />
                <el-table-column prop="addr" label="地址" />
                <el-table-column prop="phone" label="电话" />
                <el-table-column label="分类操作" width="400px">
                    <template #default="scope">
                        <el-button color="#13b9c9" class="white" @click="edit_(scope.row)">修改医院</el-button>
                        <el-button color="#13b9c9" class="white" @click="detail_(scope.row)">医院详情</el-button>
                        <el-popconfirm title="确定删除吗?" @confirm="delete_(scope.row.id)">
                            <template #reference>
                                <el-button color="#13b9c9" class="white">删除医院</el-button>
                            </template>
                        </el-popconfirm>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页器 -->
            <el-pagination v-model:current-page="hospitalList.pno" v-model:page-size="hospitalList.count"
                :page-sizes="[3, 5, 10, 12]" :small="small" :disabled="disabled" :background="background"
                layout="total, sizes, prev, pager, next, jumper" :total="hospitalList.total" @size-change="handleSizeChange"
                @current-change="handleCurrentChange" />
        </el-card>

        <!-- dialog对话框 -->
        <el-dialog v-model="dialogVisible" :title="dialogTitle" width="50%" @close="dialogClose">
            <el-form ref="ruleFormRef" :model="ruleForm" :rules="rules" label-width="120px" status-icon>
                <el-form-item label="医院名称:" prop="title">
                    <el-input v-if="dialogTitle !== '医院详情'" v-model="ruleForm.title" />
                    <span v-else>{{ ruleForm.title }}</span>
                </el-form-item>
                <el-form-item label="级别:" prop="grade">
                    <el-select v-if="dialogTitle !== '医院详情'" v-model="ruleForm.grade" clearable placeholder="请选择">
                        <el-option label="三甲医院" value="三甲医院" />
                        <el-option label="三乙医院" value="三乙医院" />
                        <el-option label="二甲医院" value="二甲医院" />
                        <el-option label="一甲医院" value="一甲医院" />
                    </el-select>
                    <span v-else>{{ ruleForm.grade }}</span>
                </el-form-item>
                <el-form-item label="图片:" prop="pic">
                    <el-input v-if="dialogTitle !== '医院详情'" v-model="ruleForm.pic" />
                    <span v-else><img class="pic" :src="ruleForm.pic" alt=""></span>
                </el-form-item>
                <el-form-item label="省份:" prop="province">
                    <el-select v-if="dialogTitle !== '医院详情'" v-model="ruleForm.province" clearable placeholder="请选择">
                        <el-option value="北京市">北京市</el-option>
                        <el-option value="天津市">天津市</el-option>
                        <el-option value="上海市">上海市</el-option>
                        <el-option value="重庆市">重庆市</el-option>
                        <el-option value="河北省">河北省</el-option>
                        <el-option value="山西省">山西省</el-option>
                        <el-option value="辽宁省">辽宁省</el-option>
                        <el-option value="吉林省">吉林省</el-option>
                        <el-option value="黑龙江省">黑龙江省</el-option>
                        <el-option value="江苏省">江苏省</el-option>
                        <el-option value="浙江省">浙江省</el-option>
                        <el-option value="安徽省">安徽省</el-option>
                        <el-option value="福建省">福建省</el-option>
                        <el-option value="江西省">江西省</el-option>
                        <el-option value="山东省">山东省</el-option>
                        <el-option value="河南省">河南省</el-option>
                        <el-option value="湖北省">湖北省</el-option>
                        <el-option value="湖南省">湖南省</el-option>
                        <el-option value="广东省">广东省</el-option>
                        <el-option value="海南省">海南省</el-option>
                        <el-option value="四川省">四川省</el-option>
                        <el-option value="贵州省">贵州省</el-option>
                        <el-option value="云南省">云南省</el-option>
                        <el-option value="陕西省">陕西省</el-option>
                        <el-option value="甘肃省">甘肃省</el-option>
                        <el-option value="青海省">青海省</el-option>
                        <el-option value="台湾省">台湾省</el-option>
                        <el-option value="内蒙古自治区">内蒙古自治区</el-option>
                        <el-option value="广西壮族自治区">广西壮族自治区</el-option>
                        <el-option value="西藏自治区">西藏自治区</el-option>
                        <el-option value="宁夏回族自治区">宁夏回族自治区</el-option>
                        <el-option value="新疆维吾尔自治区">新疆维吾尔自治区</el-option>
                    </el-select>

                    <span v-else>{{ ruleForm.province }}</span>
                </el-form-item>
                <el-form-item label="地址:" prop="addr">
                    <el-input v-if="dialogTitle !== '医院详情'" v-model="ruleForm.addr" />
                    <span v-else>{{ ruleForm.addr }}</span>
                </el-form-item>
                <el-form-item label="电话:" prop="phone">
                    <el-input v-if="dialogTitle !== '医院详情'" v-model="ruleForm.phone" />
                    <span v-else v-html="ruleForm.phone"></span>
                </el-form-item>

            </el-form>
            <template #footer v-if="dialogTitle !== '医院详情'">
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
import type { FormInstance, FormRules } from 'element-plus'
let { proxy }: any = getCurrentInstance()
let hospitalList = reactive({
    datas: [],
    pno: 1,
    total: 9,
    count: 5
})
const small = ref(false)
const background = ref(false)
const disabled = ref(false)

let getHospitalList = async () => {
    let { data: res } = await proxy.$http.get(`hospital/list?count=${hospitalList.count}&pno=${hospitalList.pno}`)
    if (res.code !== 200) {
        ElMessage({
            message: '请求失败',
            type: 'error',
            duration: 500
        })
    } else {
        hospitalList.datas = res.data
        hospitalList.total = res.total
        hospitalList.pno = res.pno

    }
}
const handleSizeChange = (val: number) => {
    hospitalList.count = val
    getHospitalList()
}
const handleCurrentChange = (val: number) => {
    hospitalList.pno = val
    getHospitalList()
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
    grade: '',
    pic: '',
    province: '',
    addr: '',
    phone: '',


})
const rules = reactive<FormRules>({
    title: [{ required: true, message: '请输入医院名称', trigger: 'blur' }],
    grade: [{ required: true, message: '请输入级别', trigger: 'blur' }],
    pic: [{ required: true, message: '请输入图片路径', trigger: 'blur' }],
    province: [{ required: true, message: '请输入省市', trigger: 'blur' }],
    addr: [{ required: true, message: '请输入地址', trigger: 'blur' }],
    phone: [{ required: true, message: '请输入电话', trigger: 'blur' }],

})
// 添加医院
const add_ = () => {
    getHospitalList()
    dialogVisible.value = true
    dialogTitle.value = '添加医院'
    Object.keys(ruleForm).forEach(item => {
        ruleForm[item] = ""
    })
}
// 添加医院请求
const addHospital = async () => {
    let { data: res } = await proxy.$http.post('hospital/add', ruleForm)
    if (res.code !== 200) {
        ElMessage({
            message: '添加医院失败',
            type: 'error',
            duration: 500
        })
    } else {
        ElMessage({
            message: '添加医院成功',
            type: 'success',
            duration: 500,
            onClose: () => {
                getHospitalList()
                dialogVisible.value = false
            }
        })
    }
}

// 修改按钮
const edit_ = (row: any) => {
    ruleForm = row

    dialogVisible.value = true
    dialogTitle.value = '修改医院'
}
const editHospital = async () => {
    let { data: res } = await proxy.$http.put('hospital/update', ruleForm)
    if (res.code !== 200) {
        ElMessage({
            type: 'error',
            message: '修改医院成功',
            duration: 500
        })
    } else {
        ElMessage({
            type: 'success',
            message: '修改医院成功',
            duration: 500,
            onClose: () => {
                getHospitalList();
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
            if (dialogTitle.value == '添加医院') {
                addHospital()
            } if (dialogTitle.value == '修改医院') {
                editHospital()
            }
        }
    })
}
// 医院详情按钮
const detail_ = (row: any) => {
    getHospitalList()
    ruleForm = row
    dialogVisible.value = true
    dialogTitle.value = '医院详情'
}
// 删除
const delete_ = async (aid: any) => {
    let { data: res } = await proxy.$http.delete(`/hospital?id=${aid}`)
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
                getHospitalList()
            }
        })
    }

}

onMounted(() => {
    getHospitalList()
})


</script>

<style lang="less" scoped></style>