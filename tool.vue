<template>
  <div class="app-container">
    <upload-excel-component class="mb20" :on-success="handleSuccess" :before-upload="beforeUpload"/>

    <div class="panel mb15">
      <div class="panelHd">查找条件</div>
      <div class="panelBd">
        <el-radio-group v-model="headerRadio">
          <el-radio class="mb10" v-for="item in tableHeader" :label="item" :key="item"></el-radio>
        </el-radio-group>
        <div>
          <el-input v-model="keyword" placeholder="请输入查找内容" size="medium" clearable style="width:300px;"></el-input>
          <el-button type="primary" icon="el-icon-search" size="medium" @click="handleFilter">查找</el-button>
          <el-button type="primary" icon="el-icon-refresh" size="medium" @click="handleReset">重置</el-button>
        </div>
      </div>
    </div>
    
    <div class="panel mb20">
      <div class="panelHd">导出配置（选择不需要导出的项）</div>
      <div class="panelBd">
        <el-checkbox-group v-model="headerCheckList">
          <el-checkbox class="mb10" v-for="item in tableHeader" :label="item" :key="item"></el-checkbox>
        </el-checkbox-group>
        <div>
          <el-input v-model="filename" placeholder="请输入文件名称" size="medium" clearable style="width:300px;"></el-input>
          <el-button type="primary" icon="el-icon-download" size="medium" :loading="downloadLoading" @click="handleDownload">导出</el-button>
        </div>
      </div>
    </div>

    <el-table :data="tableData" border highlight-current-row style="width: 100%;margin-top:20px;" ref="multipleTable" @selection-change="handleSelectionChange">
      <el-table-column type="selection" align="center"></el-table-column>
      <el-table-column v-for="item of tableHeader" :prop="item" :label="item" :key="item"></el-table-column>
      <el-table-column fixed="right" label="操作" align="center" width="120">
        <template slot-scope="scope">
          <el-button class="btn-edit" type="text" @click="handleUpdate(scope.row)">修改</el-button>
          <el-button class="btn-delete" type="text" @click="handleDelete(scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog class="manageModal" title="信息修改" :visible.sync="dialog.dialogFormVisible">
      <div class="modalCont">
        <el-scrollbar>
          <el-form class="form-flex row2" :model="dialog.form" ref="dialogForm" label-position="right" label-width="120px">
            <el-form-item :label="item" :prop="item" v-for="item in tableHeader" :key="item">
              <el-input v-model="dialog.form[item]" :placeholder="'请输入系统编号' + item" clearable></el-input>
            </el-form-item>
          </el-form>
        </el-scrollbar>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogClose">取消</el-button>
        <el-button type="primary" @click="dialogSubmit">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
import dateFormat from '@/utils'

