<template>
  <el-dialog
    :title="!dataForm.cateId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="分类名称" prop="cateName">
        <el-input v-model="dataForm.cateName" placeholder="分类名称"></el-input>
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
            cateId: 0,
            cateName: ''
            },
            dataRule: {
                cateName: [
                    { required: true, message: '分类名不能为空', trigger: 'blur' }
                ]
            }
           
        }
    },
    methods: {
        init (id){
            this.dataForm.cateId=id || 0
            this.visible = true
            
            if (this.dataForm.cateId) {
                    this.$http({
                    url: this.$http.adornUrl(`/room/equipcategory/info/${this.dataForm.cateId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataForm.cateName = data.equipCategory.cateName
                    }
                })
            }else{
                this.dataForm.cateName=''
            }
            
        },
        // 表单提交
        dataFormSubmit (){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/room/equipcategory/${!this.dataForm.cateId ? 'save' : 'update'}`),
                        method: 'post',
                        data: this.$http.adornData({
                            'cateId': this.dataForm.cateId || undefined,
                            'cateName': this.dataForm.cateName,
                    
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