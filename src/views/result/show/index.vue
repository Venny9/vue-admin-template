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
    <!-- <div class="font box" style="height: 950px"> -->
    <el-tabs
      type="border-card"
      class="font box"
      style="padding: 0px"
      @tab-click="handleClick"
    >
      <el-tab-pane label="改动视图">
        <div class="down">
          <div id="mountNode1" class="mountNode" />
          <div class="searchlist">
            <div class="line">
              <label>子视图搜索</label>
            </div>
            <div class="line">
              <label for="class"> 类 名 </label>
              <input
                id="class1"
                type="text"
                class="search"
                placeholder="选填"
              />
            </div>
            <div class="line">
              <label for="method"> 函 数 </label>
              <input
                id="method1"
                type="text"
                class="search"
                placeholder="必填"
              />
            </div>
            <div class="line">
              <button class="gradient" style="margin-left: 100px">清空</button>
              <button class="gradient blue" style="margin-left: 15px">
                搜索
              </button>
            </div>
          </div>
          <div id="message1" class="message">
            <p>message</p>
            <p>id: {{ node1.id }}</p>
            <p>name: {{ node1.method }}</p>
            <p>class: {{ node1.class }}</p>
            <p>parameters: {{ node1.parameters }}</p>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="全局视图">
        <div class="down">
          <div id="mountNode2" class="mountNode" />
          <div class="searchlist">
            <div class="line">
              <label>子视图搜索</label>
            </div>
            <div class="line">
              <label for="class"> 类 名 </label>
              <input
                id="class2"
                type="text"
                class="search"
                placeholder="选填"
              />
            </div>
            <div class="line">
              <label for="method"> 函 数 </label>
              <input
                id="method2"
                type="text"
                class="search"
                placeholder="必填"
              />
            </div>
            <div class="line">
              <button class="gradient" style="margin-left: 100px">清空</button>
              <button class="gradient blue" style="margin-left: 15px">
                搜索
              </button>
            </div>
          </div>
          <div id="message2" class="message">
            <p>message</p>
            <p>id: {{ node2.id }}</p>
            <p>name: {{ node2.method }}</p>
            <p>class: {{ node2.class }}</p>
            <p>parameters: {{ node2.parameters }}</p>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
    <!-- <p>
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
      </keep-alive>  -->
    <!-- </div> -->
  </div>
</template>
<script>
import G6 from '@antv/g6'
import axios from 'axios'

/**
  * 计算字符串的长度
  * @param {string} str 指定的字符串
  */
const calcStrLen = (str) => {
  var len = 0
  for (var i = 0; i < str.length; i++) {
    if (str.charCodeAt(i) > 0 && str.charCodeAt(i) < 128) {
      len++
    } else {
      len += 2
    }
  }
  return len
}
/**
 * 计算显示的字符串
 * @param {string} str 要裁剪的字符串
 * @param {number} maxWidth 最大宽度
 * @param {number} fontSize 字体大小
 */
const fittingString = (str, maxWidth, fontSize) => {
  if (str.length < 8) {
    return str
  }
  var fontWidth = fontSize * 1.3 // 字号+边距
  maxWidth = maxWidth * 2 // 需要根据自己项目调整
  var width = calcStrLen(str) * fontWidth
  var ellipsis = '…'
  if (width > maxWidth) {
    var actualLen = Math.floor((maxWidth - 10) / fontWidth)
    var result = str.substring(0, actualLen) + ellipsis
    return result
  }
  return str
}
// {"method":"modifyClaimTypeToImpressionAttribution",
// "isChanged":true,
// "id":"47387",
// "label":"modifyClaimTypeToImpressionAttribution",
// "class":"com.qq.gdt.data.action.util.adapter.UserActionProtoAdapter",
// "parameters":"com.qq.gdt.data.action.handler.action.vo.request.Action"}

