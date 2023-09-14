<template>
    <div class="text-center pt-15 flex flex-col">
        <h1>欢迎加入计算机协会</h1>
        <div class="my-4 p-5 md:p-10 bg-[#f5f5f7] rounded-lg mxa md:max-w-[1000px] w-[80%] md:w-[480px]">
            <el-form :model="form" label-width="80px" class="">
                <el-form-item label="学号" :rules="[
                    { required: true, message: '学号为必填项' },
                    { type: 'number', message: '学号必须为数字' },
                ]">
                    <el-input v-model="form.studentId"></el-input>
                </el-form-item>
                <el-form-item label="姓名" :rules="[
                    { required: true, message: '姓名为必填项' },
                ]">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="支付截图" :rules="[
                    { required: true, message: '支付截图为必填项' },
                ]">
                    <el-upload ref="upload" class="upload-demo w-full" drag :limit="1" :on-exceed="handleExceed"
                        action="https://run.mocky.io/v3/9d059bf9-4660-45f2-925d-ce80ad6c4d15" :auto-upload="false"
                        accept="image/png, image/jpeg" :on-change="onChange" v-model:file-list="userFile">
                        <el-icon class="el-icon--upload"><upload-filled /></el-icon>
                        <div class="el-upload__text">
                            点击上传 <b><em>支付账单截图</em></b><br>
                            需包含 <b>转账单号</b> 以供核对<br>
                            可在 <b>微信支付 - 微信支付凭证 - 查看账单详情</b> 中查看
                        </div>
                        <template #tip>
                            <div class="el-upload__tip">
                                还没有支付？请联系计协的同学询问收款码，支付后再上传截图
                            </div>
                        </template>
                    </el-upload>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="onSubmit">提交</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>

<script lang="ts" setup>

import { ref, reactive } from 'vue'
import { UploadFilled } from '@element-plus/icons-vue'
import type { UploadFile, UploadInstance, UploadProps, UploadRawFile } from 'element-plus'
import { ElNotification, genFileId } from 'element-plus'

let form = reactive({
    studentId: '',
    name: '',
    // desc: ''
})

const upload = ref<UploadInstance>()
const userFile = ref<UploadFile[]>([])

const onSubmit = async () => {
    if (form.studentId === '' || form.name === '') {
        // 弹窗提示
        ElNotification.error({
            title: '报名失败',
            message: '请填写学号和姓名信息',
        })
        return
    }

    if (form.studentId.length !== 9) {
        ElNotification.error({
            title: '报名失败',
            message: '学号格式错误，请输入9位学号',
        })
        return
    }

    let file: UploadRawFile
    if (!userFile.value || userFile.value.length === 0) {
        ElNotification.error({
            title: '报名失败',
            message: '请先上传支付截图',
        })
        return
    }
    try {
        file = userFile.value[0].raw as UploadRawFile
    } catch (e: any) {
        ElNotification.error({
            title: '报名失败',
            message: '请先上传支付截图' + e.message,
        })
        return
    }
    console.log(form, file)
    // 创建一个 formData 对象
    const formData = new FormData()
    // 将文件对象添加到 formData 对象中
    formData.append('file', file)
    formData.append('studentId', form.studentId)
    formData.append('name', form.name)

    const { data: result, error }: { data: Ref<any>, error: Ref<any> } = await useFetch('http://1.117.214.115:2016/join', {
        headers: {
            'Accept': 'application/json',
        },
        method: 'POST',
        body: formData
    })
    console.log(result.value)
    if (error.value) {
        ElNotification.error({
            title: '网络错误',
            message: '请求失败，' + error.value?.message,
        })
        return
    }

    if (result.value.code !== 200) {
        ElNotification.error({
            title: '加入失败',
            message: result.value.msg,
        })
        return
    }


    ElNotification.success({
        title: '加入成功',
        message: result.value.msg,
    })
}

const handleExceed: UploadProps['onExceed'] = (files) => {
    if (!upload.value) return
    upload.value.clearFiles()
    const file = files[0] as UploadRawFile
    file.uid = genFileId()
    upload.value.handleStart(file)
}

const onChange: UploadProps['onChange'] = (rawFile) => {
    if (!rawFile) return
    if (rawFile.raw?.type !== 'image/jpeg' && rawFile.raw?.type !== 'image/png') {
        ElNotification.error({
            title: '支付截图上传失败',
            message: '只能上传jpg/png文件',
        })
        upload.value!.clearFiles()
        return false
    } else if (rawFile.raw?.size / 1024 / 1024 > 2) {
        ElNotification.error({
            title: '支付截图上传失败',
            message: '图片文件大小不能超过2MB',
        })
        upload.value!.clearFiles()
        return false
    }
    return true
}
</script>
  