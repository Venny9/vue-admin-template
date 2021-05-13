<template>
  <div class="down">
    <div :id="canvas" class="mountNode" />
    <div class="searchlist">
      <div class="line">
        <label>子视图搜索</label>
      </div>
      <div class="line">
        <label> 类 名 </label>
        <input
          type="text"
          class="search"
          placeholder="选填"
          v-model="searchclass"
        />
      </div>
      <div class="line">
        <label> 函 数 </label>
        <input
          type="text"
          class="search"
          placeholder="必填"
          v-model="searchmethod"
        />
      </div>
      <div class="line">
        <el-button type="primary" @click="empty" style="margin-left: 30px" plain
          >清空</el-button
        >
        <el-button type="primary" :disabled="isdisabledFn" @click="searchGraph"
          >搜索</el-button
        >
      </div>
    </div>
    <div class="message">
      <div class="line" style="text-align: center">
        <label>函数信息</label>
      </div>
      <div class="hint">
        <label>（点击左图节点展示）</label>
      </div>
      <div class="message_line">
        <p>函数名称:</p>
        {{ node.method }}
      </div>
      <div class="message_line">
        <p>所在类名:</p>
        {{ node.class }}
      </div>
      <div class="message_line">
        <p>参数列表:</p>
        <ul>
          <li v-for="(item, index) in node.parameters" :key="index">
            {{ item }}
          </li>
        </ul>
      </div>
    </div>
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

export default {
  name: 'Graph',
  props: {
    url: {
      type: String,
      required: true
    },
    init: {
      type: Boolean,
      default: true
    },
    idkey: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      canvas: null,
      graph: null,
      graphData: null,
      minimap: null,
      toolbar: null,
      searchclass: '',
      searchmethod: '',
      node: {
        id: '',
        method: '',
        class: '',
        parameters: null
      }
    }
  },
  computed: {
    isdisabledFn() {
      if (this.searchmethod !== '') {
        return false
      } else {
        return true
      }
    }
  },
  created() {
    this.canvas = 'mountNode' + this.idkey
  },
  mounted() {
    if (this.init) {
      this.beginGraph(this.url)
    }
  },
  methods: {
    beginGraph(finalurl) {
      console.log(this.canvas)
      this.minimap = new G6.Minimap({
        container: this.canvas,
        size: [200, 160],
        className: 'minimap',
        type: 'default'
      })
      this.toolbar = new G6.ToolBar({
        container: this.canvas
      })
      this.graph = new G6.Graph({
        container: this.canvas,
        width: 1000,
        height: 800,
        enabledStack: true,
        layout: {
          type: 'force',
          preventOverlap: true,
          onTick: () => {
            console.log('ticking')
          },
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
            'drag-node',
            // 'activate-relations',
            {
              type: 'tooltip',
              formatText(model) {
                return model.name
              },
              offset: 4
            }
          ]
        },
        plugins: [this.minimap, this.toolbar]
      })
      this.changeGraph(finalurl)
    },
    changeGraph(finalurl) {
      axios.get(finalurl).then(response => {
        console.log(finalurl)
        this.graphData = response.data
        // 修改label字段
        console.log(this.graphData)
        this.graphData.nodes.forEach((node) => {
          node.name = node.label
          node.label = fittingString(node.label, 40, 12)
        })
        // 修改节点颜色
        this.graphData.nodes.forEach((i) => {
          if (i.isChanged) {
            i.style = Object.assign(i.style || {}, {
              fill: '#F79767'
            })
          }
        })
        this.graph.read(this.graphData)
        this.graph.on('node:click', evt => {
          this.node.id = evt.item.getModel().id
          var getOneFunc = "http://9.86.69.48:8081/getOneFunc?funcid=" + this.node.id
          axios.get(getOneFunc).then(response => {
            if (response.data.errcode === 0) {
              this.node.method = response.data.method
              this.node.class = response.data.class
              if (response.data.parameters.includes(",")) {
                var tmpArray = response.data.parameters.split(',')
                this.node.parameters = tmpArray
                console.log(this.node.parameters)
              } else if (response.data.parameters !== null) {
                this.node.parameters = [response.data.parameters]
              }
            }
          }).catch(error => console.log(error))
        })
      }).catch(error => console.log(error))
    },
    empty() {
      this.searchclass = ''
      this.searchmethod = ''
    },
    searchGraph() {
      console.log('search')
      // TODO if nomethod
      // alert
      // else havemethod
      // 调用父组件
      this.$emit('searchMethodGraph', this.searchmethod)
    }
  }
}
</script>
<style>
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
.message_line {
  padding-left: 7px;
  padding-right: 7px;
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
  padding: 16px;
  word-wrap: break-word;
}
.hint {
  text-align: center;
  font-size: 8px;
  color: #409eff;
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
p {
  font-size: 14px;
  color: rgb(104, 103, 103);
  margin: 2px;
  padding-top: 16px;
  margin-left: 0px;
}
ul {
  margin: 0px;
  padding-left: 20px;
  padding-right: 16px;
}
</style>
