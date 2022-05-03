<template>
    <div>
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
        <el-date-picker
            v-model="dataForm.mesCreateTime"
            type="daterange"
            align="right"
            unlink-panels
            range-separator="-"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            :picker-options="pickerOptions"
            value-format="yyyy-MM-dd HH:mm:ss">
        </el-date-picker>
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
                prop="mesId"
                header-align="center"
                align="center"
                width="120"
                label="留言ID">
            </el-table-column>
            <el-table-column
                prop="mesContent"
                header-align="center"
                align="center"
                label="留言内容">
            </el-table-column>
            <el-table-column
                prop="user.userUsername"
                header-align="center"
                align="center"
                label="留言人">
            </el-table-column>
            <el-table-column
                prop="mesCreateTime"
                header-align="center"
                align="center"
            
                label="留言时间">
            </el-table-column>
            <el-table-column
                header-align="center"
                align="center"
                label="留言状态">
                <template slot-scope="scope">
                {{scope.row.mesStatus==1?'未回复':'已回复'}}
                </template>
            </el-table-column>
            <el-table-column
                fixed="right"
                header-align="center"
                align="center"
                width="130"
                label="操作">
                <template slot-scope="scope">
                <el-button  type="text" size="small" @click="lookMessage(scope.row.mesId)">查看</el-button>
                <el-button  type="text" size="small" @click="messageUpdate(scope.row.mesId)">回复</el-button>
                <el-button  type="text" size="small" @click="deleteHandle(scope.row.mesId)">删除</el-button>
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
        <message-update v-if="updateVisible" ref="messageUpdate" @refreshDataList="getDataList"></message-update>
        <message-look v-if="lookVisible" ref="messageLook"></message-look>
    </div>
</template>

<script>
    import messageLook from './message-look'
    import messageUpdate from './message-update'
    export default {
        data () {
            return {
                dataForm: {
                    mesCreateTime: ''
                },
                dataList: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                dataListLoading: false,
                dataListSelections: [],
                updateVisible: false,
                lookVisible: false,
                pickerOptions: {
                    disabledDate(time) {
                        return time.getTime() > Date.now();
                    },
                    shortcuts: [
                        {
                        text: '最近一周',
                        onClick(picker) {
                            const end = new Date();
                            const start = new Date();
                            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
                            picker.$emit('pick', [start, end]);
                            }
                        }, 
                        {
                        text: '最近一个月',
                        onClick(picker) {
                            const end = new Date();
                            const start = new Date();
                            start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
                            picker.$emit('pick', [start, end]);
                            }
                        }, 
                        {
                        text: '最近三个月',
                        onClick(picker) {
                            const end = new Date();
                            const start = new Date();
                            start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
                            picker.$emit('pick', [start, end]);
                            }
                        }
                    ]
                }
                
            }
        },
        components: {
            messageLook,
            messageUpdate
        },
        activated (){
            this.getDataList();
        },
        methods: {
            // 获取数据列表
            getDataList () {
                this.dataListLoading = true
                this.$http({
                url: this.$http.adornUrl('/info/message/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'mesCreateTime': this.dataForm.mesCreateTime[0],
                    'mesEndTime': this.dataForm.mesCreateTime[1]
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
            // 回复
            messageUpdate (id) {
                this.updateVisible = true
                this.$nextTick(() => {
                this.$refs.messageUpdate.init(id)
                })
            },
            //查看
            lookMessage (id) {
                this.lookVisible=true;
                this.$nextTick(() => {
                this.$refs.messageLook.init(id)
                })
            },
            //删除
            // 删除
            deleteHandle (id) {
                var mesIds = id ? [id] : this.dataListSelections.map(item => {
                    return item.mesId
                })
                this.$confirm(`确定对[留言=${mesIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http({
                    url: this.$http.adornUrl('/info/message/delete'),
                    method: 'post',
                    data: this.$http.adornData(mesIds, false)
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
        }
    }
</script>

<style>

</style>