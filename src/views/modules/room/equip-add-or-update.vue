<template>
  <el-dialog
    :title="!dataForm.equipId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="设备名称" prop="equipName">
        <el-input v-model="dataForm.equipName" placeholder="设备名称"></el-input>
      </el-form-item>
      
      <el-form-item label="设备图标" prop="iconId">
          <el-select v-model="dataForm.iconId" placeholder="请选择" >
            <el-option
            v-for="item in iconOptions"
            :key="item.font_class"
            :label="item.name"
            :value="item.font_class">
                <span :class="lookIcon(item.font_class)"></span>
            </el-option>
          </el-select>
      </el-form-item>
      <el-form-item label="设备分类" prop="equipCateId">
          <el-select v-model="dataForm.equipCateId" placeholder="请选择" >
            <el-option
            v-for="item in cateOptions"
            :key="item.cateId"
            :label="item.cateName"
            :value="item.cateId">
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
import axios from 'axios'
export default {
    data (){
        return{
            visible: false,
            dataForm: {
                equipId: 0,
                equipName: '',
                iconId:'',
                equipCateId:''
            },
            dataRule: {
                equipName: [
                    { required: true, message: '设备名不能为空', trigger: 'blur' }
                ],
                iconId: [
                    { required: true, message: '图标不能为空', trigger: 'blur' }
                ],
                equipCateId: [
                    { required: true, message: '设备分类不能为空', trigger: 'blur' }
                ],
            },
            cateOptions:[],
            iconOptions:[]
           
        }
    },
    created (){
        const newInstance=axios.create({
            baseURL:'http://localhost:8001/data/iconfont.json',
            headers: {'Content-type': 'multipart/form-data'}
        });
        newInstance.get('./../../static/data/iconfont.json').then(res=>{
            this.iconOptions=res.data.glyphs
            
        })
    },
    methods: {
        init (id){
            this.dataForm.equipId=id || 0
            this.visible = true
            this.$http({
                    url: this.$http.adornUrl(`/room/equipcategory/getAllEquipCate`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {   
                    if (data && data.code === 0) {
                        this.cateOptions = data.equipcate
                        if (this.dataForm.equipId) {
                            this.$http({
                            url: this.$http.adornUrl(`/room/equip/info/${this.dataForm.equipId}`),
                            method: 'get',
                            params: this.$http.adornParams()
                        }).then(({data}) => {
                            if (data && data.code === 0) {
                                this.dataForm.equipName = data.equip.equipName
                                this.dataForm.iconId = (data.equip.iconId).toString()
                                this.dataForm.equipCateId = data.equip.equipCateId
                            }
                        })
                        }else{
                            this.dataForm.equipName =''
                            this.dataForm.iconId =''
                            this.dataForm.equipCateId = ''
                        }
                    }
                })    
            
        },
        // 表单提交
        dataFormSubmit (){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/room/equip/${!this.dataForm.equipId ? 'save' : 'update'}`),
                        method: 'post',
                        data: this.$http.adornData({
                            'equipId': this.dataForm.equipId || undefined,
                            'equipName': this.dataForm.equipName,
                            'iconId': this.dataForm.iconId,
                            'equipCateId': this.dataForm.equipCateId
                    
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
        lookIcon(value){
          return 'iconfont icon'+value
        },
        
    } 
}
</script>

<style>

</style>