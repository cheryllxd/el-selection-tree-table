
<template>
  <div id="app">
    <el-button @click="clear">清空</el-button>
    <selection-tree-table 
      :table-data="tableData" 
      :columns="columns"
      row-key="rowKey" 
      style="width: 100%;margin-bottom: 20px;"
      border
      :current-page="pageNo"
      :page-size='pageSize'
      :total="2"
      :pagination-props="{
        layout: 'prev, pager, next, sizes',
        pageSizes: [1,2]
      }"
      needReverse
      @select-change="selectChangeHandler"
      @current-change="pageChange"
      @size-change="sizeChangeHandler"
    >
    <template slot="expr" slot-scope="scope">
      <span v-if="scope.row.expr">{{scope.row.expr}}</span>
      <span v-else>————</span>
    </template>
    </selection-tree-table>
  </div>

</template>

<script>
import data from './data'
import SelectionTreeTable from '../packages/selection-tree-table'
import { Button } from 'element-ui';

export default {
  name: 'App',
  components: {
    SelectionTreeTable,
    'ElButton': Button
  },
  data() {
    return {
      tableData: [data[0]],
      columns: [
        {
          label: '岗位',
          prop: 'name'
        },
        {
          label: '编码',
          prop: 'rowKey'
        },
        {
          label: '负责人',
          prop: 'leader'
        },
        {
          label: '创建时间',
          prop: 'createTime'
        },
        {
          label: '经验要求',
          prop: 'expr'
        },
        {
          label: '发布天数',
          prop: 'date'
        }
      ],
      rowKey: 'rowKey',
      pageSize: 1,
      pageNo: 1
    }
  },
  methods: {
    pageChange(curPage) {
      this.pageNo = curPage
      const start = (curPage - 1) * this.pageSize
      const end = start + this.pageSize
      this.tableData = data.slice(start, end)
    },
    expandChangeHandler(row, expanded) {
      const index = this.findIndex(this.expandRows, row)
      if (expanded && index === -1) {
        this.expandRows.push(row)
      } else if (!expanded && index !== -1) {
        this.expandRows.splice(index, 1)
      }
    },
    sizeChangeHandler(size) {
      this.pageNo = 1
      this.pageSize = size
      this.pageChange(this.pageNo)
    },
    clear() {
      this.$refs.table.clearSelection()
      this.selections = []
    },
    selectChangeHandler(selection, selectionIds) {
      console.log(selectionIds)
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
