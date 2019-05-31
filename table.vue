<template>
    <el-table class="summary-table" :data="summaryData" :header-cell-style="headerCellStyle" :cell-style="cellStyle" :span-method="spanMethod" border>
      <el-table-column prop="key1" label="内容" align="center" width="150"></el-table-column>
      <el-table-column prop="val1" label="结果" align="center"></el-table-column>
      <el-table-column prop="key2" label="内容" align="center" width="150"></el-table-column>
      <el-table-column prop="val2" label="结果" align="center"></el-table-column>
      <el-table-column prop="key3" label="故障信息" align="center" width="150"></el-table-column>
      <el-table-column prop="val3" label="故障结果" align="center" min-width="200"></el-table-column>
    </el-table>
</template>

<script>
  export default {
    data() {
      return {
        summaryData:[
          {
            key1: '开始时间',
            val1: '',
            key2: '是否欠费',
            val2: '',
            key3: '故障层',
            val3: ''
          },
          {
            key1: '结束时间',
            val1: '',
            key2: '计费模式',
            val2: '',
            key3: '故障原因',
            val3: ''
          },
          {
            key1: 'IMSI',
            val1: '',
            key2: '是否行卡',
            val2: '',
            key3: '故障结论及建议',
            val3: ''
          },
          {
            key1: '电话号码',
            val1: '',
            key2: '是否开通',
            val2: '',
            key3: '故障结论及建议',
            val3: ''
          }
        ]
      }
    },
    methods: {
      headerCellStyle() {
        return 'background-color: #eef0f5 !important;'
      },
      cellStyle({row, column, rowIndex, columnIndex}) {
        if (column.property === 'key1' || column.property === 'key2' || column.property === 'key3') {
          return 'background-color: #eef0f5;'
        } else {
          return 'background-color: #fff;'
        }
      },
      spanMethod({row, column, rowIndex, columnIndex}) {
        if (column.property === 'key3' || column.property === 'val3') {
          if (rowIndex === 2) {
            return {
              rowspan: 2,
              colspan: 1
            }
          }
          if (rowIndex === 3) {
            return {
              rowspan: 0,
              colspan: 0
            }
          }
        }
      }
    }
  }
</script>