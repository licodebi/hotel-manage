<template>
    <el-dialog
    title="修改"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="评论人" >
        <el-input v-model="dataForm.user.userUsername" placeholder="评论人"></el-input>
      </el-form-item>
      <!-- <el-form-item label="时间" >
        <el-input v-model="dataForm.mesCreateTime" placeholder="留言时间"></el-input>
      </el-form-item> -->
      <el-form-item label="评论房间" >
        <el-input v-model="dataForm.room.roomName" placeholder="评论房间"></el-input>
      </el-form-item>
      <el-form-item label="评论内容" >
        <el-input v-model="dataForm.comContent" placeholder="回复内容"></el-input>
      </el-form-item>
      <el-form-item label="用户评分" >
        <el-input v-model="dataForm.comGrade" placeholder="回复内容"></el-input>
      </el-form-item>
      <el-form-item label="回复内容" >
        <el-input v-model="dataForm.comReplyContent" placeholder="回复内容"></el-input>
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
                comId: 0,
                user: '',
                // mesCreateTime: '',
                room: '',
                comContent:'',
                comGrade: '',
                comReplyContent:''

            },
             dataRule: {
                user: [
                    { required: true, message: '用户名不能为空', trigger: 'bludr' }  
                ],
                room:[{
                    required: true, message: '房间名不能为空', trigger: 'bludr'
                }],
                comContent:[{
                    required: true, message: '评论不能为空', trigger: 'bludr'
                }],
                
                comGrade:[{
                    required: true, message: '评分不能为空', trigger: 'bludr'
                }]
            }
        }
    },
    methods: {
        init(id){
            this.dataForm.comId=id;
            this.visible = true
             if (this.dataForm.comId) {
                    this.$http({
                    url: this.$http.adornUrl(`/info/comment/info/${this.dataForm.comId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataForm.user = data.comment.user
                        // this.dataForm.mesCreateTime = data.message.mesCreateTime
                        this.dataForm.room = data.comment.room
                        this.dataForm.comContent= data.comment.comContent
                        this.dataForm.comGrade = data.comment.comGrade
                        this.dataForm.comReplyContent = data.comment.comReplyContent
                    }
                })
            }else{
                this.dataForm.user=''
                this.dataForm.room=''
                this.dataForm.comContent=''
                this.dataForm.comGrade=''
                this.dataForm.comReplyContent=''
            }
        },
        // 表单提交
        dataFormSubmit (){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/info/comment/update`),
                        method: 'post',
                        data: this.$http.adornData({
                            'comId': this.dataForm.comId,
                            'comContent': this.dataForm.comContent,
                            'comGrade': this.dataForm.comGrade,
                            'comReplyContent': this.dataForm.comReplyContent
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