<template>
    <el-dialog
    title="用户"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
        <el-form-item label="用户账号" :label-width="formLabelWidth">
            <el-input v-model="dataForm.userUsername" style="width:200px"></el-input>
        </el-form-item>
        <el-form-item label="用户密码" :label-width="formLabelWidth">
             <el-input v-model="dataForm.userPassword" suffix-icon="el-icon-date" style="width:200px"></el-input>
        </el-form-item>
        <el-form-item label="用户姓名" :label-width="formLabelWidth">
             <el-input v-model="dataForm.userName"  autosize autocomplete="off" w></el-input>
        </el-form-item>
        <el-form-item label="用户电话" :label-width="formLabelWidth">
             <el-input v-model="dataForm.userPhone"  autosize autocomplete="off" w></el-input>
        </el-form-item>
        <el-form-item label="用户邮箱" :label-width="formLabelWidth">
             <el-input v-model="dataForm.userEmail"  autosize autocomplete="off" w></el-input>
        </el-form-item>
        <el-form-item label="用户地址" :label-width="formLabelWidth" >
             <el-input v-model="dataForm.userAddress"  autosize autocomplete="off" w></el-input>
        </el-form-item>
        <el-form-item label="房间照片" ref="userPhoto" prop="img">
            <el-input v-model="dataForm.userPhoto" v-if="false"></el-input>
            <el-upload
            class="avatar-uploader"
            :action="materialPictureAndText()"
            :on-success="handleAvatarSuccess"
            :show-file-list="false"
            :before-upload="beforeUpload2">
                <img style="width: 70px;height: 70px" v-if="imgUrl" :src="imgUrl" class="avatar">
                <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
        </el-form-item>
        <el-form-item label="用户状态" :label-width="formLabelWidth">
            <el-select v-model="dataForm.userStatus" placeholder="请选择">
                <el-option
                v-for="item in options"
                :key="item.statusId"
                :label="item.statusName"
                :value="item.statusId">
                </el-option>
            </el-select>
        </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
    data (){
        return {
            formLabelWidth: '80px',
            visible: false,
            dataForm: {
                userId: '',
                userUsername: '',
                userPassword: '',
                userName: '',
                userPhone: '',
                userEmail: '',
                userAddress: '',
                userPhoto: '',
                userStatus: ''

            },
             options: [
                 {statusId:1,statusName:'正常使用'},
                 {statusId:2,statusName:'已禁用'}
             ],
             dataRule: {
                userUsername: [
                    { required: true, message: '新闻标题不能为空', trigger: 'blur' }
                ],
                userPassword: [
                    { required: true, message: '新闻内容不能为空', trigger: 'blur' }
                ]
            },
            imgUrl:''
        }
    },
    methods: {
        init(id){
            //let username =this.$store.state.user.name
            //console.log(username);
            this.dataForm.userId=id;
            this.visible = true
            if (this.dataForm.userId) {
                    this.$http({
                    url: this.$http.adornUrl(`/info/user/info/${this.dataForm.userId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataForm.userUsername = data.user.userUsername
                        this.dataForm.userPassword = data.user.userPassword
                        this.dataForm.userName = data.user.userName
                        this.dataForm.userPhone = data.user.userPhone
                        this.dataForm.userEmail = data.user.userEmail
                        this.dataForm.userAddress = data.user.userAddress
                        this.dataForm.userPhoto = data.user.userPhoto
                        this.dataForm.userStatus = data.user.userStatus
                        this.imgUrl=this.$http.adornUrl(data.user.userPhoto)

                    }
                })
            }else{
                this.dataForm.userUsername = ''
                this.dataForm.userPassword = ''
                this.dataForm.userName = ''
                this.dataForm.userPhone = ''
                this.dataForm.userEmail = ''
                this.dataForm.userAddress = ''
                this.dataForm.userPhoto = ''
                this.dataForm.userStatus = ''
            }
        },
        // 表单提交
        dataFormSubmit (){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/info/user/update`),
                        method: 'post',
                        data: this.$http.adornData({
                            'userId': this.dataForm.userId,
                            'userUsername': this.dataForm.userUsername, 
                            'userPassword': this.dataForm.userPassword,
                            'userName': this.dataForm.userName, 
                            'userPhone': this.dataForm.userPhone, 
                            'userEmail': this.dataForm.userEmail, 
                            'userAddress': this.dataForm.userAddress, 
                            'userPhoto': this.dataForm.userPhoto, 
                            'userStatus': this.dataForm.userStatus, 
                    
                        })
                    }).then(({data}) => {
                        if (data && data.code === 0) {
                            this.$message({
                                message: '操作成功',
                                type: 'success',
                                duration: 1500,
                                onClose: () => {
                                    this.visible = false
                                    this.$emit('refreshDataList')
                                }
                            })
                        } else {
                            this.$message.error(data.msg)
                        }
                    })
                }
            })
        },
        //图片上传完后的回调函数
        handleAvatarSuccess (response,file) {
            if(response.code==0){
                console.log(response.success)
                this.imgUrl=this.$http.adornUrl(response.success)
                this.dataForm.userPhoto=response.success
            }else{
                this.$message({
                 type: 'info',
                message: '请重新上传'
            })  
            }
           
        },
        //照片上传
        materialPictureAndText () {
            return this.$http.adornUrl(`/info/user/savePicture?token=${this.$cookie.get('token')}`)  //前面是为了方便线上加的
        },
        //上传前的校验
        beforeUpload2 (file) {
            const isLt2M = file.size < 1024 * 1024 * 2
            // console.log('大小' + isLt2M)
            if (!isLt2M) {
            this.$message.error('上传文件大小不能超过 2MB!')
            }
            
            return isLt2M
        }
    }
}
</script>

<style>

</style>