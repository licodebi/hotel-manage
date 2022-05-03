<template>
    <div class="mod-role">
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
          <el-form-item>
              <el-input v-model="dataForm.roomName" placeholder="房间名" clearable></el-input>
          </el-form-item>
          <el-form-item>
              <el-button @click="getDataList()">查询</el-button>
              <el-button  type="primary" v-if="isAuth('room:keep:save')" @click="addOrUpdateHandle()">新增</el-button>
              <el-button  type="danger" v-if="isAuth('room:keep:delete')" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
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
              prop="roomId"
              header-align="center"
              align="center"
              label="房间id">
          </el-table-column>
          <el-table-column
              prop="roomName"
              header-align="center"
              align="center"
              label="房间名">
          </el-table-column>
          <el-table-column
              prop="roomPrice"
              header-align="center"
              align="center"
              label="房间价格">
          </el-table-column>
          <el-table-column
              prop="roomStock"
              header-align="center"
              align="center"
              label="房间总数">
          </el-table-column>
          <el-table-column
              prop="roomSurplus"
              header-align="center"
              align="center"
              label="预订数">
          </el-table-column>
          <el-table-column
              header-align="center"
              align="center"
              label="房间照片">
              <template slot-scope="scope">
                <img :src="scope.row.roomPhoto" style="width: 70px;height: 70px">
              </template>
          </el-table-column>
          <el-table-column
              prop="roomStatus"
              header-align="center"
              align="center"
              label="房间状态">
              <template slot-scope="scope">
                    <span v-if="scope.row.roomStatus==1">未满</span>
                    <span v-if="scope.row.roomStatus==2">已满</span>
                </template>
          </el-table-column>
          
          <el-table-column
              prop="roomInformation"
              header-align="center"
              align="center"
              label="房间信息">
              <template slot-scope="scope">
                    <span>{{scope.row.roomInformation.slice(0,15)+"......"}}</span> 
              </template>
          </el-table-column>
          <el-table-column
              prop="roomGrade"
              header-align="center"
              align="center"
              label="房间评分">
          </el-table-column>
          <el-table-column
              header-align="center"
              align="center"
              label="操作">
              <template slot-scope="scope">
              <el-button  type="text" size="small" v-if="isAuth('room:keep:update')" @click="addOrUpdateHandle(scope.row.roomId)">修改</el-button>
              <el-button  type="text" size="small" v-if="isAuth('room:keep:delete')" @click="deleteHandle(scope.row.roomId,scope.row.roomName)">删除</el-button>
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
        <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    </div>
</template>

<script>
import AddOrUpdate from './keep-add-or-update'
export default {
    data () {
      return {
        dataForm: {
          roomName: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
        AddOrUpdate
    },
    activated (){
        this.getDataList();
    },
    methods: {
        // 获取数据列表
        getDataList () {
            this.dataListLoading = true
            this.$http({
            url: this.$http.adornUrl('/room/keep/list'),
            method: 'get',
            params: this.$http.adornParams({
                'page': this.pageIndex,
                'limit': this.pageSize,
                'roomName': this.dataForm.roomName
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
          var roomIds = id ? [id] : this.dataListSelections.map(item => {
            return item.roomId
          })
          var roomNames = name ? [name] : this.dataListSelections.map(item => {
              return item.roomName
          }) 
          this.$confirm(`确定对[房间=${roomNames.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$http({
              url: this.$http.adornUrl('/room/keep/delete'),
              method: 'post',
              data: this.$http.adornData(roomIds, false)
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
        addOrUpdateHandle (id) {
          this.addOrUpdateVisible = true
          this.$nextTick(() => {
            this.$refs.addOrUpdate.init(id)
          })
        }
    }
}
</script>

<style>
  .el-table th.gutter{
    display: table-cell!important;
  }
</style>