export default {
  name: 'UploadExcel',
  components: { UploadExcelComponent },
  data() {
    return {
      origtableData: [],
      tableData: [],
      tableHeader: [],

      headerRadio: '',
      headerCheckList: [],
      keyword: '',

      filename: '',
      multipleSelection: [],
      downloadLoading: false,

      operateRow: '',
      dialog: {
        dialogFormVisible: false,
        form: {}
      },
    }
  },
  methods: {
    beforeUpload(file) {
      const isLt1M = file.size / 1024 / 1024 < 100

      if (isLt1M) {
        return true
      }

      this.$message({
        message: 'Please do not upload files larger than 100m in size.',
        type: 'warning'
      })
      return false
    },
    handleSuccess({ results, header }) {
      // console.log(dateFormat(new Date(), 'yyyy-MM-dd 00:00:00'))
      // console.log(dateFormat(new Date('2018/1/2'), 'yyyy-MM-dd'))
      const resultsFormat = results.map((item,index) => {
        item.id = Date.now() + index
        return item
      })

      this.origtableData = this.tableData = resultsFormat
      this.tableHeader = header
      
      this.headerRadio = this.tableHeader[0]
      console.log(results,header)
      //.includes(this.keyword)
    },
    handleFilter() {
      this.tableData = this.origtableData.filter(todo => {
        console.log(todo,todo[this.headerRadio])
        //todo[this.headerRadio]
        if (todo[this.headerRadio] === undefined) {
          return false
        }
        const str = todo[this.headerRadio]
        // if (str === '') return false
        if (str.indexOf(this.keyword) < 0) {
          return false
        } else {
          return true
        }
      })
      console.log('arr2',this.tableData)
      //console.log(todo,this.headerRadio,todo[this.headerRadio])
      // const arr = this.tableData.filter(todo => todo[this.headerRadio].indexOf(this.keyword) >= 0)

      // console.log(arr)
    },
    handleReset() {
      this.keyword = ''
      this.tableData = this.origtableData
    },

    // 导出
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    handleDownload() {
      // if (this.multipleSelection.length) {
      // } else {
      //   this.$message({
      //     message: 'Please select at least one item',
      //     type: 'warning'
      //   })
      // }

      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        // const tHeader = ['Id', 'Title', 'Author', 'Readings', 'Date']
        // const filterVal = ['id', 'title', 'author', 'pageviews', 'display_time']

        const finalVal = this.headerCheckList.length ? this.tableHeader.filter(item=>!this.headerCheckList.some(ele=>ele===item)) : this.tableHeader
        console.log('this.headerCheckList',this.tableHeader,this.headerCheckList,finalVal)
        const tHeader = finalVal
        const filterVal = finalVal

        const list = this.multipleSelection.length ? this.multipleSelection : this.origtableData
        const data = this.formatJson(filterVal, list)

        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: this.filename
        })
        this.$refs.multipleTable.clearSelection()
        this.downloadLoading = false
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => v[j]))
    },

    // 修改
    handleUpdate(row) {
      this.dialog.form  = row
      this.operateRow = row
      this.dialog.dialogFormVisible = true
    },
    handleDelete(row) {
      this.$confirm('确认要删除这条数据吗, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.origtableData.splice(this.origtableData.findIndex(item => item.id === row.id), 1)
      })
    },

    dialogSubmit() {
      const idx = this.origtableData.findIndex(item => item.id === this.operateRow.id)
      this.origtableData[idx] = this.dialog.form
      this.dialog.dialogFormVisible = false
    },
    dialogClose() {
      this.dialog.dialogFormVisible = false
    }
  }
}
</script>

<style type="text/css" lang="scss">
  $tableThBg: #f4f4f4;
  .el-table {
    width: 100%;
    .el-table__header {
        tr, th {
          background-color: $tableThBg !important;
          transition: background .3s ease;
        }
    }
  }
  .btn-edit {
    color: #409eff;
  }
  .btn-delete {
    color: #f56c6c;
  }

  .form-flex {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    flex-wrap: wrap;

    &.row2 {
      > .el-form-item {
        width: 50%;
      }
    }
    &.row3 {
      > .el-form-item {
        width: 33.333333%;
      }
    }
    &.row4 {
      > .el-form-item {
        width: 25%;
      }
    }
  }

  .el-radio,
  .el-checkbox {
    margin-left: 0 !important;
    margin-right: 30px !important;
  }
</style>
<style type="text/css" lang="scss" scoped>
  .app-container {
    margin: 0 auto;
    width: 80%;
  }
  .mb30 {
    margin-bottom: 30px;
  }
  .mb20 {
    margin-bottom: 20px;
  }
  .mb15 {
    margin-bottom: 15px;
  }
  .mb10 {
    margin-bottom: 10px;
  }

  .panel {
    border: 1px solid #dedede;
    border-radius: 4px;
  }
  .panelHd {
    padding: 0 10px;
    line-height: 40px;
    font-size: 14px;
    color: #000;
    border-bottom: 1px solid #dedede;
  }
  .panelBd {
    padding: 10px;
  }
</style>
