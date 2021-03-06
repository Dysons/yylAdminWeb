<template>
  <div class="app-container">
    <!-- 日志查询 -->
    <div class="filter-container">
      <el-select v-model="logQuery.type" class="filter-item" placeholder="日志类型" style="width:110px;" clearable>
        <el-option key="1" label="登录日志" value="1" />
        <el-option key="2" label="操作日志" value="2" />
      </el-select>
      <el-input v-model="logQuery.user_keyword" class="filter-item" style="width: 135px;" placeholder="用户账号/昵称" clearable />
      <el-input v-model="logQuery.request_keyword" class="filter-item" style="width: 155px;" placeholder="请求IP/地区/ISP" clearable />
      <el-input v-model="logQuery.menu_keyword" class="filter-item" style="width: 280px;" placeholder="菜单链接/名称" clearable />
      <el-date-picker v-model="logQuery.create_time" type="daterange" style="width: 240px;top: -4px;" range-separator="-" start-placeholder="开始日期" end-placeholder="结束日期" value-format="yyyy-MM-dd" />
      <el-button class="filter-item" type="primary" @click="logSearch">查询</el-button>
      <el-button class="filter-item" type="primary" style="float:right;" @click="logRefresh">刷新</el-button>
    </div>
    <!-- 日志列表 -->
    <el-table v-loading="loading" :data="logData" :height="height" style="width: 100%" border @sort-change="logSort">
      <el-table-column prop="admin_log_id" label="ID" min-width="100" sortable="custom" fixed="left" />
      <el-table-column prop="username" label="用户账号" min-width="110" />
      <el-table-column prop="nickname" label="用户昵称" min-width="110" />
      <el-table-column prop="menu_url" label="菜单链接" min-width="235" />
      <el-table-column prop="menu_name" label="菜单名称" min-width="130" />
      <el-table-column prop="request_method" label="请求方式 " min-width="110" sortable="custom" />
      <el-table-column prop="request_ip" label="请求IP" min-width="130" sortable="custom" />
      <el-table-column prop="request_region" label="请求地区" min-width="150" />
      <el-table-column prop="request_isp" label="请求ISP" min-width="110" />
      <el-table-column prop="create_time" label="请求时间" min-width="160" sortable="custom" />
      <el-table-column label="操作" min-width="150" align="right" fixed="right" class-name="small-padding fixed-width">
        <template slot-scope="{ row }">
          <el-button size="mini" type="primary" @click="logDetail(row)">详情</el-button>
          <el-button size="mini" type="danger" @click="logDelete(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 日志分页 -->
    <pagination v-show="logCount > 0" :total="logCount" :page.sync="logQuery.page" :limit.sync="logQuery.limit" @pagination="logLists" />
    <!-- 日志详情 -->
    <el-dialog :title="'日志信息：' + logModel.admin_log_id" :visible.sync="logDialog" width="65%" top="1vh" :before-close="logCancel">
      <el-form ref="formRef" :rules="logRules" :model="logModel" label-width="100px" class="dialog-body" :style="{height:height+60+'px'}">
        <el-form-item label="用户ID" prop="admin_user_id">
          <el-input v-model="logModel.admin_user_id" />
        </el-form-item>
        <el-form-item label="用户账号" prop="username">
          <el-input v-model="logModel.username" />
        </el-form-item>
        <el-form-item label="用户昵称" prop="nickname">
          <el-input v-model="logModel.nickname" />
        </el-form-item>
        <el-form-item label="菜单ID" prop="admin_menu_id">
          <el-input v-model="logModel.admin_menu_id" />
        </el-form-item>
        <el-form-item label="菜单链接" prop="menu_url">
          <el-input v-model="logModel.menu_url" />
        </el-form-item>
        <el-form-item label="菜单名称" prop="menu_name">
          <el-input v-model="logModel.menu_name" />
        </el-form-item>
        <el-form-item label="请求方式" prop="request_method">
          <el-input v-model="logModel.request_method" />
        </el-form-item>
        <el-form-item label="请求IP" prop="request_ip">
          <el-input v-model="logModel.request_ip" />
        </el-form-item>
        <el-form-item label="请求地区" prop="request_region">
          <el-input v-model="logModel.request_region" />
        </el-form-item>
        <el-form-item label="请求ISP" prop="request_isp">
          <el-input v-model="logModel.request_isp" />
        </el-form-item>
        <el-form-item label="请求时间" prop="create_time">
          <el-input v-model="logModel.create_time" />
        </el-form-item>
        <el-form-item label="请求参数" prop="request_param">
          <pre>{{ logModel.request_param }}</pre>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="logCancel">取消</el-button>
        <el-button type="primary" @click="logSubmit">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import screenHeight from '@/utils/screen-height'
import Pagination from '@/components/Pagination'
import { logList, logInfo, logDele } from '@/api/admin'

export default {
  name: 'Log',
  components: { Pagination },
  data() {
    return {
      height: 680,
      loading: false,
      logData: [],
      logCount: 0,
      logQuery: {
        page: 1,
        limit: 13,
        type: ''
      },
      logDialog: false,
      logModel: {},
      logRules: {}
    }
  },
  created() {
    this.height = screenHeight()
    this.logLists()
  },
  methods: {
    // 日志列表
    logLists() {
      this.loading = true
      logList(this.logQuery)
        .then(res => {
          this.logData = res.data.list
          this.logCount = res.data.count
          this.loading = false
        })
        .catch(() => {
          this.loading = false
        })
    },
    // 日志排序
    logSort(sort) {
      this.logQuery.sort_field = sort.prop
      this.logQuery.sort_type = ''
      if (sort.order === 'ascending') {
        this.logQuery.sort_type = 'asc'
        this.logLists()
      }
      if (sort.order === 'descending') {
        this.logQuery.sort_type = 'desc'
        this.logLists()
      }
    },
    // 日志查询
    logSearch() {
      this.logQuery.page = 1
      this.logLists()
    },
    // 日志刷新
    logRefresh() {
      this.logQuery = this.$options.data().logQuery
      this.logLists()
    },
    // 日志详情
    logDetail(row) {
      this.loading = true
      logInfo({ admin_log_id: row.admin_log_id })
        .then(res => {
          this.logReset(res.data)
          this.logDialog = true
          this.loading = false
        })
        .catch(() => {
          this.loading = false
        })
    },
    // 日志删除
    logDelete(row) {
      this.$confirm('确定要删除吗？', '提示', {
        type: 'warning'
      })
        .then(() => {
          this.loading = true
          logDele({ admin_log_id: row.admin_log_id })
            .then(res => {
              this.$message({ message: res.msg, type: 'success' })
              this.logReset()
              this.logLists()
            })
            .catch(() => {
              this.loading = false
            })
        })
        .catch(() => {})
    },
    // 日志详情重置
    logReset(row = {}) {
      this.logModel = row
    },
    // 日志详情取消
    logCancel() {
      this.logDialog = false
      this.logReset()
    },
    // 日志详情确认
    logSubmit() {
      this.logDialog = false
      this.logReset()
    }
  }
}
</script>
