<template>
    <div class="mod-role">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.condition" placeholder="用户账号或用户电话" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
                <el-button v-if="isAuth('info:user:delete')"  type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
            </el-form-item>
        </el-form>
        <el-table
        :data="dataList"
        border
        v-loading="dataListLoading"
        @selection-change="selectionChangeHandle">
            <el-table-column
                type="selection"
                header-align="center"
                align="center">
            </el-table-column>
            <el-table-column
                prop="userId"
                header-align="center"
                align="center"
                label="Id"
                width="40px">
            </el-table-column>
            <el-table-column
                prop="userUsername"
                header-align="center"
                align="center"
                label="用户账号">
                <template slot-scope="scope">
                    <el-button type="text" @click="findOrder(scope.row.userId)">{{scope.row.userUsername}}</el-button>
                </template>
            </el-table-column>
            <el-table-column
                prop="userPassword"
                header-align="center"
                align="center"
                label="用户密码">
            </el-table-column>
            <el-table-column
                prop="userName"
                header-align="center"
                align="center"
                label="用户姓名">
            </el-table-column>
         
            <el-table-column
                prop="userPhone"
                header-align="center"
                align="center"
                label="用户电话">
            </el-table-column>
               <el-table-column
                prop="userEmail"
                header-align="center"
                align="center"
                label="用户邮箱">
            </el-table-column>
            <el-table-column
                prop="userAddress"
                header-align="center"
                align="center"
                label="用户地址">
            </el-table-column>
            
            <el-table-column
                header-align="center"
                align="center"
                label="房间照片">
                <template slot-scope="scope">
                    <img :src="scope.row.userPhoto" style="width: 70px;height: 70px">
                </template>
            </el-table-column>
            <el-table-column
                prop="userStatus"
                header-align="center"
                align="center"
                label="用户状态">
                <template slot-scope="scope">
                    <span v-if="scope.row.userStatus==1">正常使用</span>
                    <span v-if="scope.row.userStatus==2">已禁用</span>
                </template>
            </el-table-column>
            <el-table-column
                header-align="center"
                align="center"
                label="操作">
                <template slot-scope="scope">
                <el-button  v-if="isAuth('info:user:update')"  type="text" size="small" @click="userUpdateHandle(scope.row.userId)">修改</el-button>
                <el-button  v-if="isAuth('info:user:delete')"  type="text" size="small" @click="deleteHandle(scope.row.userId,scope.row.userUsername)">删除</el-button>
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
        <user-update v-if="updateVisible" ref="userUpdate" @refreshDataList="getDataList"></user-update>
    </div>
</template>

<script>
import userUpdate from './user-update'
export default {
    data () {
      return {
        dataForm: {
          condition: ''
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
        userUpdate
    },
    activated (){
        this.getDataList();
    },
    methods: {
        // 获取数据列表
        getDataList () {
            this.dataListLoading = true
            this.$http({
            url: this.$http.adornUrl('/info/user/list'),
            method: 'get',
            params: this.$http.adornParams({
                'page': this.pageIndex,
                'limit': this.pageSize,
                'condition': this.dataForm.condition
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
        // 删除
        deleteHandle (id,name) {
          var userIds = id ? [id] : this.dataListSelections.map(item => {
            return item.userId
          })
          var userUsernames = name ? [name] : this.dataListSelections.map(item => {
              return item.userUsername
          }) 
          this.$confirm(`确定对[房间=${userUsernames.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$http({
              url: this.$http.adornUrl('/info/user/delete'),
              method: 'post',
              data: this.$http.adornData(userIds, false)
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
      // 新增 / 修改
        userUpdateHandle (id) {
          this.updateVisible = true
          this.$nextTick(() => {
            this.$refs.userUpdate.init(id)
          })
        },
        findOrder(userId){
            this.$router.push({path:"/info-order",query:{userId:userId}})
        }
    }
}
</script>

<style>
</style>