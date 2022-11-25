<script setup>
const props = defineProps({
  /* data 和 draggable 都是 给 el-table 组件使用的 */
  data: { type: Array, required: true },
  draggable: { type: Boolean, default: false },
  /* columns 是给 el-table-column 组件使用的 */
  columns: {
    type: Array,
    required: true,
    /* 数组中需要的是一个对象*/
    /*
    这些 props 都是 el-table-column 的 props 配置项
    自定义添加了 slot 选项，可以从组件最外层传递选项进去配置插槽。
    props: {
      // 自定义封装的 props 选项
        slots:{
          // key 是 el-table-column 的 default 插槽
          // value 是在 外面传递插槽的时候 写的插槽名字。
          // 例如 slots:{default:'operate'}
          // 那么就可以在 AbtSmartTable 下 写 operate 插槽。
          type:[Object, null],
          default:()=>null
        },
        id:{
          type:[String, Number, null, undefined],
          default:()=>null
        }
        // 以下全部是 el-table-column props 选项。
        type: {
          type: String,
          default: 'default'
        },
        label: String,
        className: String,
        labelClassName: String,
        property: String,
        prop: String,
        width: {},
        minWidth: {},
        renderHeader: Function,
        sortable: {
          type: [Boolean, String],
          default: false
        },
        sortMethod: Function,
        sortBy: [String, Function, Array],
        resizable: {
          type: Boolean,
          default: true
        },
        columnKey: String,
        align: String,
        headerAlign: String,
        showTooltipWhenOverflow: Boolean,
        showOverflowTooltip: Boolean,
        fixed: [Boolean, String],
        formatter: Function,
        selectable: Function,
        reserveSelection: Boolean,
        filterMethod: Function,
        filteredValue: Array,
        filters: Array,
        filterPlacement: String,
        filterMultiple: {
          type: Boolean,
          default: true
        },
        index: [Number, Function],
        sortOrders: {
          type: Array,
          default() {
            return ['ascending', 'descending', null];
          },
          validator(val) {
            return val.every(order => ['ascending', 'descending', null].indexOf(order) > -1);
          }
        }
      },
     */
  },
})
const attrs = useAttrs()
</script>
<script>
import { defineComponent } from 'vue'
import Sortable from 'sortablejs'

export default defineComponent({
  name: 'BaseTable',
  data() {
    return {
      list: null,
      total: null,
      sortable: null,
      oldList: [],
      newList: [],
    }
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger',
      }
      return statusMap[status]
    },
    getColumnPropValue(option, slotProps) {
      const row = slotProps.row
      const prop = option ? option.prop : undefined
      let result
      if (prop) {
        try {
          result = row[prop]
        } catch (error) {
          throw new Error(`prop不存在，请确认 columnOption 和 data 中是否存在 ${prop} 属性`)
        }
      } else {
        result = '无数据'
      }
      return result
    },
  },
  methods: {
    // 在 vue3 中 $attrs and $listeners 都在 attrs 中
    getListeners() {
      return this.$listeners
    },
    getAttrs() {
      return this.$attrs
    },
    /* 表格拖动 */
    initSortData() {
      if (this.draggable) {
        this.list = this.data.map((v) => v)
        this.total = this.$attrs.total
        /* 排序用的是 id */
        this.oldList = this.list.map((v) => v.id)
        this.newList = this.oldList.slice()
      }
    },
    initSort() {
      if (this.draggable) {
        this.$nextTick(() => {
          this.setSort()
        })
      }
    },
    setSort() {
      const el = this.$refs.dragTable.$el.querySelectorAll('.el-table__body-wrapper > table > tbody')[0]
      this.sortable = Sortable.create(el, {
        ghostClass: 'sortable-ghost', // Class name for the drop placeholder,
        setData: function (dataTransfer) {
          // to avoid Firefox bug
          // Detail see : https://github.com/RubaXa/Sortable/issues/1012
          dataTransfer.setData('Text', '')
        },
        onEnd: (evt) => {
          /* 操作仅仅只操作 list 和 newList */
          const targetRow = this.list.splice(evt.oldIndex, 1)[0]
          this.list.splice(evt.newIndex, 0, targetRow)

          // for show the changes, you can delete in you code
          const tempIndex = this.newList.splice(evt.oldIndex, 1)[0]
          this.newList.splice(evt.newIndex, 0, tempIndex)
        },
      })
      console.log(this.sortable)
    },
  },
  computed: {
    listeners() {
      // 因为 composition api 中已经没有 $listeners 了，
      // 为了在模板中使用不报错，所以在这里定义下。
      return this.$listeners
    },
  },
  watch: {
    data() {
      if (this.data !== null && this.draggable) {
        this.initSortData()
      }
    },
    draggable(newval) {
      /*当draggble为 true 的时候，我们允许拖动，否则禁用。 */
      this.sortable.option('disabled', !newval)
      console.log(newval)
    },
  },
  mounted() {
    if (this.data !== null && this.draggable) {
      this.initSortData()
      this.initSort()
    }
  },
})
</script>
<template>
  <div>
    <el-table ref="dragTable" :data="props.data ? props.data : []" v-on="listeners" v-bind="attrs" style="width: 100%">
      <el-table-column v-bind="columnOption" v-for="(columnOption, idx) in props.columns" :key="columnOption.id">
        <template v-if="columnOption?.slots?.default" #default="slotProps">
          <slot :name="columnOption?.slots?.default" :slotProps="slotProps">
            {{ columnOption | getColumnPropValue(slotProps) }}
          </slot>
        </template>
      </el-table-column>
    </el-table>
    <div v-if="props.draggable">
      <div class="show-d"><el-tag>The default order :</el-tag> {{ oldList }}</div>
      <div class="show-d"><el-tag>The after dragging order :</el-tag> {{ newList }}</div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
/* 拖动相关 */
::v-deep .sortable-ghost {
  opacity: 0.8;
  color: #fff !important;
  background: #42b983 !important;
}

.icon-star {
  margin-right: 2px;
}

.drag-handler {
  width: 20px;
  height: 20px;
  cursor: pointer;
}

.show-d {
  margin-top: 15px;
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