<template>
  <div class="app-container">
    <div class="font box">
      <label>风险评估等级</label>
      <span v-if="score >= 70" class="level red">高风险</span>
      <span v-else-if="score >= 35" class="level yellow">中风险</span>
      <span v-else-if="score >= 0" class="level green">低风险</span>
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
            <span v-if="score >= 70" class="level red">高风险</span>
            <span v-else-if="score >= 35" class="level yellow">中风险</span>
            <span v-else-if="score >= 0" class="level green">低风险</span>
            本次得分 : {{ score }}
          </p>
          <jsCdn
            cdn="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.2/MathJax.js?config=TeX-AMS_HTML"
          />
          <el-collapse v-model="activeNames">
            <el-collapse-item title="模型定义" name="1">
              <div>
                函数调用链视图是基于静态+动态分析获得的为以单个函数为节点，调用关系为边的有向图。
              </div>
              <div>
                风险评估模型是基于函数调用链，并且综合代码质量、业务场景、测试质量、历史问题等，给出的本次代码修改潜在风险评估。
              </div>
              <div>计算公式：</div>
              <vue-mathjax
                formula="$$SQA( SCC_{i}  )= \sum\limits_{j=1}^mSQA( F_{i,j,project} ) \times  ra1 + SQA( Business_{i} ) \times ra2 + SQA( Test_{j} ) \times ra3 $$"
              />
              <vue-mathjax formula=" $$+ SQA( Defect_{i} ) \times ra4 $$" />
              <div>
                其中，SQA(Software quality
                assessment)表示软件质量评估；SCC(Software code
                change)表示软件代码变更；ra*表示风险系数，不同维度风险系数定义不同。
              </div>
            </el-collapse-item>
            <el-collapse-item title="代码质量 Project" name="2">
              <vue-mathjax
                formula=" $$ SQA( F_{j,project} )= \frac{O(Fj)- O_{standard} }{ O_{standard} }  \times  \alpha +  \frac{Outdegree( F_{j} )  \times  Indegree( F_{j} )}{LinksCount}  \times \beta $$"
              />
              <div>代码质量通过函数圈复杂度和函数出度和入度...解释</div>
              <div>本次计算：修改函数 圈复杂度 出度/入度</div>
            </el-collapse-item>
            <el-collapse-item title="业务场景 Business" name="3">
              <vue-mathjax
                formula="$$SQA( Business_{i} )= \frac{ \sum\limits_{k=1}^m( \lg Priority_{k}  ) }{ \sum\limits_{k=1}^n( \lg Priority_{k}  )}$$"
              />
              <div>本次影响的接口 优先级系数</div>
            </el-collapse-item>
            <el-collapse-item title="测试质量 Test" name="4">
              <vue-mathjax
                formula="$$SQA( Test_{i} )= \frac{CoveredLinkscount - Linkscount}{Linkscount}$$"
              />
              <div>本次测试覆盖详情</div>
            </el-collapse-item>
            <el-collapse-item title="历史问题 Defect" name="5">
              <vue-mathjax
                formula="$$SQA( Defect_{i} )=   \frac{ \sum\limits_{k=1}^m Defectcount_{k}  }{\sum\limits_{k=1}^n Defectcount_{k} }$$"
              />
              <div>历史相关调用链出现问题频率</div>
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
      v-model="activeName"
      type="card"
      class="font box"
      style="padding: 0px"
      @tab-click="handleClick"
    >
      <el-tab-pane label="改动视图" name="1">
        <Graph
          ref="partGraph"
          idkey="part"
          :url="parturl"
          :id="id"
          @searchMethodGraph="searchMethodGraph"
        />
      </el-tab-pane>
      <el-tab-pane label="全局视图" name="2">
        <Graph
          ref="allGraph"
          idkey="all"
          :url="allurl"
          :id="id"
          :init="false"
          @searchMethodGraph="searchMethodGraph"
        />
      </el-tab-pane>
      <el-tab-pane label="搜索视图" :disabled="!isSearch" name="3">
        <Graph
          ref="searchGraph"
          idkey="search"
          :id="id"
          :url="searchurl"
          :init="false"
          @searchMethodGraph="searchMethodGraph"
        />
      </el-tab-pane>
    </el-tabs>
  </div>
</template>
<script>
import Graph from '@/components/Graph'
import axios from 'axios'
import { VueMathjax } from 'vue-mathjax'

export default {
  components: {
    Graph,
    'vue-mathjax': VueMathjax,
    'jsCdn': {
      render(_) {
        return _('script', {
          attrs: {
            type: 'text/javascript',
            src: this.cdn
          }
        })
      },
      props: {
        cdn: {
          type: String,
          required: true
        }
      }
    }
  },
  data() {
    return {
      drawer: false,
      activeName: "1",
      activeNames: ['1'],
      id: '', // 拼接到url中
      firstInit: true,
      score: -1,
      parturl: '',
      allurl: '',
      searchurl: '',
      riskurl: '',
      tableData: null,
      isSearch: false
    }
  },
  created() {
    this.id = this.$route.query.id
    this.parturl = 'http://9.86.69.48:8081/modifygraph?projectid=' + this.id + '&gitversion=&branchname='
    this.allurl = 'http://9.86.69.48:8081/callgraph?projectid=' + this.id + '&gitversion=&branchname='
    this.riskurl = 'http://9.86.69.48:8081/getrisk?projectid=' + this.id + '&gitversion&branchname='
    axios.get(this.riskurl).then(response => {
      console.log(response.data)
      this.tableData = response.data.data
      this.score = response.data.riskLevel
    }).catch(error => console.log(error))
  },
  methods: {
    handleClick(tab, event) {
      if (tab.label === '全局视图' && this.firstInit) {
        this.firstInit = false
        this.$refs.allGraph.beginGraph(this.allurl)
      }
    },
    searchMethodGraph(searchurl) {
      this.searchurl = searchurl
      if (!this.isSearch) {
        this.isSearch = true
        // 如果是第一次画，开始画图
        this.$refs.searchGraph.beginGraph(this.searchurl)
      } else {
        // 不是第一次，修改数据
        this.$refs.searchGraph.changeGraph(this.searchurl)
      }
      this.activeName = "3"
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
.el-avatar,
.el-drawer {
  overflow-x: hidden;
  overflow-y: auto;
}
</style>
