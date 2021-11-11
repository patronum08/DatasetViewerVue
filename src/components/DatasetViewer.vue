<template>
  <div>
    <el-container>
      <el-header>
        <el-row :gutter="20">
          <el-col :span="6">
            <div class="grid-content bg-purple">
              数据集：
              <el-select v-model="datasetname" placeholder="选择一个数据集">
                <el-option
                    v-for="item in options"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                </el-option>
              </el-select>
            </div>
          </el-col>
          <el-col :span="6">
            <div class="grid-content bg-purple">
              <div class="block">
                ID:
                <el-cascader v-model="currentId"
                             :placeholder="currentId"
                             :options="dataIds"
                             filterable
                             @change="handleChange"></el-cascader>
              </div>
            </div>
          </el-col>
          <el-col :span="6">
            <div class="grid-content bg-purple">
              <div class="block">
                <el-row>
                  <el-button type="success" v-on:click="getRandomId()">Random Next</el-button>
                </el-row>
              </div>
            </div>
          </el-col>
        </el-row>
      </el-header>

      <el-container>
        <el-aside width="900px">
          <template class="myBox">
            <el-table
                :data="textTable"
                style="width: 100%"
                stripe
                border>>
              <el-table-column
                  prop="text"
                  label="text"
                  width="900">
              </el-table-column>
            </el-table>
          </template>
        </el-aside>
        <el-container>
          <el-main>
            <template>
              Question, Answer and Program:
              <ul id="example-1" align="left">
                <li>
                  Q: {{ question }}
                </li>
                <li>
                  A: {{ answer }}
                </li>
                <li>
                  P: {{ steps }}
                </li>
              </ul>
              Gold Texts:
              <ul id="example-2" align="left">
                <li v-for="sentence in goldTexts" :key="sentence">
                  {{ sentence }}
                </li>
              </ul>
              Table:
              <el-table
                  :data="tableData"
                  style="width: 100%"
                  height="800"
                  :row-class-name="tableRowClassName">>
                <el-table-column v-for="header in headers" :key="header"
                                 :prop="header"
                                 :label="header"
                                 width="180">
                </el-table-column>
              </el-table>
            </template>
          </el-main>
        </el-container>
      </el-container>
    </el-container>

  </div>
</template>

<script>
import Vue from 'vue';
import Element from 'element-ui';
import axios from 'axios'
import GLOBAL from '@/components/global_variable'

Vue.use(Element, {size: 'small', zIndex: 3000});

