<template>
  <div class="app-container">
    <!-- 地区添加 -->
    <div class="filter-container">
      <el-button class="filter-item" type="primary" style="float:right;margin-left:10px" @click="regionAddition('')">添加</el-button>
      <el-button class="filter-item" type="primary" style="float:right;" @click="regionRefresh">刷新</el-button>
    </div>
    <!-- 地区列表 -->
    <el-table
      v-loading="loading"
      :data="regionData"
      :height="height+100"
      style="width: 100%"
      row-key="region_id"
      border
      lazy
      :load="regionListLoad"
      @sort-change="regionSort"
    >
      <el-table-column prop="region_name" label="名称" min-width="250" fixed="left" />
      <el-table-column prop="region_pinyin" label="拼音" min-width="250" />
      <el-table-column prop="region_jianpin" label="简拼" min-width="120" sortable="custom" />
      <el-table-column prop="region_initials" label="首字母" min-width="90" sortable="custom" />
      <el-table-column prop="region_citycode" label="区号" min-width="80" sortable="custom" />
      <el-table-column prop="region_zipcode" label="邮编" min-width="80" sortable="custom" />
      <el-table-column prop="region_sort" label="排序" min-width="80" sortable="custom" />
      <el-table-column prop="region_id" label="ID" min-width="80" sortable="custom" />
      <el-table-column prop="region_pid" label="PID" min-width="80" />
      <el-table-column label="操作" width="220" fixed="right" align="right" class-name="small-padding fixed-width">
        <template slot-scope="{ row }">
          <el-button size="mini" type="primary" @click="regionAddition(row)">添加</el-button>
          <el-button size="mini" type="primary" @click="regionModify(row)">修改</el-button>
          <el-button size="mini" type="danger" @click="regionDelete(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 地区添加、修改 -->
    <el-dialog :title="regionModel.dialog_title" :visible.sync="regionDialog" top="1vh">
      <el-form ref="formRef" :rules="regionRules" :model="regionModel" class="dialog-body" label-width="100px" :style="{height:height+80+'px'}">
        <el-form-item label="父级" prop="region_pid">
          <el-cascader v-model="regionModel.region_pid" :options="regionData" :props="regionProps" style="width:100%" clearable filterable placeholder="一级" @change="regionPidChange" />
        </el-form-item>
        <el-form-item label="名称" prop="region_name">
          <el-input v-model="regionModel.region_name" clearable placeholder="请输入名称：北京市" />
        </el-form-item>
        <el-form-item label="拼音">
          <el-col :span="11">
            <el-input v-model="regionModel.region_pinyin" clearable placeholder="请输入拼音：Beijing" />
          </el-col>
          <el-col class="line" :span="3" style="text-align:center">排序</el-col>
          <el-col :span="10">
            <el-input v-model="regionModel.region_sort" clearable placeholder="请输入排序：1000" />
          </el-col>
        </el-form-item>
        <el-form-item label="简拼">
          <el-col :span="11">
            <el-input v-model="regionModel.region_jianpin" clearable placeholder="请输入简拼：BJ" />
          </el-col>
          <el-col class="line" :span="3" style="text-align:center">首字母</el-col>
          <el-col :span="10">
            <el-input v-model="regionModel.region_initials" clearable placeholder="请输入首字母：B" />
          </el-col>
        </el-form-item>
        <el-form-item label="区号" prop="region_citycode">
          <el-input v-model="regionModel.region_citycode" clearable placeholder="请输入区号：010" />
        </el-form-item>
        <el-form-item label="邮编" prop="region_zipcode">
          <el-input v-model="regionModel.region_zipcode" clearable placeholder="请输入邮编：100000" />
        </el-form-item>
        <el-form-item label="" prop="">
          <el-col :span="12">
            <el-link type="primary" href="https://www.ip138.com/post/" target="_blank">查询区号邮编</el-link>
          </el-col>
          <el-col :span="12">
            <el-link type="primary" href="http://api.map.baidu.com/lbsapi/getpoint/index.html" target="_blank">查询经度纬度</el-link>
          </el-col>
        </el-form-item>
        <el-form-item label="经度" prop="region_longitude">
          <el-input v-model="regionModel.region_longitude" clearable placeholder="请输入经度：116.403263" />
        </el-form-item>
        <el-form-item label="纬度" prop="region_latitude">
          <el-input v-model="regionModel.region_latitude" clearable placeholder="请输入纬度：39.915156" />
        </el-form-item>
        <el-form-item label="添加">
          <el-col :span="11">
            <el-input v-model="regionModel.create_time" clearable placeholder="" disabled />
          </el-col>
          <el-col class="line" :span="3" style="text-align:center">更新</el-col>
          <el-col :span="10">
            <el-input v-model="regionModel.update_time" clearable placeholder="" disabled />
          </el-col>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="regionCancel">取消</el-button>
        <el-button type="primary" @click="regionSubmit">提交</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import screenHeight from '@/utils/screen-height'

import {
  regionList,
  regionInfo,
  regionAdd,
  regionEdit,
  regionDele
} from '@/api/region'

