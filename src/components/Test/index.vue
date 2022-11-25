<script setup>
import { Message } from "element-ui"
import { fetchList } from '@/api/article'
import { set } from "vue"
import request from "@/utils/request"
const props = defineProps({})
const list = ref([])
const total = ref(null)
const listLoading = ref(true)
const listQuery = reactive({ page: 1, limit: 10 })
const test = () => console.log('test attrs')
/* 函数定义 */
const getList = async function () {
  listLoading.value = true
  const { data } = await fetchList(listQuery)
  const items = data.items
  total.value = data.total
  list.value = items.map(v => {
    set(v, 'edit', false)
    v.originalTitle = v.title //  will be used when user click the cancel botton
    return v
  })
  listLoading.value = false
}

/* 函数定义 */
/* 取消编辑 */
const cancelEdit = function (row) {
  row.title = row.originalTitle
  row.edit = false
  Message({
    message: 'The title has been restored to the original value',
    type: 'warning'
  })
}

/* 提交编辑 */
const confirmEdit = function (row) {
  row.edit = false
  row.originalTitle = row.title
  Message({
    message: 'The title has been edited',
    type: 'success'
  })
}

/* 执行函数 */
getList()

/* 其他测试 */
const foo = ref("composition api test")
const isDark = useDark()
const toggle = useToggle(isDark)
let result = ref('')

// test request
request({
  url: "https://jsonplaceholder.typicode.com/todos/1",
  method: 'get'
}).then(res => {
  console.log(res);
  result.value = res
})
const draggable = ref(true);

// table 列配置项与 el-table-
const columnOptions = ref([
  {
    align: "center",
    label: "Id",
    prop: "id",
    width: "80"
  },
  /* 测试下我们的动态插槽 */
  {
    slots: { default: "date" },
    align: "center",
    label: "Date",
    width: "180px",
  },
  {
    align: "center",
    width: "120px",
    label: "Author",
    prop: "author"
  },
  {
    slots: { default: "icon" },
    width: "100px",
    label: "Importance",
    prop: "importance"
  }, {
    slots: { default: "status" },
    "class-name": "status-col",
    label: "Status",
    width: "110",
    prop: "status"
  },
  {
    slots: { default: "title" },
    "min-width": "300px",
    label: "Title"
  },
  {
    slots: { default: "edit" },
    align: "center",
    label: "Actions",
    width: "120",
  }
])
</script>

<script>
import { defineComponent } from 'vue'
export default defineComponent({
  name: 'Test',
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
  },
  components: {}, data() { return {} }, watch: {}, computed: {}, created() { }, mounted() { }, methods: {}
})
</script>

