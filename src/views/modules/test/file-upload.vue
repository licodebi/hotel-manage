<template>
    <el-dialog title="添加图文" :visible.sync="dialogVisible">
        <el-form :model="configForm" ref="configForm" label-width="100px"
                :rules="rules"
                v-loading="loading2">
            <h3 class="title modal-headline">添加图文</h3>
            <el-form-item label=" 图文标题" prop="imgTitle">
            <el-input v-model="configForm.imgTitle"
                        type="text"
                        auto-complete="off" placeholder="请输入标题"></el-input>
            </el-form-item>
            <el-form-item label="添加文字" prop="imgText">
            <el-input v-model="configForm.imgText"
                        type="textarea"
                        auto-complete="off" placeholder="请输入文字内容"></el-input>
            </el-form-item>
            <el-form-item label="添加图片" ref="uploadElement" prop="img">
            <el-input v-model="configForm.img" v-if="false"></el-input>
            <el-upload
                class="upload-demo"
                :on-success="handleAvatarSuccess"
                accept="image/jpeg,image/jpg,image/png"
                :action="materialPictureAndText()"
                :auto-upload="false"
                :before-upload="beforeUpload2"
                ref="upload"
                :on-change="handleChange"
                multiple
                :data="configForm"> 
                <el-button type="success" size="small">选择文件</el-button>
            </el-upload>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button type="primary"
                    @click="submitImg('configForm')" >确 定</el-button>
            <el-button @click="dialogVisible = false">取 消</el-button>
        </div>
    </el-dialog>
</template>

<script>
export default {
    data() {
        return {
            dialogVisible: false,
            configForm: {
                imgTitle: '',
                imgText: '',
                img: ''
            },
            loading2: false,
            rules: {
                imgTitle: [
                    {required: true, message: '请输入活动名称', trigger: 'blur'},
                ]
            }
        }
    },
    methods: {
        init(){
            this.dialogVisible=true;
        },
        submitImg (forName) {
            let self = this
            this.$confirm('此操作将无法撤回, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(() => {
            self.$refs[forName].validate((valid) => {
                if (valid) {
                self.$refs.upload.submit()
                self.dialogVisible = false
                self.loading2 = false
                } else {
                return false
                }
            })
            }).catch(() => {
            this.$message({
                type: 'info',
                message: '已取消更新'
            })
            })
        },
 
        handleAvatarSuccess (response, file) {
            let self = this
            let resp = response
            console.log(response)
            if (resp.result === 200) {
            setTimeout(() => {
                self.dialogVisible = false//关闭弹窗
        
                self.$refs.upload.clearFiles()
                self.$message.success(resp.msg)//显示后台信息
                self.getImgData()//上传后刷新表单
            }, 1000)
            } else {
                self.$message.error(resp.msg)//显示后台信息
                self.$refs.upload.clearFiles()//清空表单
            }
        },
        materialPictureAndText () {
            return this.$http.adornUrl(`/pictureManage/materialPicture?token=${this.$cookie.get('token')}`)  //前面是为了方便线上加的
        },
        beforeUpload2 (file) {
            const isLt2M = file.size < 1024 * 1024 * 2
            // console.log('大小' + isLt2M)
            if (!isLt2M) {
            this.$message.error('上传文件大小不能超过 2MB!')
            }
            let size = file.size / 1024
            console.log('大小' + size)
            let _URL = window.URL || window.webkitURL
            let img = new Image()
            img.onload = function () {
                let width = img.width
                let height = img.height
                console.log('width--->' + width)
                console.log('height--->' + height)
            }
            img.src = _URL.createObjectURL(file)
            return isLt2M
        }
    }
}
</script>

<style>

</style>