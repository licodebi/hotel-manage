<template>
  <el-dialog
    :title="!dataForm.cateId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="新闻标题" prop="newsTitle">
        <el-input v-model="dataForm.newsTitle" placeholder="新闻标题"></el-input>
      </el-form-item>
      <el-form-item label="新闻内容" prop="newsContent">
        <el-input v-model="dataForm.newsContent" placeholder="新闻内容"></el-input>
      </el-form-item>
      <el-form-item label="创建日期" prop="newsCreateTime">
        <el-date-picker
           v-model="dataForm.newsCreateTime"
            type="datetime"
            placeholder="选择日期时间"
            value-format="yyyy-MM-dd HH:mm:ss">
        </el-date-picker>
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
        return{
            visible: false,
            dataForm: {
            newsId: 0,
            newsTitle: '',
            newsContent: '',
            newsCreateTime: '',
            },
            dataRule: {
                newsTitle: [
                    { required: true, message: '新闻标题不能为空', trigger: 'blur' }
                ],
                newsContent: [
                    { required: true, message: '新闻内容不能为空', trigger: 'blur' }
                ]
            }
           
        }
    },
    methods: {
        init (id){
            this.dataForm.newsId=id || 0
            this.visible = true
            
            if (this.dataForm.newsId) {
                    this.$http({
                    url: this.$http.adornUrl(`/info/news/info/${this.dataForm.newsId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataForm.newsTitle = data.news.newsTitle
                        this.dataForm.newsContent = data.news.newsContent
                        this.dataForm.newsCreateTime = data.news.newsCreateTime
                    }
                })
            }else{
                this.dataForm.newsTitle=''
                this.dataForm.newsContent=''
                this.dataForm.newsCreateTime=''

            }
            
        },
        // 表单提交
        dataFormSubmit (){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/info/news/${!this.dataForm.newsId ? 'save' : 'update'}`),
                        method: 'post',
                        data: this.$http.adornData({
                            'newsId': this.dataForm.newsId || undefined,
                            'newsTitle': this.dataForm.newsTitle,
                            'newsContent': this.dataForm.newsContent,
                            'newsCreateTime': this.dataForm.newsCreateTime
                    
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