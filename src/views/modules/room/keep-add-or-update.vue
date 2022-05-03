<template>
  <el-dialog
    :title="!dataForm.roomId ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="房间名" prop="roomName">
        <el-input v-model="dataForm.roomName" placeholder="房间名" style="width: 150px"></el-input>
      </el-form-item>
      <el-form-item label="房间价格" prop="roomPrice">
        <el-input v-model="dataForm.roomPrice" placeholder="房间价格"  style="width: 100px"></el-input>
      </el-form-item>
      <el-form-item label="房间状态" prop="roomStatus">
          <el-select v-model="dataForm.roomStatus" placeholder="请选择">
            <el-option
            v-for="item in options2"
            :key="item.statusId"
            :label="item.statusName"
            :value="item.statusId">
            </el-option>
          </el-select>
      </el-form-item>
      <!-- <el-form-item label="房间分类" prop="roomCategoryId">
        <el-select v-model="dataForm.roomCategoryId" placeholder="请选择">
          <el-option
          v-for="item in options"
          :key="item.cateId"
          :label="item.cateName"
          :value="item.cateId">
          </el-option>
        </el-select>
      </el-form-item> -->
      <el-form-item label="房间数" prop="roomStock">
        <el-input v-model="dataForm.roomStock" placeholder="房间数" style="width: 100px"></el-input>
      </el-form-item>
      <el-form-item label="预订数" prop="roomSurplus">
        <el-input v-model="dataForm.roomSurplus" placeholder="预订数" style="width: 100px" ></el-input>
      </el-form-item>
      <el-form-item label="房间信息" prop="roomInformation">
        <el-input v-model="dataForm.roomInformation" placeholder="房间信息" maxlength="100" show-word-limit type="textarea" :autosize="{ minRows: 2, maxRows: 4}"></el-input>
      </el-form-item>
      <el-form-item label="房间设备" >
        <el-checkbox-group v-model="checkedEquip">
          <el-checkbox v-for="equip in equips" :label="equip.equipName" :key="equip.equipId">{{equip.equipName}}</el-checkbox>
        </el-checkbox-group>
      </el-form-item>
      <el-form-item label="房间评分" prop="roomGrade">
        <el-input v-model="dataForm.roomGrade" placeholder="房间评分" style="width: 100px" ></el-input>
      </el-form-item>
      <el-form-item label="收藏数" prop="roomLike">
        <el-input v-model="dataForm.roomLike" placeholder="收藏数" style="width: 100px" ></el-input>
      </el-form-item>
      <el-form-item label="总销量" prop="roomSales">
        <el-input v-model="dataForm.roomSales" placeholder="总销量" style="width: 100px" ></el-input>
      </el-form-item>
      <el-form-item label="房间类型" prop="roomType">
        <el-input v-model="dataForm.roomType" placeholder="房间类型" style="width: 100px" ></el-input>人间
      </el-form-item>
      <!-- 图片上传 -->
      <!-- <el-form-item label="添加图片" ref="roomPhoto" prop="img">
        <el-input v-model="dataForm.roomPhoto" v-if="false"></el-input>
        <el-upload
        class="avatar-uploader"
        :action="materialPictureAndText()"
        :on-success="handleAvatarSuccess"
        :show-file-list="false"
        :before-upload="beforeUpload2">
            <img style="width: 70px;height: 70px" v-if="imgUrl" :src="imgUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
        </el-upload>
      </el-form-item> -->
      <el-form-item label="添加图片" ref="roomPhoto" prop="img">
        <el-input v-model="dataForm.roomPhoto" v-if="false"></el-input>
        <el-upload
        list-type="picture-card"
        class="avatar-uploader"
        :on-remove="handleRemove"
        :on-success="handleAvatarSuccess"
        :action="materialPictureAndText()"
        :before-upload="beforeUpload2"
        :file-list="fileList"
        >
          <i slot="default" class="el-icon-plus"></i>
          <div slot="fileList" slot-scope="{fileList}">
            <img
              class="el-upload-list__item-thumbnail"
              :src="this.$http.adornUrl(fileList.url)" alt=""
            >
            <span class="el-upload-list__item-actions">
              <span
                v-if="!disabled"
                class="el-upload-list__item-delete"
                @click="handleRemove(file)"
              >
                <i class="el-icon-delete"></i>
              </span>
            </span>
          </div>
        </el-upload>
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
        var validStock=(rule, value,callback)=>{
            const reg = /(^[1-9]\d*$)/;    
            if (!reg.test(value)){
                callback(new Error('请输入正整数'))
                if(value<this.roomSurplus){
                  callback(new Error('库存数不可小于已预订数'))
                }
            }else {
                callback()
            }
        }
        var validplus=(rule, value,callback)=>{
            const reg = /(^[0-9]\d*$)/;    
            if (!reg.test(value)){
                callback(new Error('请输入正整数'))
                if(value<this.roomSurplus){
                  callback(new Error('库存数不可小于已预订数'))
                }
            }else {
                callback()
            }
        }
        return{
            visible: false,
            dataForm: {
                roomId: 0,
                roomName: '',
                roomPrice:'',
                roomStatus:'',
                roomStock:'',
                roomInformation:'',
                roomGrade:'',
                roomType:'',
                roomPhoto:'',
                roomLike:'',
                roomSales:'',
                roomSurplus:'',
            },
            dataRule: {
                roomName: [
                    { required: true, message: '房间名不能为空', trigger: 'blur' }  
                ],
                roomPrice:[{
                    required: true, message: '房间价格不能为空', trigger: 'blur'}
                ],
                roomStatus:[{
                    required: true, message: '请选择房间状态', trigger: 'blur'
                }],
                
                roomStock:[{
                    required: true, message: '请输入房间数', trigger: 'blur'},                    
                    {validator:validStock,trigger: 'blur'}
                ],
                roomInformation:[{
                    required: true, message: '请输入房间信息', trigger: 'blur'
                }],
                roomSurplus:[{
                    required: true, message: '请输入房间数', trigger: 'blur'},                    
                    {validator:validplus,trigger: 'blur'}
                ]


            },
            options2: [
              {statusId:1,statusName:'未满'},
              {statusId:2,statusName:'已满'}
            ],  
            checkedEquip: [],
            equips: [],
            disabled: false,
            fileList: [],
            picList: []
        }
    },
    methods: {
        init (id){
            this.fileList= []
            this.picList= []
            this.dataForm.roomId=id || 0
            this.visible = true
            if (this.dataForm.roomId) {
                this.$http({
                    url: this.$http.adornUrl(`/room/keep/info/${this.dataForm.roomId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataForm.roomName = data.room.roomName,
                        this.dataForm.roomPrice = data.room.roomPrice,
                        this.dataForm.roomStatus = data.room.roomStatus,
                        this.dataForm.roomStock = data.room.roomStock,
                        this.dataForm.roomSurplus = data.room.roomSurplus,
                        this.dataForm.roomInformation = data.room.roomInformation,
                        this.dataForm.roomGrade = data.room.roomGrade,
                        this.dataForm.roomType = data.room.roomType,
                        this.dataForm.roomLike = data.room.roomLike,
                        this.dataForm.roomSales = data.room.roomSales,
                        this.dataForm.roomPhoto = data.room.roomPhoto,
                        this.imgUrl=this.$http.adornUrl(data.room.roomPhoto)
                        this.checkedEquip=[]
                        this.equips = data.allequip
                        for (let i in data.equip) {
                          this.checkedEquip.push(data.equip[i].equipName)
                        }
                        let photo=data.room.roomPhoto.split(',')
                        for (let key in photo) {
                          this.fileList.push({url:this.$http.adornUrl(photo[key])})
                          this.picList.push(photo[key])
                        }
                  }
                })
             }
             else{
                this.$http({
                    url: this.$http.adornUrl(`/room/keep/getAllEquip`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {

                  this.checkedEquip=[]
                  this.equips = data.allequip
                  this.dataForm.roomName = ''
                  this.dataForm.roomSurplus = ''
                  this.dataForm.roomPrice = ''
                  this.dataForm.roomStatus = ''
                  this.dataForm.roomStock = ''
                  this.dataForm.roomInformation = ''
                  this.dataForm.roomGrade = ''
                  this.dataForm.roomType = ''
                  this.dataForm.roomLike = ''
                  this.dataForm.roomSales = ''
                  this.dataForm.roomPhoto = ''
                  this.picList=[]
                  this.fileList=[]

                })
               
             }
            
        },
        // 表单提交
        dataFormSubmit (){
            this.dataForm.roomPhoto=this.picList.toString();
            console.log(this.dataForm.roomPhoto);
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/room/keep/${!this.dataForm.roomId ? 'save' : 'update'}`),
                        method: 'post',
                        data: this.$http.adornData({
                            'roomId': this.dataForm.roomId || undefined,
                            'roomName': this.dataForm.roomName,
                            'roomPrice': this.dataForm.roomPrice,
                            'roomStatus': this.dataForm.roomStatus,
                            'roomCategoryId': this.dataForm.roomCategoryId,
                            'roomStock': this.dataForm.roomStock,
                            'roomInformation': this.dataForm.roomInformation,
                            'roomGrade': this.dataForm.roomGrade,
                            'roomType': this.dataForm.roomType,
                            'roomPhoto': this.dataForm.roomPhoto,
                            'roomLike': this.dataForm.roomLike,
                            'roomSales': this.dataForm.roomSales,
                            'roomSurplus': this.dataForm.roomSurplus,
                            'checkedEquip': this.checkedEquip
                    
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
        handleAvatarSuccess (response,file,fileList) {
      
            if(response.code==0){
                //this.imgUrl=this.$http.adornUrl(response.success)
                //this.dataForm.roomPhoto=response.success
                this.picList.push(response.success)
                this.fileList.push({url:this.$http.adornUrl(response.success)})
                console.log(this.fileList);
                console.log(this.picList);
            }else{
                this.$message({
                 type: 'info',
                message: '请重新上传'
            })  
            }
           
        },
        //照片上传
        materialPictureAndText () {
            return this.$http.adornUrl(`/pictureManage/materialPicture?token=${this.$cookie.get('token')}`)  //前面是为了方便线上加的
        },
        //上传前的校验
        beforeUpload2 (file) {
            const isLt2M = file.size < 1024 * 1024 * 2
            // console.log('大小' + isLt2M)
            if (!isLt2M) {
            this.$message.error('上传文件大小不能超过 2MB!')
            }
            
            return isLt2M
        },
        handleRemove(file) {
          console.log(file);
          for (let i in this.fileList) {
            if(file.url==this.fileList[i].url){
              this.fileList.splice(i,1)
              this.picList.splice(i,1)
            }
          }
          let path = file.url
          let str = path.split("/upload/");
          this.$http({
            url: this.$http.adornUrl(`/pictureManage/deletePicture?token=${this.$cookie.get('token')}`),
            method: 'post',
            params: this.$http.adornParams({
              "filePath":str[1]
              
            })
            
            }).then(({data}) => {
              if (data && data.code === 0){
              }
            })
        }
    } 
}
</script>

<style>

</style>