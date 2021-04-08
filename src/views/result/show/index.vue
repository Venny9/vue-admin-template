<template>
  <div class="app-container">
    <h1>评估结果</h1>
    <hr />
    <div class="details">
      <p>git revision : {{ gitrevision }}</p>
      <p>
        风险评估结果:
        <span style="color: green">低</span>
        <span style="color: yellow">中</span>
        <span style="color: red">高</span>
      </p>
      <p>修改函数 :</p>
      <p>影响调用链个数: (详情)</p>
      <p>影响接口:</p>
      <p>调用链覆盖率：</p>
      <p>关联历史bug数：</p>
      <hr />
      <p>
        <label> 调用链视图类型 </label>
        <button
          :class="{ gradient: true, yellow: !isAll, green: isAll }"
          style="margin-left: 10px"
          @click="graphType('part')"
        >
          改动视图
        </button>
        <button
          :class="{ gradient: true, yellow: isAll, green: !isAll }"
          style="margin-left: 2px"
          @click="graphType('all')"
        >
          全局视图
        </button>
      </p>
    </div>
    <keep-alive>
      <Graph :url="url" />
    </keep-alive>
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
      gitrevision: '',
      isAll: false,
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
}
.details {
  text-align: left;
  font-size: 20px;
}
h1 {
  text-align: left;
}
.gradient {
  text-decoration: none;
  color: white;
  font-size: 18px;
  padding: 4px 8px;
  display: inline-block;
  position: relative;
  border: 1px solid rgba(0, 0, 0, 0.21);
  border-bottom: 4px solid rgba(0, 0, 0, 0.21);
  border-radius: 4px;
  text-shadow: 0 1px 0 rgba(0, 0, 0, 0.15);
}
.yellow {
  background: rgba(240, 210, 100, 1);
}
.green {
  background: rgba(130, 200, 160, 1);
}
.gradient.blue {
  background: rgba(102, 152, 203, 1);
}
.gradient.blue:active {
  background: #608fbf;
}
</style>