export default {
  data() {
    return {
      id: '', // 拼接到url中
      firstInit: true,
      graph1: null,
      graphData1: null,
      node1: {
        id: '',
        method: '',
        class: '',
        parameters: ''
      },
      graph2: null,
      graphData2: null,
      node2: {
        id: '',
        method: '',
        class: '',
        parameters: ''
      },
      score: 20,
      parturl: 'http://9.86.69.48:8081/modifygraph?projectid=48586&gitversion=&branchname=',
      allurl: 'http://9.86.69.48:8081/callgraph?projectid=48586&gitversion=&branchname=',
      tableData: [{
        key: 'Git分支号',
        value: 'feature/20210323-dpa-update-proto'
      }, {
        key: 'Git修订版',
        value: 'ad8550fc1b3401a69ce0117dbb19b91da80c5c86'
      }, {
        key: '修改函数',
        value: 'setConversionClaim\nmodifyClaimTypeToImpressionAttribution\nupdateImpressionAttributionWhiteList\nschedulerUpdateConfig'
      }, {
        key: '影响调用链百分比',
        value: '2%'
      }, {
        key: '影响接口',
        value: 'action/add\naction/addvenny'
      }, {
        key: '调用链覆盖率',
        value: '暂无'
      }, {
        key: '关联历史bug数',
        value: '暂无'
      }]
    }
  },
  mounted() {
    this.id = this.$route.query.id
    const minimap1 = new G6.Minimap({
      container: 'mountNode1',
      size: [200, 160],
      className: 'minimap',
      type: 'default'
    })
    const toolbar1 = new G6.ToolBar({
      container: 'mountNode1'
    })
    this.graph1 = new G6.Graph({
      container: 'mountNode1',
      width: 1000,
      height: 800,
      enabledStack: true,
      layout: {
        type: 'force',
        preventOverlap: true,
        onLayoutEnd: () => {
          console.log('force layout done')
        }
      },
      defaultNode: {
        labelCfg: {
          position: 'center',
          style: {
            fontSize: 6
          }
        }
      },
      defaultEdge: {
        style: {
          endArrow: true
        },
        labelCfg: {
          autoRotate: true
        }
      },
      modes: {
        default: [
          'zoom-canvas',
          'drag-canvas',
          // 'drag-node',
          // 'activate-relations',
          {
            type: 'tooltip',
            formatText(model) {
              return model.method
            },
            offset: 2
          }
        ]
      },
      plugins: [minimap1, toolbar1]
    })
    axios.get(this.parturl).then(response => {
      this.graphData1 = response.data
      // 修改label字段
      console.log(this.graphData1)
      this.graphData1.nodes.forEach((node) => {
        node.label = fittingString(node.label, 40, 12)
      })
      // 修改节点颜色
      this.graphData1.nodes.forEach((i) => {
        if (i.isChanged) {
          i.style = Object.assign(i.style || {}, {
            fill: '#F79767'
          })
        }
      })
      this.graph1.read(this.graphData1)
      this.graph1.on('node:click', evt => {
        this.node1.id = evt.item.getModel().id
        this.node1.method = evt.item.getModel().method
        this.node1.class = evt.item.getModel().class
        this.node1.parameters = evt.item.getModel().parameters
      })
    }).catch(error => console.log(error))
  },
  methods: {
    handleClick(tab, event) {
      if (tab.label == '全局视图' && this.firstInit) {
        this.beginAllGraph();
      }
    },
    beginAllGraph() {
      const minimap2 = new G6.Minimap({
        container: 'mountNode2',
        size: [200, 160],
        className: 'minimap',
        type: 'default'
      })
      const toolbar2 = new G6.ToolBar({
        container: 'mountNode2'
      })
      this.graph2 = new G6.Graph({
        container: 'mountNode2',
        width: 1000,
        height: 800,
        enabledStack: true,
        layout: {
          type: 'force',
          preventOverlap: true,
          // onTick: () => {
          //   console.log('ticking')
          // },
          onLayoutEnd: () => {
            console.log('force layout done')
          }
        },
        defaultNode: {
          labelCfg: {
            position: 'center',
            style: {
              fontSize: 6
            }
          }
        },
        defaultEdge: {
          style: {
            endArrow: true
          },
          labelCfg: {
            autoRotate: true
          }
        },
        modes: {
          default: [
            'zoom-canvas',
            'drag-canvas',
            // 'drag-node',
            // 'activate-relations',
            {
              type: 'tooltip',
              formatText(model) {
                return model.method
              },
              offset: 2
            }
          ]
        },
        plugins: [minimap2, toolbar2]
      })
      axios.get(this.allurl).then(response => {
        this.graphData2 = response.data
        // 修改label字段
        console.log(this.graphData2)
        this.graphData2.nodes.forEach((node) => {
          node.label = fittingString(node.label, 40, 12)
        })
        // 修改节点颜色
        this.graphData2.nodes.forEach((i) => {
          if (i.isChanged) {
            i.style = Object.assign(i.style || {}, {
              fill: '#F79767'
            })
          }
        })
        this.graph2.read(this.graphData2)
        this.graph2.on('node:click', evt => {
          this.node2.id = evt.item.getModel().id
          this.node2.method = evt.item.getModel().method
          this.node2.class = evt.item.getModel().class
          this.node2.parameters = evt.item.getModel().parameters
        })
      }).catch(error => console.log(error))
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
.el-tabs--border-card > .el-tabs__content {
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
.el-table .cell {
  white-space: pre-line;
}
.link {
  vertical-align: baseline;
}
.down {
  position: relative;
}
.mountNode {
  border-style: solid;
  border-width: 1px;
  border-color: rgba(0, 0, 0, 0.21);
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  position: absolute;
  top: 0px;
  left: 0px;
  width: 1000px;
  height: 800px;
}
.searchlist {
  text-align: center;
  position: absolute;
  top: 0px;
  left: 1004px;
  border-style: solid;
  border-width: 1px;
  border-color: rgba(0, 0, 0, 0.21);
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  width: 300px;
  height: 200px;
  padding: 16px;
}
.line {
  padding: 7px;
}
.search {
  font-size: 16px;
  padding: 4px 8px;
  margin-right: 6px;
  border: 1px solid rgba(0, 0, 0, 0.21);
  border-bottom: 2px solid rgba(0, 0, 0, 0.21);
  border-radius: 4px;
}
.message {
  text-align: left;
  position: absolute;
  top: 206px;
  left: 1004px;
  border-style: solid;
  border-width: 1px;
  border-color: rgba(0, 0, 0, 0.21);
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  width: 300px;
  height: 594px;
  padding: 4px;
  word-wrap: break-word;
}
.g6-tooltip {
  padding: 6px 6px;
  color: #444;
  background-color: rgba(254, 254, 254, 0.9);
  border: 1px solid rgba(0, 0, 0, 0.21);
  border-radius: 4px;
}
.g6-component-toolbar {
  position: absolute;
  top: 0px;
  left: 202px;
  border-style: solid;
  border-width: 1px;
  border-color: rgba(0, 0, 0, 0.21);
}
.minimap {
  border-right: 1px solid rgba(0, 0, 0, 0.21);
  border-bottom: 1px solid rgba(0, 0, 0, 0.21);
  background: #fefefe;
  opacity: 0.9;
  position: absolute;
  top: 0px;
  left: 0px;
}
.g6-minimap-viewport {
  border: 1px solid rgba(0, 0, 0, 0.45);
}
</style>
