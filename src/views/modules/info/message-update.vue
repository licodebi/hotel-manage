<template>
    <el-dialog
    title="回复"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="留言人" >
        <el-input v-model="dataForm.user.userUsername" placeholder="留言人"></el-input>
      </el-form-item>
      <el-form-item label="留言时间" >
        <el-input v-model="dataForm.mesCreateTime" placeholder="留言时间"></el-input>
      </el-form-item>
      <el-form-item label="留言内容" >
        <el-input v-model="dataForm.mesContent" placeholder="留言内容"></el-input>
      </el-form-item>
      <el-form-item label="回复内容" >
        <el-input v-model="dataForm.mesReply" placeholder="回复内容"></el-input>
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
            visible: false,
            dataForm: {
                mesId: 0,
                mesNickName: '',
                mesCreateTime: '',
                mesContent: '',
                mesReply:'',
                user:''

            },
        }
    },
    methods: {
        init(id){
            let username =this.$store.state.user.name
            //console.log(username);
            this.dataForm.mesId=id;
            this.visible = true
             if (this.dataForm.mesId) {
                    this.$http({
                    url: this.$http.adornUrl(`/info/message/info/${this.dataForm.mesId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataForm.mesNickName = data.message.mesNickName
                        this.dataForm.mesCreateTime = data.message.mesCreateTime
                        this.dataForm.mesContent = data.message.mesContent
                        this.dataForm.mesReply = data.message.mesReply
                        this.dataForm.user = data.message.user
                    }
                })
            }else{
                this.dataForm.mesNickName=''
                this.dataForm.mesCreateTime=''
                this.dataForm.mesContent=''
                this.dataForm.user=''
                this.dataForm.mesReply=''
            }
        },
        // 表单提交
        dataFormSubmit (){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/info/message/update`),
                        method: 'post',
                        data: this.$http.adornData({
                            'mesId': this.dataForm.mesId,
                            'mesNickName': this.dataForm.mesNickName,
                            'mesCreateTime': this.dataForm.mesCreateTime,
                            'mesReply' : this.dataForm.mesReply,
                            'mesReplyName' : this.$store.state.user.name,
                            'mesStatus' : 2
                    
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
        }
    }
}
</script>

<style>

</style>