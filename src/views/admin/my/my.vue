<template>
  <div class="app-container">
    <el-tabs v-model="activeName">
      <el-tab-pane v-if="checkPermission(['admin/AdminMy/myInfo'])" label="我的信息" name="info">
        <MyInfo />
      </el-tab-pane>
      <el-tab-pane v-if="checkPermission(['admin/AdminMy/myEdit'])" label="修改信息" name="edit">
        <MyEdit />
      </el-tab-pane>
      <el-tab-pane v-if="checkPermission(['admin/AdminMy/myPwd'])" label="修改密码" name="pwd">
        <MyPwd />
      </el-tab-pane>
      <el-tab-pane v-if="checkPermission(['admin/AdminMy/myAvatar'])" label="修改头像" name="avatar">
        <MyAvatar />
      </el-tab-pane>
      <el-tab-pane v-if="checkPermission(['admin/AdminMy/myLog'])" label="我的日志" name="log">
        <MyLog />
      </el-tab-pane>
      <el-tab-pane v-if="checkPermission(['admin/AdminMy/mySetting'])" label="我的设置" name="setting">
        <MySetting />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import checkPermission from '@/utils/permission' // 权限判断函数
import MyInfo from './components/info'
import MyEdit from './components/edit'
import MyPwd from './components/pwd'
import MyAvatar from './components/avatar'
import MyLog from './components/log'
import MySetting from './components/setting'

export default {
  name: 'My',
  components: { MyInfo, MyEdit, MyPwd, MyAvatar, MyLog, MySetting },
  data() {
    return {
      activeName: 'info'
    }
  },
  created() {},
  methods: { checkPermission }
}
</script>
