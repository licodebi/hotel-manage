<template>
    <div>
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
        <el-form-item>
            <el-input v-model="dataForm.orderId" placeholder="订单号" clearable></el-input>
        </el-form-item>
        <el-form-item>
            <el-button @click="getDataList()">查询</el-button>
            <el-button  type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
        </el-form-item>
        </el-form>
        <el-table
        :data="dataList"
        border
        v-loading="dataListLoading"
        @selection-change="selectionChangeHandle"
        style="width: 100%;">
            <el-table-column
                type="selection"
                header-align="center"
                align="center"
                width="50">
            </el-table-column>
            <el-table-column
                prop="orderId"
                header-align="center"
                align="center"
                width="120"
                label="订单ID">
            </el-table-column>
            <el-table-column
                prop="room.roomName"
                header-align="center"
                align="center"
                label="房间名">
            </el-table-column>
            <el-table-column
                prop="user.userUsername"
                header-align="center"
                align="center"
                label="预订账户">
            </el-table-column>
            <el-table-column
                prop="orderUserNumber"
                header-align="center"
                align="center"
                label="人数">
            </el-table-column>
            <el-table-column
                prop="orderCreateTime"
                header-align="center"
                align="center"
                label="订单创建时间">
            </el-table-column>
            <el-table-column
                prop="orderStatus"
                header-align="center"
                align="center"
                label="订单状态">
                <template slot-scope="scope">
                    <span v-if="scope.row.orderStatus==1">未支付</span>
                    <span v-if="scope.row.orderStatus==2">已支付</span>
                    <span v-if="scope.row.orderStatus==3">已取消</span>
                    <span v-if="scope.row.orderStatus==4">已完成</span>
                    <span v-if="scope.row.orderStatus==5">已评论</span>
                </template>
            </el-table-column>
            <el-table-column
                fixed="right"
                header-align="center"
                align="center"
                label="操作">
                <template slot-scope="scope">
                    <el-button  type="text" size="small" @click="orderRegister(scope.row.orderId)">入住登记</el-button>
                    <el-button  type="text" size="small" @click="orderLook(scope.row.orderId)">查看详情</el-button>
                    <el-button  type="text" size="small" @click="orderUpdate(scope.row.orderId)">修改</el-button>
                    <el-button  type="text" size="small" @click="deleteHandle(scope.row.orderId)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination
        @size-change="sizeChangeHandle"
        @current-change="currentChangeHandle"
        :current-page="pageIndex"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize"
        :total="totalPage"
        layout="total, sizes, prev, pager, next, jumper">
        </el-pagination>
        <!-- 弹窗, 新增 / 修改 -->
        <!-- <comment-update v-if="updateVisible" ref="commentUpdate" @refreshDataList="getDataList"></comment-update> -->
        <order-look v-if="lookVisible" ref="orderLook" @refreshDataList="getDataList"></order-look>
        <order-update v-if="updateVisible" ref="orderUpdate" @refreshDataList="getDataList"></order-update>
        <order-register v-if="registerVisible" ref="orderRegister" @refreshDataList="getDataList"></order-register>
    </div>
</template>

<script>

    import orderLook from './order-look'
    import orderUpdate from './order-update'
    import orderRegister from './order-register'

    export default {
        data () {
            return {
                dataForm: {
                    orderId: ''
                },
                dataList: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                dataListLoading: false,
                dataListSelections: [],
                updateVisible: false,
                lookVisible: false,
                registerVisible: false                
            }
        },
        components: {
            orderLook,
            orderUpdate,
            orderRegister
        },
        activated (){
            let userId = this.$route.query.userId;
            if(userId==null){
                this.getDataList();
            }else{
                this.getDataListByUser(userId)
            }
           
        },
        methods: {
            // 获取数据列表
            getDataList () {
                this.dataListLoading = true
                this.$http({
                url: this.$http.adornUrl('/info/order/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'orderId': this.dataForm.orderId
                })
                }).then(({data}) => {
                if (data && data.code === 0) {
                
                    this.dataList = data.page.list
                    this.totalPage = data.page.totalCount
                } else {
                    this.dataList = []
                    this.totalPage = 0
                }
                this.dataListLoading = false
                })
            },
            // 获取数据列表
            getDataListByUser (userId) {
                this.dataListLoading = true
                this.$http({
                url: this.$http.adornUrl('/order/listByUser'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'userId': userId
                })
                }).then(({data}) => {
                if (data && data.code === 0) {
                
                    this.dataList = data.page.list
                    this.totalPage = data.page.totalCount
                } else {
                    this.dataList = []
                    this.totalPage = 0
                }
                this.dataListLoading = false
                })
            },
            // 每页数
            sizeChangeHandle (val) {
                this.pageSize = val
                this.pageIndex = 1
                this.getDataList()
            },
            // 当前页
            currentChangeHandle (val) {
                this.pageIndex = val
                this.getDataList()
            },
            // 多选
            selectionChangeHandle (val) {
                this.dataListSelections = val
            },
            // 修改
            orderUpdate (id) {
                this.updateVisible = true
                this.$nextTick(() => {
                this.$refs.orderUpdate.init(id)
                })
            },
            //查看订单详情
            orderLook (id) {
                this.lookVisible = true
                this.$nextTick(() => {
                this.$refs.orderLook.init(id)
                })
            },
            // 删除
            deleteHandle (id) {
                var orderIds = id ? [id] : this.dataListSelections.map(item => {
                    return item.orderId
                })
                this.$confirm(`确定对[订单=${orderIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http({
                    url: this.$http.adornUrl('/info/order/delete'),
                    method: 'post',
                    data: this.$http.adornData(orderIds, false)
                    }).then(({data}) => {
                    if (data && data.code === 0) {
                        this.$message({
                        message: '操作成功',
                        type: 'success',
                        duration: 1500,
                        onClose: () => {
                            this.getDataList()
                        }
                        })
                    } else {
                        this.$message.error(data.msg)
                    }
                    })
                }).catch(() => {})
            },
            //入住登记
            orderRegister (id) {
                this.registerVisible = true
                this.$nextTick(() => {
                this.$refs.orderRegister.init(id)
                })
            },
        }
    }
</script>

<style>

</style>