# el-selection-tree-table

基于element-ui table组件二次封装的树表格组件，主要解决element-ui中数表格中checkbox无法进行父子组件联动的问题，其中还包含了element-ui中的分页组件，needReverse参数用于配置是否需要在分页过程中保持每一页的选中状态和expand状态



<p style="color: red; font-weight: bolder">该组件依赖于element-ui，在使用该组件前请确保全局安装el-table el-table-column和el-pagination组件 </p>



## 安装

```
npm install @luxiaodan/el-selection-tree-table
```

### 使用
```vue

<template>
  <div>
    <selection-tree-table 
      :table-data="tableData" 
      :columns="columns"
      row-key="rowKey" 
      style="width: 100%"
      border
      :current-page="pageNo"
      ::page-size="pageSize"
      :total="2"
      :has-pagination="false"
      :pagination-props="{
        layout: 'prev, pager, next, sizes',
        pageSizes: [1,2]
      }"
      :need-reverse="true"
      @select-change="selectChangeHandler"
      @current-change="pageChange"
      @size-change="sizeChangeHandler"
    >
    <template slot="expr" slot-scope="scope">
      <span v-if="scope.row.expr">{{ scope.row.expr }}</span>
      <span v-else>————</span>
    </template>
    </selection-tree-table>
  </div>

</template>

<script>
import data from './data'
import SelectionTreeTable from '@luxiaodan/el-selection-tree-table'

export default {
  name: 'Example',
  components: {
    SelectionTreeTable
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
```
```js
const data = [
  {
    rowKey: 2,
    name: '设计岗',
    leader: '吴九',
    createTime: '2018-07-24',
    expr: '',
    date: '26天',
    children: [
      {
        rowKey: 21,
        name: '视觉设计',
        leader: '吴九',
        createTime: '2018-07-24',
        expr: '',
        date: '26天',
        children: [
          {
            rowKey: 211,
            name: '视觉设计师',
            leader: '吴小九',
            createTime: '2018-07-24',
            expr: '1-3年',
            date: '26天'
          },
          {
            rowKey: 212,
            name: '网页设计师',
            leader: '吴小十',
            createTime: '2018-07-24',
            expr: '3-5年',
            date: '26天'
          }
        ]
      },
      {
        rowKey: 22,
        name: '交互设计/用户体验',
        leader: '郑十',
        createTime: '2018-07-24',
        expr: '',
        date: '26天',
        children: [
          {
            rowKey: 221,
            name: '交互设计师',
            leader: '郑小⑩',
            createTime: '2018-07-24',
            expr: '5-10年',
            date: '24天'
          }
        ]
      }
    ]
  }
]

export default data
```

### props

| props                    | 说明                                           |
| ------------------------ | ---------------------------------------------- |
| tableData                | 表格数据                                       |
| columns                  | 表头配置，参考el-table Table-column Attributes |
| rowKey                   | 必须，默认为id                                 |
| hasPagination            | 是否需要分页，默认为true                       |
| currentPage              | 当前页 hasPagination=true为必须                |
| pageSize                 | pageSize， hasPagination=true为必须            |
| total                    | hasPagination=true为必须                       |
| paginationProps          | 其他Pagination配置项 参考el-pagination组件     |
| needReverse              | 是否需要保持分页里每一个页面的选中和折叠状态   |
| 其他参数配置参考el-table |                                                |

### events

| 事件          | 说明                                 | 参数                                    |
| ------------- | ------------------------------------ | --------------------------------------- |
| select        | checkbox点击后发出的事件，同el-table | selections, selectionIds, row           |
| select-all    | 同el-table                           | selections, selectionIds, isAllSelected |
| select-change |                                      | selections, selectionIds                |
|               | 其他表格事件参考el-table             |                                         |
|               | 其他分页事件参考el-pagination        |                                         |



