<template>
  <div class="down">
    <div id="mountNode" class="mountNode" />
    <div class="searchlist">
      <div class="line">
        <label>在当前视图搜索</label>
      </div>
      <div class="line">
        <label for="class"> 类 名 </label>
        <input id="class" type="text" class="search" placeholder="选填" />
      </div>
      <div class="line">
        <label for="method"> 函 数 </label>
        <input id="method" type="text" class="search" placeholder="必填" />
      </div>
      <div class="line">
        <button class="gradient" style="margin-left: 100px">清空</button>
        <button class="gradient blue" style="margin-left: 15px">搜索</button>
      </div>
    </div>
    <div id="message" class="message">
      <p>message</p>
      <p>id: {{ node.id }}</p>
      <p>name: {{ node.method }}</p>
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
    }
  },
  data() {
    return {
      graph: null,
      graphData: null,
      node: {
        id: '',
        method: ''
      }
    }
  },
  mounted() {
    const minimap = new G6.Minimap({
      container: 'mountNode',
      size: [200, 160],
      className: 'minimap',
      type: 'default'
    })
    const toolbar = new G6.ToolBar({
      container: 'mountNode'
    })
    this.graph = new G6.Graph({
      container: 'mountNode',
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
          'activate-relations',
          {
            type: 'tooltip',
            formatText(model) {
              return model.method
            },
            offset: 2
          }
        ]
      },
      plugins: [minimap, toolbar]
    })
    axios.get(this.url).then(response => {
      this.graphData = response.data
      // 修改label字段
      this.graphData.nodes.forEach((node) => {
        node.label = fittingString(node.label, 40, 12)
      })
      // 修改节点颜色
      /* this.graphData.nodes.forEach((i) => {
        i.style = Object.assign(i.style || {}, {
          fill: '#F79767'
        })
      }) */
      this.graph.read(this.graphData)
      this.graph.on('node:click', evt => {
        this.node.id = evt.item.getModel().id
        this.node.method = evt.item.getModel().method
      })
    }).catch(error => console.log(error))
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
