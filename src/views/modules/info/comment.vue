<template>
    <div>
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
        <el-form-item>
            <el-input v-model="dataForm.comRoom" placeholder="房间名称" clearable></el-input>
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
                prop="comId"
                header-align="center"
                align="center"
                width="120"
                label="评论ID">
            </el-table-column>
            <el-table-column
                prop="user.userUsername"
                header-align="center"
                align="center"
                label="评论人">
            </el-table-column>
            <el-table-column
                prop="room.roomName"
                header-align="center"
                align="center"
                label="评论房间">
            </el-table-column>
            <el-table-column
                prop="comContent"
                header-align="center"
                align="center"
                label="评论内容">
            </el-table-column>
            <el-table-column
                prop="comCreateTime"
                header-align="center"
                align="center"
                label="评论时间">
            </el-table-column>
            <el-table-column
                prop="comGrade"
                header-align="center"
                align="center"
                label="用户评分">
            </el-table-column>
            
            <el-table-column
                fixed="right"
                header-align="center"
                align="center"
                width="130"
                label="操作">
                <template slot-scope="scope">
                <el-button  type="text" size="small" @click="messageUpdate(scope.row.comId)">回复</el-button>
                <el-button  type="text" size="small" @click="deleteHandle(scope.row.comId)">删除</el-button>
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
        <comment-update v-if="updateVisible" ref="commentUpdate" @refreshDataList="getDataList"></comment-update>
    </div>
</template>

<script>

    import commentUpdate from './comment-update'
    export default {
        data () {
            return {
                dataForm: {
                    comRoom: ''
                },
                dataList: [],
                pageIndex: 1,
                pageSize: 10,
                totalPage: 0,
                dataListLoading: false,
                dataListSelections: [],
                updateVisible: false                
            }
        },
        components: {
            commentUpdate
        },
        activated (){
            this.getDataList();
        },
        methods: {
            // 获取数据列表
            getDataList () {
                this.dataListLoading = true
                this.$http({
                url: this.$http.adornUrl('/info/comment/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'comRoom': this.dataForm.comRoom
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
            messageUpdate (id) {
                this.updateVisible = true
                this.$nextTick(() => {
                this.$refs.commentUpdate.init(id)
                })
            },
            // 删除
            deleteHandle (id) {
                var comIds = id ? [id] : this.dataListSelections.map(item => {
                    return item.comId
                })
                this.$confirm(`确定对[评论=${comIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    this.$http({
                    url: this.$http.adornUrl('/info/comment/delete'),
                    method: 'post',
                    data: this.$http.adornData(comIds, false)
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