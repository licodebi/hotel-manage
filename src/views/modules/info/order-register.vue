<template>
    <el-dialog
    title="入住登记"
    :close-on-click-modal="false"
    :visible.sync="visible">
 
    <div  style="margin-left:15px;margin-bottom:10px"><span>订单号:  </span><span>{{dataForm.orderId}}</span></div>
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
        
        <el-form-item label="入住人数" :label-width="formLabelWidth">
              <el-input-number v-model="dataForm.orderUserNumber" @change="handleUserNumChange" :min="dataForm.minNum" :max="dataForm.maxNum" label="入住人数"></el-input-number>
        </el-form-item>
        <div class="diaStyle" v-for="(orderUser,index) in dataForm.orderUserList " :key="orderUser.index">
            <el-form-item label="入住人" :prop="'orderUserList.'+index+'.userName'" :rules="dataRule.userName">
                <label slot="label">入住人{{index+1}}</label>
                <el-input
                placeholder="请输入姓名"
                suffix-icon="el-icon-user"
                v-model="orderUser.userName"
                style="width:200px">
                </el-input>
            </el-form-item>
            <el-form-item label="入住人" :prop="'orderUserList.'+index+'.userCard'" :rules="dataRule.userCard">
                <label slot="label">身份证</label>
                <el-input
                placeholder="请输入身份证"
                suffix-icon="el-icon-user"
                v-model="orderUser.userCard"
                style="width:200px">
                </el-input>
            </el-form-item>
        </div>
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
        var validOrderUser=(rule, value,callback)=>{
            const reg = /^[\u4e00-\u9fa5]+$/;    
            if (!reg.test(value)){
                callback(new Error('请输入中文姓名'))
            }else {
                callback()
            }
        };
        var validOrderCard=(rule, value,callback)=>{
            const reg = /^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/;    
            if (!reg.test(value)){
                callback(new Error('请正确的身份证'))
            }else {
                callback()
            }
        };
        return {
            formLabelWidth: '80px',
            visible: false,
            
            dataForm: {
                orderId: '',
                orderUserNumber: '',
                orderUserIdcard:'',
                orderUserName:'',
                minNum:0,
                maxNum:0,
                orderUserList:[],

            },
            options: [
                 
            ],
            dataRule: {
                orderId: [
                    { required: true, message: '新闻标题不能为空', trigger: 'blur' }
                ],
                orderUserName: [
                    { required: true, message: '新闻内容不能为空', trigger: 'blur' }
                ],
                userName: [
                    {required: true, message: '请填写入住人', trigger: 'blur'},
                    {validator:validOrderUser,trigger: 'blur' }
                ],
                userCard: [
                    {required: true, message: '请填写入住人', trigger: 'blur'},
                    {validator:validOrderCard,trigger: 'blur' }
                ]
            }
        }
    },
    methods: {
        init(id){
            this.dataForm.orderId=id;
            this.visible = true
             if (this.dataForm.orderId) {
                    this.$http({
                    url: this.$http.adornUrl(`/info/order/info/${this.dataForm.orderId}`),
                    method: 'get',
                    params: this.$http.adornParams()
                }).then(({data}) => {
                    if (data && data.code === 0) {
                       
                        this.dataForm.orderUserName = data.order.orderUserName
                        this.dataForm.orderUserNumber = data.order.orderUserNumber
                        this.dataForm.minNum=data.order.orderNum
                        this.dataForm.maxNum=data.order.orderNum*data.order.room.roomType
                        this.orderUserIdcard=data.order.orderUserIdcard
                        this.dataForm.orderUserList=[]
                        if(data.order.orderUserIdcard!=null&&data.order.orderUserName!=null){
                            let userNameList=data.order.orderUserName.split(',')
                            let userCardList=data.order.orderUserIdcard.split(',')
                            for (let key=0;key<data.order.orderUserNumber;key++) {
                                this.dataForm.orderUserList.push({userName:userNameList[key],userCard:userCardList[key]})
                            }
                        }else{
                            let userList=[]
                            for (let key=0;key<data.order.orderUserNumber;key++) {
                                userList.push({userName:'',userCard:''})
                            }
                            this.dataForm.orderUserList=userList
                        }
                        
                        
                    }
                })
            }else{
                this.dataForm.orderUserName=''
                this.dataForm.orderUserIdcard=''
                this.dataForm.orderUserNumber=''
                this.dataForm.orderId=''
                this.dataForm.orderUserList=[]
                this.dataForm.minNum=0
                this.dataForm.maxNum=0

            }
        },
        // 表单提交
        dataFormSubmit (){
            
            this.$refs['dataForm'].validate((valid) => {
                if (valid) {
                    this.getNameAndCard(this.dataForm.orderUserList)
                    this.$http({
                        url: this.$http.adornUrl(`/info/order/register`),
                        method: 'post',
                        data: this.$http.adornData({
                            'orderId': this.dataForm.orderId,
                            'orderUserName': this.dataForm.orderUserName,
                            'orderUserIdcard': this.dataForm.orderUserIdcard,
                            'orderUserNumber': this.dataForm.orderUserNumber
                    
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
        //将姓名和身份证数组转为字符串
        getNameAndCard(orderUserList){
            let nameList=[]
            let cardList=[]
            for (let key=0;key<orderUserList.length;key++) {
                nameList.push(orderUserList[key].userName)
                cardList.push(orderUserList[key].userCard)
            }
            this.dataForm.orderUserName=nameList.toString()
            this.dataForm.orderUserIdcard=cardList.toString()

        },
        handleUserNumChange(value){
            let num =0
            if(value>this.dataForm.orderUserList.length){
                num=value-this.dataForm.orderUserList.length
                for (let key=0;key<num;key++) {
                    this.dataForm.orderUserList.push({userName:'',userCard:''})
                }
            }else{
                num=this.dataForm.orderUserList.length-value
                for (let key=0;key<num;key++) {
                    this.dataForm.orderUserList.pop()
                }
            }
        }
    }
}
</script>

<style>

</style>