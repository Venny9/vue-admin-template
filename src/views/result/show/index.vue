<template>
  <div class="app-container">
    <div class="font box">
      <label>风险评估结果</label>
      <span v-if="score >= 85" class="level green">低</span>
      <span v-else-if="score >= 60" class="level yellow">中</span>
      <span v-else class="level red">高</span>
    </div>
    <div class="box">
      <el-table
        :data="tableData"
        border
        :show-header="false"
        style="width: 100%"
        :highlight-current-row="true"
      >
        <el-table-column prop="key" width="200" />
        <el-table-column prop="value">
          <template slot-scope="scope">
            <label v-if="scope.row.key == '影响调用链百分比'">
              {{ scope.row.value }}
              <el-link type="primary" class="link">(详情)</el-link>
            </label>
            <label v-else>{{ scope.row.value }}</label>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="font box">
      <label> 调用链视图类型 </label>
    </div>
    <el-tabs
      type="card"
      class="font box"
      style="padding: 0px"
      @tab-click="handleClick"
    >
      <el-tab-pane label="改动视图">
        <Graph ref="partGraph" idkey="part" :url="parturl" />
      </el-tab-pane>
      <el-tab-pane label="全局视图">
        <Graph ref="allGraph" idkey="all" :url="allurl" :init="false" />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script>
import Graph from '@/components/Graph'
import axios from 'axios'

export default {
  components: {
    Graph
  },
  data() {
    return {
      id: '', // 拼接到url中
      firstInit: true,
      score: 20,
      parturl: '',
      allurl: '',
      riskurl: '',
      tableData: null
    }
  },
  created() {
    this.id = this.$route.query.id
    this.parturl = 'http://9.86.69.48:8081/modifygraph?projectid=' + this.id + '&gitversion=&branchname='
    this.allurl = 'http://9.86.69.48:8081/callgraph?projectid=' + this.id + '&gitversion=&branchname='
    this.riskurl = 'http://9.86.69.48:8081/getrisk?projectid=' + this.id + '&gitversion&branchname='
  },
  mounted() {
    axios.get(this.riskurl).then(response => {
      console.log(response.data)
      this.tableData = response.data.data
    }).catch(error => console.log(error))
  },
  methods: {
    handleClick(tab, event) {
      if (tab.label === '全局视图' && this.firstInit) {
        this.firstInit = false
        this.$refs.allGraph.beginGraph()
      }
    }
  }
}
</script>

<style>
.app-container {
  min-height: calc(100vh - 50px);
  overflow: auto;
  background-color: rgba(253, 254, 255);
}
.font {
  text-align: left;
  font-size: 18px;
}
.box {
  width: 1350px;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  padding: 18px 20px;
  margin: 12px;
  background-color: #fff;
}
.el-tabs--card > .el-tabs__content {
  padding: 23px;
}
.el-tabs__content {
  height: 850px;
}
.level {
  color: #fff;
  margin: 5px;
  height: 24px;
  padding: 0 8px;
  line-height: 22px;
  font-size: 20px;
  border-width: 1px;
  border-style: solid;
  border-radius: 2px;
  box-sizing: border-box;
  white-space: nowrap;
}
.el-table .cell {
  white-space: pre-line;
}
.link {
  vertical-align: baseline;
}
.yellow {
  background: #fac858;
}
.green {
  background: #41b584;
}
.red {
  background: #f34d37;
}
</style>