<template>
  <div class="border-1 p-5 border-dark-100">
    <div class="pb-5"><span class="text-2xl font-bold">表格测试</span></div>
    <!-- 表格测试 -->
    <el-switch v-model="draggable" active-color="#13ce66" inactive-color="#ff4949">
    </el-switch>
    <BaseTable :columns="columnOptions" :draggable="draggable" v-loading="listLoading" :data="list ? list : []"
      border fit highlight-current-row v-on:current-change="test">
      <!-- 插槽测试 -->
      <template #date="{ slotProps }">
        {{ slotProps.row.timestamp | parseTime('{y}-{m}-{d} {h}:{i}') }}
      </template>
      <!-- <template name="as" #as="slotProps">
        {{ slotProps.slotProps.row }}
      </template> -->
      <template #icon="{ slotProps }">
        <svg-icon v-for="n in + slotProps?.row.importance" :key="n" icon-class="star" class="meta-item__icon" />
      </template>
      <template #status="{ slotProps }">
        <el-tag :type="slotProps.row.status | statusFilter">
          {{ slotProps.row.status }}
        </el-tag>
      </template>
      <template #title="{ slotProps }">
        <template v-if="slotProps.row.edit">
          <el-input v-model="slotProps.row.title" class="edit-input" size="small" />
          <el-button class="cancel-btn" size="small" icon="el-icon-refresh" type="warning"
            @click="cancelEdit(slotProps.row)">
            cancel
          </el-button>
        </template>
        <span v-else>{{ slotProps.row.title }}</span>
      </template>
      <template #edit="{ slotProps }">
        <el-button v-if="slotProps.row.edit" type="success" size="small" icon="el-icon-circle-check-outline"
          @click="confirmEdit(slotProps.row)">
          Ok
        </el-button>
        <el-button v-else type="primary" size="small" icon="el-icon-edit" @click="slotProps.row.edit = !slotProps.row.edit">
          Edit
        </el-button>
      </template>
    </BaseTable>

    <hr>
    <!-- 其他测试 -->
    <div class="pb-5"><span class="text-2xl font-bold">Test</span>
      <button class="flex items-center justify-center w-9 h-9 focus:outline-none" @click="toggle()">
        <pisMdiWhiteBalanceSunny v-if="isDark" class="text-yellow-500" />
        <!-- mdi:white-balance-sunny -->
        <!-- ep:sunrise -->
        <pisMdiMoonWaningCrescent v-else class="text-gray-800" />
      </button>
    </div>
    <hr>
    <div class="test-scss">test scss</div>
    <div>{{ foo }}</div>
    <el-button type="primary">test-el-button</el-button>
    <button type="button"
      class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800">Default</button>
    <button type="button"
      class="py-2.5 px-5 mr-2 mb-2 text-sm font-medium text-gray-900 focus:outline-none bg-white rounded-lg border border-gray-200 hover:bg-gray-100 hover:text-blue-700 focus:z-10 focus:ring-4 focus:ring-gray-200 dark:focus:ring-gray-700 dark:bg-gray-800 dark:text-gray-400 dark:border-gray-600 dark:hover:text-white dark:hover:bg-gray-700">Alternative</button>
    <button type="button"
      class="text-white bg-gray-800 hover:bg-gray-900 focus:outline-none focus:ring-4 focus:ring-gray-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-gray-800 dark:hover:bg-gray-700 dark:focus:ring-gray-700 dark:border-gray-700">Dark</button>
    <button type="button"
      class="text-gray-900 bg-white border border-gray-300 focus:outline-none hover:bg-gray-100 focus:ring-4 focus:ring-gray-200 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-gray-800 dark:text-white dark:border-gray-600 dark:hover:bg-gray-700 dark:hover:border-gray-600 dark:focus:ring-gray-700">Light</button>
    <button type="button"
      class="focus:outline-none text-white bg-green-700 hover:bg-green-800 focus:ring-4 focus:ring-green-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-green-600 dark:hover:bg-green-700 dark:focus:ring-green-800">Green</button>
    <button type="button"
      class="focus:outline-none text-white bg-red-700 hover:bg-red-800 focus:ring-4 focus:ring-red-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:bg-red-600 dark:hover:bg-red-700 dark:focus:ring-red-900">Red</button>
    <button type="button"
      class="focus:outline-none text-white bg-yellow-400 hover:bg-yellow-500 focus:ring-4 focus:ring-yellow-300 font-medium rounded-lg text-sm px-5 py-2.5 mr-2 mb-2 dark:focus:ring-yellow-900">Yellow</button>
    <button type="button"
      class="focus:outline-none text-white bg-purple-700 hover:bg-purple-800 focus:ring-4 focus:ring-purple-300 font-medium rounded-lg text-sm px-5 py-2.5 mb-2 dark:bg-purple-600 dark:hover:bg-purple-700 dark:focus:ring-purple-900">Purple</button>
    <button type="button"
      class="testAt focus:outline-none text-white bg-blue-500 hover:bg-purple-800 focus:ring-4 font-medium rounded-lg text-sm px-5 py-2.5 mb-2 dark:bg-purple-600 dark:hover:bg-purple-700 dark:focus:ring-purple-900">TestAt</button>

    <hr />
    <span> 测试图标自动导入 pisTablerAlien </span>
    <pisTablerAlien></pisTablerAlien>
    <hr>
    <div>
      {{ result }}
    </div>
  </div>
</template>

<style lang='scss' scoped>
.test-scss {
  color: rgb(0, 255, 128);
}

:-moz-focusring {
  outline: auto;
}

/* 简单的测试样式 */
.testAt:focus {
  // outline: 3px solid orange;
  @apply bg-blue-500 hover: bg-pink-300 text-white;
}


/* 行内编辑相关 */
.edit-input {
  padding-right: 100px;
}

.cancel-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}
</style>
