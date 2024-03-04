<script lang="ts" setup>
import { reactive, ref, watch, computed } from "vue"
import { type FormInstance, type FormRules, ElMessage, ElMessageBox } from "element-plus"
import { CirclePlus } from "@element-plus/icons-vue"
import { usePagination } from "@/hooks/usePagination"
import { formatDateTime } from "@/utils/index"

import { Form, Create, Delete, Get, List, Update, apiPrefix } from "@/api/system/group"
import { List as UserList } from "@/api/system/user"
import { User } from "@/api/system/user"
import { Group } from "@/api/system/group"

defineOptions({
  // 命名当前组件
  name: "group"
})

const loading = ref<boolean>(false)
const { paginationData, handleCurrentChange, handleSizeChange } = usePagination()

//#region 增
const dialogVisible = ref<boolean>(false)
const formRef = ref<FormInstance | null>(null)

const formData = ref<Form>({})

const formRules: FormRules = reactive({
  name: [{ required: true, trigger: "blur", message: "请输入名称" }],
  sort: [{ required: true, trigger: "blur", message: "请输入排序" }],
  status: [{ required: true, trigger: "change", message: "请选择启用状态" }]
})

const handleCreate = () => {
  formRef.value?.validate((valid: boolean, fields) => {
    if (valid) {
      if (currentUpdateId.value === undefined) {
        Create(formData.value)
          .then(() => {
            ElMessage.success("新增成功")
            getTableData()
          })
          .finally(() => {
            dialogVisible.value = false
          })
      } else {
        Update(currentUpdateId.value, {
          ...formData.value
        })
          .then(() => {
            ElMessage.success("修改成功")
            getTableData()
          })
          .finally(() => {
            dialogVisible.value = false
          })
      }
    } else {
      console.error("表单校验不通过", fields)
    }
  })
}
const handleCreateSon = (row: Group) => {
  formData.value.parent_id = row.id
  dialogVisible.value = true
}
const resetForm = () => {
  currentUpdateId.value = undefined
  formRef.value?.resetFields()
  formData.value = {}
}
//#endregion

//#region 删
const handleDelete = (row: Group) => {
  ElMessageBox.confirm(`确定要删除吗？`, "提示", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
    type: "warning"
  }).then(() => {
    Delete({ id: [row.id] }).then(() => {
      ElMessage.success("删除成功")
      getTableData()
    })
  })
}
//#endregion

//#region 改
const currentUpdateId = ref<undefined | number>(undefined)
const handleUpdate = (row: Group) => {
  Get(row.id).then((res) => {
    currentUpdateId.value = res.data.id
    Object.assign(formData.value, res.data || {})
    dialogVisible.value = true
  })
}
//#endregion

//#region 查
const tableData = ref<Group[]>([])
const getTableData = () => {
  loading.value = true
  List({
    page: paginationData.currentPage,
    page_size: paginationData.pageSize
  })
    .then((res) => {
      paginationData.total = res.data.total
      tableData.value = res.data.list
      handleGroupOption()
    })
    .catch(() => {
      tableData.value = []
    })
    .finally(() => {
      loading.value = false
    })
}

//#endregion
//#region 其他
const userList = ref<User[]>([])
const getUserList = () => {
  UserList({ page: 1, page_size: 999 }).then((res) => {
    userList.value = res.data.list
  })
}
getUserList()

const groupCascaderProps = {
  emitPath: false,
  checkStrictly: true,
  label: "name",
  value: "id"
}
let groupOption = [{ id: 0, name: "顶级部门" }]
const handleGroupOption = () => {
  groupOption = [{ id: 0, name: "顶级部门" }]
  tableData.value.forEach((element) => {
    groupOption.push(element)
  })
}
//#endregion
/** 监听分页参数的变化 */
watch([() => paginationData.currentPage, () => paginationData.pageSize], getTableData, { immediate: true })
const timeHandle = computed(() => (time: string) => {
  return formatDateTime(time)
})
</script>

<template>
  <div class="app-container">
    <el-card v-loading="loading" shadow="never">
      <div class="toolbar-wrapper">
        <div>
          <el-button type="primary" v-permission="apiPrefix + '_POST'" :icon="CirclePlus" @click="dialogVisible = true"
            >新增部门</el-button
          >
        </div>
      </div>
      <div class="table-wrapper">
        <el-table :data="tableData" row-key="id">
          <el-table-column prop="name" label="部门名称" min-width="130" />
          <el-table-column prop="sort" label="排序" min-width="110" />
          <el-table-column prop="status" label="启用" min-width="110">
            <template #default="scope">
              {{ scope.row.status ? "是" : "否" }}
            </template>
          </el-table-column>
          <el-table-column prop="created_at" label="创建时间" min-width="160" sortable>
            <template #default="scope">
              {{ timeHandle(scope.row.created_at) }}
            </template>
          </el-table-column>
          <el-table-column prop="updated_at" label="最后更新时间" min-width="160" sortable>
            <template #default="scope">
              {{ timeHandle(scope.row.updated_at) }}
            </template>
          </el-table-column>
          <el-table-column label="操作" width="200" align="center">
            <template #default="scope">
              <el-button
                type="primary"
                v-permission="apiPrefix + '_POST'"
                text
                bg
                size="small"
                @click="handleCreateSon(scope.row)"
                >子部门</el-button
              >
              <el-button
                type="primary"
                v-permission="apiPrefix + '/:id_PUT'"
                text
                bg
                size="small"
                @click="handleUpdate(scope.row)"
                >修改</el-button
              >
              <el-button
                type="danger"
                v-permission="apiPrefix + '_DELETE'"
                text
                bg
                size="small"
                @click="handleDelete(scope.row)"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="pager-wrapper">
        <el-pagination
          background
          :layout="paginationData.layout"
          :page-sizes="paginationData.pageSizes"
          :total="paginationData.total"
          :page-size="paginationData.pageSize"
          :currentPage="paginationData.currentPage"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>
    <!-- 新增/修改 -->
    <el-dialog
      v-model="dialogVisible"
      :title="currentUpdateId === undefined ? '新增' : '修改'"
      @close="resetForm"
      width="30%"
    >
      <el-form ref="formRef" :model="formData" :rules="formRules" label-width="100px" label-position="right">
        <el-form-item prop="name" label="部门名称">
          <el-input v-model="formData.name" placeholder="请输入" />
        </el-form-item>
        <el-form-item prop="parentId" label="上级部门">
          <el-cascader
            v-model="formData.parent_id"
            :options="groupOption"
            :props="groupCascaderProps"
            placeholder="请选择"
            :show-all-levels="false"
            clearable
          />
        </el-form-item>
        <el-form-item prop="sort" label="排序">
          <el-input v-model.number="formData.sort" placeholder="请输入" />
        </el-form-item>
        <el-form-item prop="status" label="是否启用">
          <el-radio-group v-model="formData.status">
            <el-radio :label="1">是</el-radio>
            <el-radio :label="0">否</el-radio>
          </el-radio-group>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleCreate">确认</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<style lang="scss" scoped>
.search-wrapper {
  margin-bottom: 20px;
  :deep(.el-card__body) {
    padding-bottom: 2px;
  }
}

.toolbar-wrapper {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.table-wrapper {
  margin-bottom: 20px;
}

.pager-wrapper {
  display: flex;
  justify-content: flex-end;
}
</style>