<template>
  <div class="app-container">
    <div class="font box">
      <label>风险评估结果</label>
      <span v-if="score >= 85" class="level green">低</span>
      <span v-else-if="score >= 60" class="level yellow">中</span>
      <span v-else class="level red">高</span>
    </div>
    <div class="font box">
      <p>Git分支号 : {{ gitbranch }}</p>
      <p>Git修订版 : {{ gitrevision }}</p>
      <p>修改函数 :
        <ul>
          <li v-for="item in changedMethodList" :key="item">
            {{ item}}
          </li>
        </ul>
      </p>
      <p>影响调用链百分比: 2% <el-link type="info">(详情)</el-link></p>
      <p>影响接口:
        <ul>
          <li v-for="item in affectedAPI" :key="item">
            {{ item}}
          </li>
        </ul>
      </p>
      <p>调用链覆盖率：暂无</p>
      <p>关联历史bug数：暂无</p>
    </div>
    <div class="font box" style="height: 950px">
      <p>
        <label> 调用链视图类型 </label>
        <button
          :class="{ gradient: true, blue: !isAll }"
          style="margin-left: 10px"
          @click="graphType('part')"
        >
          改动视图
        </button>
        <button
          :class="{ gradient: true, blue: isAll }"
          style="margin-left: 2px"
          @click="graphType('all')"
        >
          全局视图
        </button>
      </p>
      <keep-alive>
        <Graph :url="url" />
      </keep-alive>
    </div>
  </div>
</template>
<script>
import Graph from '../../../components/Graph/index'
export default {
  components: {
    Graph
  },
  data() {
    return {
      id: '', // 拼接到url中
      gitrevision: 'ad8550fc1b3401a69ce0117dbb19b91da80c5c86',
      gitbranch: 'feature/20210323-dpa-update-proto',
      changedMethodList: ['setConversionClaim', 'modifyClaimTypeToImpressionAttribution', 'updateImpressionAttributionWhiteList', 'schedulerUpdateConfig'],
      affectedAPI: ['action/add', 'action/addvenny'],
      isAll: false,
      score: 20,
      url: 'http://9.86.69.48:8081/modifygraph'
    }
  },
  mounted() {
    this.id = this.$route.query.id
  },
  methods: {
    graphType(type) {
      if (type === 'all' && this.isAll || type === 'part' && !this.isAll) {
        // do nothing
      } else if (type === 'all' && !this.isAll) {
        this.isAll = true
        console.log('all graph')
      } else if (type === 'part' && this.isAll) {
        this.isAll = false
        console.log('part graph')
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
  font-size: 20px;
}
.box {
  width: 1350px;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  padding: 18px 20px;
  margin: 12px;
  background-color: #fff;
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
.gradient {
  text-decoration: none;
  font-size: 18px;
  padding: 4px 8px;
  display: inline-block;
  position: relative;
  border: 1px solid rgba(0, 0, 0, 0.21);
  border-bottom: 4px solid rgba(0, 0, 0, 0.21);
  border-radius: 4px;
  text-shadow: 0 1px 0 rgba(0, 0, 0, 0.15);
  background: #fff;
  color: #606266;
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
.gradient.blue {
  color: white;
  background: #409eff;
}
.gradient.blue:active {
  color: white;
  background: #2b8beb;
}
</style>
