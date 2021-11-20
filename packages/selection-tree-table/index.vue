<template>
  <div class="selection-tree-table">
    <el-table
      ref="table"
      :data="treeTableData"
      :tree-props="treeProps"
      :row-key="rowKey"
      v-bind="$attrs"
      v-on="$listeners"
      @expand-change="expandChangeHandler"
      @select="selectionChangeHandler"
      @select-all="selectAllHandler">
      <el-table-column type="selection" width="55" />
      <el-table-column v-for="col in columns" :key="col.prop" v-bind="col">
        <template slot-scope="{row, index}">
          <slot 
            v-if="$scopedSlots[col.prop]"  
            :name="col.prop" 
            :row="row" 
            :index="index"/>
            <span v-else>{{ row[col.prop] }}</span>
        </template>
      </el-table-column>
    </el-table>
    <div class="selection-tree-table__pagination" v-if="hasPagination">
      <el-pagination
        :current-page="currentPage"
        :page-size="pageSize"
        :total="total"
        v-bind="paginationProps"
        v-on="$listeners" />
    </div>
  </div>
</template>

<script>
import cloneDeep from 'lodash/cloneDeep'
// import { Table, TableColumn, Pagination } from 'element-ui'

export default {
  name: 'SelectionTreeTable',
  props: {
    tableData: Array,
    columns: {
      type: Array,
      required: true
    },
    rowKey: {
      type: String,
      default: 'id'
    },
    hasPagination: {
      type: Boolean,
      default: true
    },
    currentPage: Number,
    pageSize: Number,
    total: Number,
    paginationProps: {
      type: Object,
      default: () => ({})
    },
    needReverse: { // 分页后是否需要保持上一页的选中和折叠状态
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      treeTableData: [],
      selections: [],
      flatTreeDate: [],
      expandRows: [],
      defaultTreeProps: { children: 'children' }
    }
  },
  // components: {
  //   'ElTable': Table,
  //   'ElTableColumn': TableColumn,
  //   'ElPagination': Pagination
  // },
  computed: {
    treeProps () {
      return {
        ...this.defaultTreeProps,
        ...(this.$attrs.treeProps || {})
      }
    },
    selectionIds() {
      return this.selections.map(item => item[this.rowKey])
    }
  },
  watch: {
    tableData() {
      this.treeTableData = cloneDeep(this.tableData)
      this.flatTreeDate = []
      this.formatTableRows(this.treeTableData)
      if(!this.needReverse) {
        this.selections = []
      }
      // 切换页面保持之前的选中的状态
      if(this.needReverse) {
          this.$nextTick(() => {
          if(!this.$refs.table) return 
          this.$refs.table.clearSelection()
          if (this.flatTreeDate.some(item => this.isSelected(item))) {
            this.setSelection()
          }
          this.setExpand()
        })
      }
    }
  },
  created() {
    this.treeTableData= cloneDeep(this.tableData)
    this.formatTableRows(this.treeTableData)
  },
  methods: {
    formatTableRows(rows, level = 0) {
      rows && rows.forEach(row => {
        row.$level = level
        row.$children = row[this.treeProps.children]
        this.flatTreeDate.push(row)
        if (row.$children && row.$children.length > 0) {
          row.$children.forEach(children => {
            children.$parent = row
          })
          this.formatTableRows(row.$children, level + 1)
        }
      })
    },
    findIndex(source, target) {
      return source.findIndex(item => item[this.rowKey] === target[this.rowKey])
    },
    selectionChangeHandler(selection, row) {
      const isChecked = (this.findIndex(selection, row) >= 0)
      if (isChecked) {
        // 该节点加入所有选项中
        if (this.findIndex(this.selections, row) === -1) {
          this.selections.push(row)
        }
        // 该节点下的所有子节点需全部选中
        this.checkDeepChildren(row)
        // 如果该节点的父节点下的子节点全部选中，则父节点也需要选中
        this.checkDeepParent(row)
      } else {
        // 该节点从所有选项中删除
        const index = this.findIndex(this.selections, row)
        if (index !== -1) {
          this.selections.splice(index, 1)
        }
        // 该节点下的子节点全部取消勾选
        this.cancelDeepChildren(row)
        // 该节点的父节点及祖先全部取消勾选
        this.cancelDeepParent(row)
      }
      this.$emit('select', this.selections, this.selectionIds, row)
      this.$emit('select-change', this.selections, this.selectionIds)
    },
    isSelected(node) {
      return this.selectionIds.includes(node[this.rowKey])
    },
    toggleRowSelection(row, toggle) {
      const index = this.findIndex(this.selections, row)
      this.$refs.table.toggleRowSelection(row, toggle)
      if (toggle && index === -1) {
        this.selections.push(row)
      } else if (!toggle && index !== -1) {
        this.selections.splice(index, 1)
      }
    },
    checkDeepChildren(row) {
      if (row.$children && row.$children.length > 0) {
        row.$children.forEach(children => {
          this.toggleRowSelection(children, true)
          this.checkDeepChildren(children)
        })
      }
    },
    cancelDeepChildren(row) {
      if (row.$children && row.$children.length > 0) {
        row.$children.forEach(children => {
          this.toggleRowSelection(children, false)
          this.cancelDeepChildren(children)
        })
      }
    },
    cancelDeepParent(row) {
      const parent = row.$parent
      if (parent) {
        this.toggleRowSelection(parent, false)
        this.cancelDeepParent(parent)
      }
    },
    checkDeepParent(row) {
      const parent = row.$parent
      if (parent && parent.children.every(node => this.isSelected(node))) {
        this.toggleRowSelection(parent, true)
        this.checkDeepParent(parent)
      }
    },
    selectAllHandler() {
      const isAllSelected = this.$refs.table.$refs.tableHeader.isAllSelected
      this.flatTreeDate.forEach(row => {
        this.toggleRowSelection(row, isAllSelected)
      })
      this.$emit('select-all', this.selections, this.selectionIds, isAllSelected)
      this.$emit('select-change', this.selections, this.selectionIds)
    },
    setSelection() {
      this.selectionIds.forEach(id => {
        const row = this.flatTreeDate.find(data => data[this.rowKey] === id)
        row && this.$refs.table.toggleRowSelection(row, true)
      })
    },
    setExpand() {
      this.expandRows.forEach(item => {
        this.$refs.table.toggleRowExpansion(item, true)
      })
    },
    expandChangeHandler(row, expanded) {
      if(this.needReverse) {
        const index = this.findIndex(this.expandRows, row)
        if (expanded && index === -1) {
          this.expandRows.push(row)
        } else if (!expanded && index !== -1) {
          this.expandRows.splice(index, 1)
        }
      }
      this.$emit('expand-change', row, expanded)
    },

    /**供外部组件调用 */
    clearAllSelection() {
      this.$refs.table.clearSelection()
      this.selections = []
    },
    getSelections() {
      return this.selections
    },
    getSeletionIds() {
      return this.selectionIds
    }
  }
}
</script>

<style scoped>
.selection-tree-table__pagination {
  width: 100%;
  padding: 10px 0;
  display: flex;
}
.selection-tree-table__pagination  .el-pagination {
  margin-left: auto;
}
</style>