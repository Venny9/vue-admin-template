<template>
  <div class="app-container">
    <div class="font box">
      <label>风险评估等级</label>
      <span v-if="score >= 85" class="level green">低风险</span>
      <span v-else-if="score >= 60" class="level yellow">中风险</span>
      <span v-else class="level red">高风险</span>
      <el-button type="text" @click="drawer = true"> （详情） </el-button>
      <el-drawer
        title="风险评估模型计算过程"
        :visible.sync="drawer"
        size="50%"
        :with-header="false"
      >
        <div class="box" style="width: 770px">
          <label>风险评估模型</label>
          <p style="font-size: 15px">
            <span v-if="score >= 85" class="level green">低风险</span>
            <span v-else-if="score >= 60" class="level yellow">中风险</span>
            <span v-else class="level red">高风险</span>
            本次得分 : {{ score }}
          </p>
          <el-collapse v-model="activeNames">
            <el-collapse-item title="模型定义" name="1">
              <p>
                与现实生活一致：与现实生活的流程、逻辑保持一致，遵循用户习惯的语言和概念；
              </p>
              <div>
                在界面中一致：所有的元素和结构需保持一致，比如：设计样式、图标和文本、元素的位置等。
              </div>
            </el-collapse-item>
            <el-collapse-item title="project" name="2">
              <div>
                控制反馈：通过界面样式和交互动效让用户可以清晰的感知自己的操作；
              </div>
              <div>
                页面反馈：操作后，通过页面元素的变化清晰地展现当前状态。
              </div>
            </el-collapse-item>
            <el-collapse-item title="business" name="3">
              <div>简化流程：设计简洁直观的操作流程；</div>
              <div>
                清晰明确：语言表达清晰且表意明确，让用户快速理解进而作出决策；
              </div>
              <div>
                帮助用户识别：界面简单直白，让用户快速识别而非回忆，减少用户记忆负担。
              </div>
            </el-collapse-item>
            <el-collapse-item title="test" name="4">
              <div>
                用户决策：根据场景可给予用户操作建议或安全提示，但不能代替用户进行决策；
              </div>
              <div>
                结果可控：用户可以自由的进行操作，包括撤销、回退和终止当前操作等。
              </div>
            </el-collapse-item>
            <el-collapse-item title="defect" name="4">
              <div>
                用户决策：根据场景可给予用户操作建议或安全提示，但不能代替用户进行决策；
              </div>
              <div>
                结果可控：用户可以自由的进行操作，包括撤销、回退和终止当前操作等。
              </div>
            </el-collapse-item>
          </el-collapse>
        </div>
      </el-drawer>
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
            {{ scope.row.value }}
            <!-- <label v-if="scope.row.key == '影响调用链百分比'">
              {{ scope.row.value }}
              <el-link type="primary" class="link">(详情)</el-link>
            </label>
            <label v-else>{{ scope.row.value }}</label> -->
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
      drawer: false,
      activeNames: ['1'],
      id: '', // 拼接到url中
      firstInit: true,
      score: 90,
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
  font-size: 18px;
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
.el-collapse-item__header {
  font-size: 18px;
}
.el-collapse-item__content {
  font-size: 15px;
}
</style>
