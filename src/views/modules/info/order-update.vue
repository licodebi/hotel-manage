<template>
    <el-dialog
    title="订单"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
        <el-form-item label="订单号" :label-width="formLabelWidth">
            <el-input v-model="dataForm.orderId" style="width:200px" disabled></el-input>
        </el-form-item>
        <el-form-item label="房间名" :label-width="formLabelWidth">
             <el-input v-model="dataForm.room.roomName" suffix-icon="el-icon-date" style="width:200px"></el-input>
        </el-form-item>
        <el-form-item label="预订房间数" :label-width="formLabelWidth" >
             <el-input v-model="dataForm.orderNum" autosize autocomplete="off" w disabled></el-input>
        </el-form-item>
        <el-form-item label="预订人电话" :label-width="formLabelWidth" >
             <el-input v-model="dataForm.orderUserPhone"  autosize autocomplete="off" w></el-input>
        </el-form-item>
        <el-form-item label="预订人邮箱" :label-width="formLabelWidth" >
             <el-input v-model="dataForm.orderUserMail"  autosize autocomplete="off" w></el-input>
        </el-form-item>
        <el-form-item label="用户名" :label-width="formLabelWidth">
             <el-input v-model="dataForm.user.userUsername"  autosize autocomplete="off" w disabled></el-input>
        </el-form-item>
        <el-form-item label="入住人数" :label-width="formLabelWidth">
             <el-input v-model="dataForm.orderUserNumber"  autosize autocomplete="off" w disabled></el-input>
        </el-form-item>
        <el-form-item label="创建时间" :label-width="formLabelWidth">
            <el-date-picker
                v-model="dataForm.orderCreateTime"
                type="datetime"
                placeholder="选择日期时间"
                value-format="yyyy-MM-dd HH:mm:ss">
            </el-date-picker>
        </el-form-item>
        <el-form-item label="入住日期" :label-width="formLabelWidth" >
            <el-date-picker
                v-model="dataForm.orderCheckTime"
                type="date"
                placeholder="选择日期时间"
                value-format="yyyy-MM-dd">
            </el-date-picker>
        </el-form-item>
        <el-form-item label="退房日期" :label-width="formLabelWidth">
            <el-date-picker
                v-model="dataForm.orderEndTime"
                type="date"
                placeholder="选择日期时间"
                value-format="yyyy-MM-dd">
            </el-date-picker>
            
        </el-form-item>
        <el-form-item label="价格" :label-width="formLabelWidth">
            <el-input v-model="dataForm.orderPrice"  autosize autocomplete="off" w></el-input>
        </el-form-item>
        <el-form-item label="订单状态" :label-width="formLabelWidth">
            <el-select v-model="dataForm.orderStatus" placeholder="请选择">
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
                orderId: '',
                room:'',
                user:'',
                orderUserName: '',
                orderUserPhone: '',
                orderUserMail: '',
                orderUserNumber: '',
                orderCreateTime: '',
                orderCheckTime: '',
                orderEndTime: '',
                orderPrice: '',
                orderStatus: '',
                orderRoom:'',
                orderUser:'',
                orderNum:''

            },
             options: [
                 {statusId:1,statusName:'未支付'},
                 {statusId:2,statusName:'已支付'},
                 {statusId:3,statusName:'已取消'},
                 {statusId:4,statusName:'已完成'},
                 {statusId:5,statusName:'已评论'},
             ],
             dataRule: {
                orderId: [
                    { required: true, message: '新闻标题不能为空', trigger: 'blur' }
                ],
                orderUserName: [
                    { required: true, message: '新闻内容不能为空', trigger: 'blur' }
                ]
            }
        }
    },
    methods: {
        init(id){
            //let username =this.$store.state.user.name
            //console.log(username);
            this.dataForm.orderId=id;
            this.visible = true
             if (this.dataForm.orderId) {
                    this.$http({
                    url: this.$http.adornUrl(`/info/order/info/${this.dataForm.orderId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.dataForm.room = data.order.room
                        this.dataForm.user = data.order.user
                        this.dataForm.orderUserName = data.order.orderUserName
                        this.dataForm.orderUserPhone = data.order.orderUserPhone
                        this.dataForm.orderUserMail = data.order.orderUserMail
                        this.dataForm.orderUserNumber = data.order.orderUserNumber
                        this.dataForm.orderCreateTime = data.order.orderCreateTime
                        this.dataForm.orderCheckTime = data.order.orderCheckTime
                        this.dataForm.orderEndTime = data.order.orderEndTime
                        this.dataForm.orderPrice = data.order.orderPrice
                        this.dataForm.orderStatus = data.order.orderStatus
                        this.dataForm.orderUser = data.order.orderUser
                        this.dataForm.orderRoom = data.order.orderRoom
                    }
                })
            }else{
                this.dataForm.orderId= '',
                this.dataForm.room='',
                this.dataForm.user='',
                this.dataForm.orderUserName= '',
                this.dataForm.orderUserPhone= '',
                this.dataForm.orderUserMail= '',
                this.dataForm.orderUserNumber= '',
                this.dataForm.orderCreateTime= '',
                this.dataForm.orderCheckTime= '',
                this.dataForm.orderEndTime= '',
                this.dataForm.orderPrice= '',
                this.dataForm.orderStatus= '',
                this.dataForm.orderRoom='',
                this.dataForm.orderUser='',
                this.dataForm.orderNum=''
            }
        },
        // 表单提交
        dataFormSubmit (){
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.$http({
                        url: this.$http.adornUrl(`/info/order/update`),
                        method: 'post',
                        data: this.$http.adornData({
                            'orderId': this.dataForm.orderId,
                            'orderUserName': this.dataForm.orderUserName,
                            'orderUserPhone': this.dataForm.orderUserPhone,
                            'orderUserNumber' : this.dataForm.orderUserNumber,
                            'orderUserMail' : this.dataForm.orderUserMail,
                            'orderCreateTime' : this.dataForm.orderCreateTime,
                            'orderCheckTime' : this.dataForm.orderCheckTime,
                            'orderEndTime' : this.dataForm.orderEndTime,
                            'orderPrice' : this.dataForm.orderPrice,
                            'orderStatus' : this.dataForm.orderStatus,
                            'orderRoom' : this.dataForm.orderRoom,
                            'orderUser' : this.dataForm.orderUser
                    
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