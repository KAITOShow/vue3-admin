<script lang="ts" setup>
import { ref } from "vue"
import logRecordLogin from "./login.vue"
import logRecordOperation from "./operation.vue"
import logRecordUser from "./user.vue"

defineOptions({
  // 命名当前组件
  name: "log"
})

const activeName = ref("operationLog")
</script>
// TODO 改成懒加载，不要一次性加载所有tab的api，只有点击tab的时候才加载对应的api，用户体验更好
<template>
  <div class="app-container">
    <el-tabs v-model="activeName" type="border-card" class="demo-tabs">
      <el-tab-pane label="操作日志" name="operationLog"><logRecordOperation /></el-tab-pane>
      <el-tab-pane label="用户操作日志" name="userLog"><logRecordUser /></el-tab-pane>
      <el-tab-pane label="登录日志" name="loginLog">
        <logRecordLogin />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<style lang="scss" scoped></style>