export default {
  name: 'Regions',
  components: { },
  data() {
    return {
      height: 680,
      loading: false,
      loadingTime: 0,
      regionData: [],
      regionCount: 0,
      regionQuery: {},
      regionDialog: false,
      regionProps: {
        expandTrigger: 'click',
        checkStrictly: true,
        value: 'region_id',
        label: 'region_name',
        lazy: true,
        lazyLoad(node, resolve) {
          regionList({ region_pid: node.data.region_id })
            .then(res => {
              resolve(res.data.list)
            })
            .catch(() => {})
        }
      },
      regionModel: {
        dialog_title: '',
        region_id: '',
        region_pid: 0,
        region_name: '',
        region_pinyin: '',
        region_jianpin: '',
        region_initials: '',
        region_citycode: '',
        region_zipcode: '',
        region_longitude: '',
        region_latitude: '',
        region_sort: 1000
      },
      regionRules: {
        region_name: [{ required: true, message: '请输入名称', trigger: 'blur' }],
        region_pinyin: [{ required: true, message: '请输入拼音', trigger: 'blur' }],
        region_jianpin: [{ required: true, message: '请输入简拼', trigger: 'blur' }],
        region_initials: [{ required: true, message: '请输入首字母', trigger: 'blur' }],
        region_citycode: [{ required: true, message: '请输入区号', trigger: 'blur' }],
        region_zipcode: [{ required: true, message: '请输入邮编', trigger: 'blur' }],
        region_longitude: [{ required: true, message: '请输入经度', trigger: 'blur' }],
        region_latitude: [{ required: true, message: '请输入纬度', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.height = screenHeight()
    this.regionList()
  },
  methods: {
    loadOpen() {
      this.loading = true
    },
    loadClose() {
      const that = this
      setTimeout(function() {
        that.loading = false
      }, that.loadingTime)
    },
    message(msg, type = 'success') {
      this.$message({
        message: msg,
        type: type
      })
    },
    // 地区列表
    regionList() {
      this.loadOpen()
      this.regionData = []
      regionList(this.regionQuery)
        .then(res => {
          this.regionData = res.data.list
          this.regionCount = res.data.count
          this.loadClose()
        })
        .catch(() => {
          this.loadClose()
        })
    },
    // 地区列表排序
    regionSort(sort) {
      this.regionQuery.sort_field = sort.prop
      this.regionQuery.sort_type = ''
      if (sort.order === 'ascending') {
        this.regionQuery.sort_type = 'asc'
        this.regionList()
      }
      if (sort.order === 'descending') {
        this.regionQuery.sort_type = 'desc'
        this.regionList()
      }
    },
    // 地区列表加载
    regionListLoad(tree, treeNode, resolve) {
      regionList({ region_pid: tree.region_id })
        .then(res => {
          resolve(res.data.list)
        })
        .catch(() => {})
    },
    // 地区信息
    regionInfo(region_id) {
      regionInfo({ region_id: region_id })
        .then(res => {
          this.regionReset(res.data)
        })
    },
    // 地区刷新
    regionRefresh() {
      this.regionQuery = {}
      this.regionList()
    },
    // 地区添加
    regionAddition(row) {
      this.regionDialog = true
      if (row) {
        this.regionModel.dialog_title = '地区添加'
        this.regionModel.region_id = ''
        this.regionModel.region_pid = row.region_id
        this.regionModel.region_name = ''
        this.regionModel.region_pinyin = ''
        this.regionModel.region_jianpin = ''
        this.regionModel.region_initials = ''
        this.regionModel.region_citycode = ''
        this.regionModel.region_zipcode = ''
        this.regionModel.region_longitude = ''
        this.regionModel.region_latitude = ''
        this.regionModel.region_sort = 1000
      } else {
        this.regionReset()
      }
    },
    // 地区修改
    regionModify(row) {
      this.regionDialog = true
      this.regionInfo(row.region_id)
    },
    // 地区删除
    regionDelete(row) {
      this.$confirm('确定删除吗（它所有子地区也将删除）？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.loadOpen()
          regionDele({ region_id: row.region_id })
            .then(res => {
              this.$message({ type: 'success', message: res.msg })
              this.regionReset()
              this.regionList()
            })
            .catch(() => {
              this.loadClose()
            })
        })
        .catch(() => {})
    },
    // 地区经纬度获取
    regionLonLatBtn() {
      this.regionLonLatDialog = true
    },
    // 地区父级选择
    regionPidChange(value) {
      if (value) {
        this.regionModel.region_pid = value[value.length - 1]
      }
    },
    // 地区添加、修改重置
    regionReset(row) {
      if (row) {
        this.regionModel = row
        this.regionModel.dialog_title = '地区修改：' + row.region_name
      } else {
        this.regionModel = {}
        this.regionModel.dialog_title = '地区添加'
        this.regionModel.region_pid = 0
      }
    },
    // 地区添加、修改取消
    regionCancel() {
      this.regionDialog = false
      this.regionReset()
    },
    // 地区添加、修改提交
    regionSubmit() {
      this.$refs['formRef'].validate(valid => {
        if (valid) {
          this.loadOpen()
          if (this.regionModel.region_id) {
            regionEdit(this.regionModel)
              .then(res => {
                this.regionDialog = false
                this.message(res.msg)
                this.regionReset()
                this.regionList()
              })
              .catch(() => {
                this.loadClose()
              })
          } else {
            regionAdd(this.regionModel)
              .then(res => {
                this.regionDialog = false
                this.message(res.msg)
                this.regionReset()
                this.regionList()
              })
              .catch(() => {
                this.loadClose()
              })
          }
        }
      })
    }
  }
}
</script>