export default {
  name: 'DatasetViewer',
  props: {
    msg: String
  },
  data() {
    return {
      question: '',
      answer: '',
      steps: '',
      currentId: '',
      headers: [],
      tableData: [],
      textTable: [],
      allTexts: [],
      goldTexts: [],
      goldRows: [],
      options: [{
        value: '数据集1',
        label: 'FinQA'
      },],
      datasetname: 'FinQA',
      filterText: '',
      dataIds: [],
      defaultProps: {
        children: 'children',
        label: 'label'
      },
    };
  },

  watch: {
    filterText(val) {
      this.$refs.tree.filter(val);
    },
    currentId: async function (newId) {
      await this.getAllTexts()
      await this.getTable()
      console.log(newId)
    }
  },

  mounted: async function () {
    await this.getDataIds()
    await this.getRandomId()
    await this.getTable()
  },


  methods: {
    filterNode(value, data) {
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },
    async getRandomId() {
      var newId = ''
      console.log(GLOBAL.DefaultIP + '/getRandomId')
      await axios.get(GLOBAL.DefaultIP + '/getRandomId', {})
          .then((response) => {
            console.log(response.data)
            newId = response.data
          })
          .catch((error) => {
            alert(error)
          })
      console.log(newId)
      this.currentId = newId
    },
    async getDataIds() {
      var _this = this
      console.log(GLOBAL.DefaultIP + '/getAllIds')
      await axios.get(GLOBAL.DefaultIP + '/getAllIds')
          .then((response) => {
            console.log(response.data)
            _this.dataIds = response.data
          })
          .catch((error) => {
            alert(error)
          })
    },
    handleChange(value) {
      console.log(value);
      this.currentId = value;
    },
    async handelTableData(table) {
      var _tableData = []
      for (let i = 1; i < table.length; i++) {
        var row = {};
        this.headers = []
        for (let j = 0; j < table[0].length; j++) {
          row[table[0][j]] = table[i][j]
          this.headers.push(table[0][j])
        }
        _tableData.push(row)
      }
      this.rawTableData = table
      this.tableData = _tableData
    },
    async getTable() {
      var _this = this
      console.log(GLOBAL.DefaultIP + '/getTable')
      await axios.get(GLOBAL.DefaultIP + '/getTable', {
        params: {
          id: _this.currentId,
          dataType: 'all'
        }
      })
          .then((response) => {
            console.log(response.data)
            _this.handelTableData(response.data)
          })
          .catch((error) => {
            alert(error)
          })
      await this.getAllTexts()
    },
    async getAllTexts() {
      var _this = this
      this.allTexts = []
      console.log(GLOBAL.DefaultIP + '/getTextsById')
      await axios.get(GLOBAL.DefaultIP + '/getTextsById', {
        params: {
          id: _this.currentId,
        }
      })
          .then((response) => {
            console.log(response.data)
            _this.allTexts = response.data
          })
          .catch((error) => {
            alert(error)
          })

      console.log(GLOBAL.DefaultIP + '/getGoldTextsById')
      await axios.get(GLOBAL.DefaultIP + '/getGoldTextsById', {
        params: {
          id: _this.currentId,
        }
      })
          .then((response) => {
            console.log(response.data)
            _this.goldTexts = response.data
          })
          .catch((error) => {
            alert(error)
          })

      this.textTable = []
      for (var i = 0; i < this.allTexts.length; i++) {
        this.textTable.push({'text': this.allTexts[i]})
        console.log(this.allTexts[i])
      }

      console.log(GLOBAL.DefaultIP + '/getGoldRowsById')
      await axios.get(GLOBAL.DefaultIP + '/getGoldRowsById', {
        params: {
          id: _this.currentId,
        }
      })
          .then((response) => {
            console.log(response.data)
            _this.goldRows = response.data
          })
          .catch((error) => {
            alert(error)
          })

      console.log(GLOBAL.DefaultIP + '/getQuestionAndAnswer')
      await axios.get(GLOBAL.DefaultIP + '/getQuestionAndAnswer', {
        params: {
          id: _this.currentId,
        }
      })
          .then((response) => {
            console.log(response.data)
            _this.question = response.data[0]
            _this.answer = response.data[1]
          })
          .catch((error) => {
            alert(error)
          })

      console.log(GLOBAL.DefaultIP + '/getSteps')
      await axios.get(GLOBAL.DefaultIP + '/getSteps', {
        params: {
          id: _this.currentId,
        }
      })
          .then((response) => {
            console.log(response.data)
            _this.steps = response.data
          })
          .catch((error) => {
            alert(error)
          })
    },
    tableRowClassName({row, rowIndex}) {
      if (this.goldRows.includes(rowIndex + 1)) {
        return 'success-row';
      }
      console.log(row)
      return '';
    },
    textTableRowClassName({row, rowIndex}) {
      if (this.goldTexts.indexOf(this.allTexts[rowIndex]) >= 0) {
        return 'success-row';
      }
      console.log('row index:')
      console.log(rowIndex)
      console.log(this.goldTexts)
      console.log(this.allTexts[rowIndex])
      console.log(row)
      return '';
    },

  },
}

</script>


<style>
.el-row {
  margin-bottom: 20px;

&
:last-child {
  margin-bottom: 0;
}

}
.el-col {
  border-radius: 4px;
}

.bg-purple-dark {
  background: #99a9bf;
}

.bg-purple {
  background: #d3dce6;
}

.bg-purple-light {
  background: #e5e9f2;
}

.grid-content {
  border-radius: 4px;
  min-height: 36px;
}

.row-bg {
  padding: 10px 0;
  background-color: #f9fafc;
}

.el-header, .el-footer {
  background-color: #B3C0D1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  background-color: #ffffff;
  color: #ffffff;
  text-align: left;
  line-height: 20px;
}

.el-main {
  background-color: #ffffff;
  color: #333;
  text-align: left;
  line-height: 25px;
}

body > .el-container {
  margin-bottom: 40px;
}

.el-container:nth-child(5) .el-aside,
.el-container:nth-child(6) .el-aside {
  line-height: 260px;
}

.el-container:nth-child(7) .el-aside {
  line-height: 320px;
}

.el-table .warning-row {
  background: oldlace;
}

.el-table .success-row {
  background: #f0f9eb;
}

body {
  margin-bottom: 200%;
}

/* Box styles */
.myBox {
  border: none;
  padding: 5px;
  font: 24px/36px sans-serif;
  width: 200px;
  height: 800px;
  overflow: scroll;
}

/* Scrollbar styles */
::-webkit-scrollbar {
  width: 12px;
  height: 12px;
}

::-webkit-scrollbar-track {
  background: #f5f5f5;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background: #ccc;
}

::-webkit-scrollbar-thumb:hover {
  background: #999;
}
</style>
