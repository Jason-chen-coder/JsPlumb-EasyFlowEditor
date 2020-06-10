<template>
  <div v-if="easyFlowVisible" style="height: calc(100vh);">
    <el-row>
      <!--顶部工具菜单-->
      <el-col :span="24">
        <div class="ef-tooltar">
          <el-link type="primary" :underline="false">Easy Editor</el-link>
          <el-divider direction="vertical"></el-divider>
          <el-link type="primary" :underline="false">{{data.name}}</el-link>
          <el-divider direction="vertical"></el-divider>
          <el-button
            type="text"
            icon="el-icon-delete"
            size="large"
            @click="deleteElement"
            :disabled="!this.activeElement.type"
          ></el-button>
          <el-divider direction="vertical"></el-divider>
          <el-button type="text" icon="el-icon-download" size="large" @click="downloadData"></el-button>
          <div style="float: right;margin-right: 5px">
            <!-- 选择节点连接线样式 -->
            <el-select @change="selectLineStyle" v-model="lineValue" placeholder="节点连接线样式">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
            <el-button plain round icon="el-icon-document" @click="dataInfo" size="mini">流程信息</el-button>
            <el-button plain round @click="dataReloadA" icon="el-icon-refresh" size="mini">切换流程A</el-button>
            <el-button plain round @click="dataReloadB" icon="el-icon-refresh" size="mini">切换流程B</el-button>
            <el-button plain round @click="dataReloadC" icon="el-icon-refresh" size="mini">切换流程C</el-button>
          </div>
        </div>
      </el-col>
    </el-row>
    <!-- 工具栏下方区域 -->
    <div style="display: flex;height: calc(100% - 47px);">
      <!-- 左侧节点实例 -->
      <div style="width: 230px;border-right: 1px solid #dce3e8;">
        <node-menu @addNode="addNode" ref="nodeMenu"></node-menu>
      </div>
      <!-- 放大缩小工具 -->
      <div class="zoom-tool" @mousedown.stop>
        <i class="el-icon-plus enlarged-btn"></i>
        <el-slider
          @dblclick="scaleValue=1"
          @input="startScale"
          :step="0.005"
          :max="1.5"
          v-model="scaleValue"
          vertical
          height="90px"
        ></el-slider>
        <i class="el-icon-minus shrink-btn"></i>
      </div>
      <!-- 中间画布部分 -->
      <div class="center-container">
        <div
          id="efContainer"
          @mousewheel.stop="changeZoom($event)"
          ref="efContainer"
          class="container"
          v-flowDrag
        >
          <div class="all-nodes" ref="allnodes">
            <template v-for="node in data.nodeList">
              <!-- 每个节点元素 -->
              <flow-node
                :id="node.id"
                :key="node.id"
                :node="node"
                :activeElement="activeElement"
                @changeNodeSite="changeNodeSite"
                @nodeRightMenu="nodeRightMenu"
                @clickNode="clickNode"
              ></flow-node>
            </template>
          </div>
        </div>
      </div>
      <!-- 右侧表单 -->
      <div :class="{'node-form-active':isNodeFormActive}" class="node-detail">
        <flow-node-form
          ref="nodeForm"
          @changeNodeForm="changeNodeForm"
          @setLineLabel="setLineLabel"
          @changeNodeSite="changeNodeSite"
        ></flow-node-form>
      </div>
    </div>
    <!-- 流程数据详情 -->
    <flow-info v-if="flowInfoVisible" ref="flowInfo" :data="data"></flow-info>
  </div>
</template>

<script>
// import draggable from 'vuedraggable'
import { jsPlumb } from 'jsplumb'
// 使用修改后的jsplumb
// import './jsplumb'
import flowNode from '@/components/node'
import nodeMenu from '@/components/node_menu'
import FlowInfo from '@/components/info'
import FlowNodeForm from './node_form'
import lodash from 'lodash'
import { data_A } from '../data/data_A'
import { data_B } from '../data/data_B'
import { data_C } from '../data/data_C'

export default {
  data () {
    return {
      // jsPlumb 实例
      jsPlumb: null,
      // 控制画布销毁
      easyFlowVisible: true,
      // 控制流程数据显示与隐藏
      flowInfoVisible: false,
      // 是否加载完毕标志位
      loadEasyFlowFinish: false,
      // 数据
      data: {},
      // 激活的元素、可能是节点、可能是连线
      activeElement: {
        // 可选值 node 、line
        type: undefined,
        // 节点ID
        nodeId: undefined,
        // 连线ID
        sourceId: undefined,
        targetId: undefined
      },
      // 基础配置
      jsplumbSetting: {
        // 动态锚点、位置自适应
        Anchors: ['Top', 'TopCenter', 'TopRight', 'TopLeft', 'Right', 'RightMiddle', 'Bottom', 'BottomCenter', 'BottomRight', 'BottomLeft', 'Left', 'LeftMiddle'],
        Container: 'efContainer',
        // 连线的样式 StateMachine、Flowchart，Bezier、Straight
        Connector: ['Flowchart', { curviness: 50 }],
        // Connector: ['Bezier', { stub: [0, 0], gap: 1, cornerRadius: 50, alwaysRespectStubs: true }],
        // 鼠标不能拖动删除线
        ConnectionsDetachable: false,
        // 删除线的时候节点不删除
        DeleteEndpointsOnDetach: false,
        // 连线的端点：圆形
        // Endpoint: ['Dot', {radius: 3}],
        // 连线的端点：矩形
        Endpoint: ['Rectangle', { height: 10, width: 10 }],
        // 线端点的样式
        EndpointStyle: { fill: '', outlineWidth: 1 },
        // 是否打开jsPlumb的内部日志记录
        LogEnabled: true,
        // 绘制连线
        PaintStyle: {
          stroke: '#000000',
          strokeWidth: 1,
          outlineStroke: 'transparent',
          // 设定线外边的宽，单位px
          outlineWidth: 10
        },
        // 绘制连线箭头
        Overlays: [
          ['Arrow', { width: 10, length: 8, location: 1, foldback: 0.8 }],
          ['Label', { location: 0.1, id: 'label', cssClass: 'aLabel' }]
        ],
        // 绘制图的模式 svg、canvas
        RenderMode: 'svg',
        DragOptions: {
          cursor: 'pointer',
          zIndex: 2000,
          hoverClass: 'dropHover',
          activeClass: 'dragActive'
        },
        // 鼠标滑过线的样式
        HoverPaintStyle: { stroke: 'orange', strokeWidth: 3, cursor: 'pointer' },
        // 滑过锚点效果
        // EndpointHoverStyle: { fill: 'red' }
      },
      // jsplumb连接参数
      jsplumbConnectOptions: {
        isSource: true,
        isTarget: true,
        // 动态锚点、提供了4个方向 Continuous、AutoDefault
        anchor: 'Continuous',
        // 设置连线上面的label样式
        labelStyle: {
          cssClass: 'flowLabel'
        },
        // 修改了jsplumb 源码，支持label 为空传入自定义style
        emptyLabelStyle: {
          cssClass: 'emptyFlowLabel'
        }
      },
      jsplumbSourceOptions: {
        /* "span"表示标签，".className"表示类，"#id"表示元素id */
        filter: '.flow-node-drag',
        filterExclude: false,
        anchor: 'Continuous',
        allowLoopback: false
      },
      jsplumbTargetOptions: {
        // "span"表示标签，".className"表示类，"#id"表示元素id
        filter: '.flow-node-drag',
        filterExclude: false,
        anchor: 'Continuous',
        allowLoopback: false
      },
      isNodeFormActive: false,
      // 连接线的样式选项
      options: [{
        value: '["StateMachine", { curviness: 50 }]',
        label: 'StateMachine'
      }, {
        value: '["Flowchart", { curviness: 50 }]',
        label: 'Flowchart'
      }, {
        value: '["Bezier", { stub: [0, 0], gap: 1, cornerRadius: 50, alwaysRespectStubs: true }]',
        label: 'Bezier'
      }, {
        value: '["Straight", { curviness: 50 }]',
        label: 'Straight'
      }],
      // 连接线的样式值
      lineValue: '',
      // 当前的数据
      currentData: {},
      scaleValue: 1,
    }
  },
  components: {
    // draggable, 
    flowNode, nodeMenu, FlowInfo, FlowNodeForm
  },
  directives: {
    'flowDrag': {
      bind (el, binding, vnode, oldNode) {
        console.log(vnode, oldNode)
        if (!binding) {
          return
        }
        el.onmousedown = (e) => {
          if (e.button == 2) {
            // 右键不管
            return
          }
          //  鼠标按下，计算当前原始距离可视区的高度
          let disX = e.clientX
          let disY = e.clientY
          el.style.cursor = 'move'

          document.onmousemove = function (e) {
            // 移动时禁止默认事件
            e.preventDefault()
            const left = e.clientX - disX
            disX = e.clientX
            el.scrollLeft += -left

            const top = e.clientY - disY
            disY = e.clientY
            el.scrollTop += -top
          }

          document.onmouseup = function () {
            el.style.cursor = 'auto'
            document.onmousemove = null
            document.onmouseup = null
          }
        }
      }
    }
  },
  mounted () {
    // 实例化,生成一个jsPlumb实例
    this.jsPlumb = jsPlumb.getInstance()
    this.$nextTick(() => {
      // 默认加载流程A的数据、在这里可以根据具体的业务返回符合流程数据格式的数据即可
      this.currentData = data_B
      this.dataReload(this.currentData)
    })
  },
  methods: {
    selectLineStyle () {
      console.log(eval(this.lineValue))
      this.jsplumbSetting.Connector = eval(this.lineValue)
      this.dataReload(this.currentData)
    },
    // // 开始放大缩小
    startScale () {
      this.zoom(this.scaleValue)
    }
    ,
    zoom (scale) {
      this.$refs.efContainer.style.transform = `scale(${scale})`
      this.jsPlumb.setZoom(0.75);
    },
    changeZoom () {
      console.log(1)
      // var _this = this;
      var canvasDom = document.getElementById('efContainer');
      // var mainDom = document.querySelector('.center-container');

      var p = ["webkit", "moz", "ms", "o"];
      window.addEventListener("mousewheel DOMMouseScroll", function (e) {
        var delta = (e.wheelDelta && (e.wheelDelta > 0 ? 1 : -1)) ||  // chrome & ie &其它  
          (e.detail && (e.detail > 0 ? -1 : 1));// firefox
        var originX = (e.clientX - 353 - 20) / 50 * 100;
        var originY = (e.clientY - 40 - 20) / 50 * 100;
        var oString = originX + "% " + originY + "%";
        if (delta > 0) {
          // 向上滚  
          for (var i = 0; i < p.length; i++) {
            canvasDom.style[p[i] + "Transform"] = "scale(" + this.zoomNum + ")";
            canvasDom.style[p[i] + "TransformOrigin"] = oString;
          }

          canvasDom.style["transform"] = "scale(" + this.zoomNum + ")";
          canvasDom.style["transformOrigin"] = oString;
          this.zoomNum += 0.1;
        } else if (delta < 0) {
          // 向下滚    
          if (this.zoomNum > 0.1) {
            this.zoomNum -= 0.1;
            for (var j = 0; j < p.length; j++) {
              canvasDom.style[p[j] + "Transform"] = "scale(" + this.zoomNum + ")";
              canvasDom.style[p[j] + "TransformOrigin"] = oString;
            }

            canvasDom.style["transform"] = "scale(" + this.zoomNum + ")";
            canvasDom.style["transformOrigin"] = oString;
          }
        }
        return false;
      });
    },
    // 返回唯一标识
    getUUID () {
      return Math.random().toString(36).substr(3, 10)
    },
    jsPlumbInit () {
      // 初始化jsplumb容器
      this.jsPlumb.ready(() => {
        // 导入默认配置
        this.jsPlumb.importDefaults(this.jsplumbSetting)
        // 会使整个jsPlumb立即重绘。
        this.jsPlumb.setSuspendDrawing(false, true);
        // 初始化节点
        this.loadEasyFlow()
        // 给连接线添加点击事件
        this.jsPlumb.bind('click', (conn, originalEvent) => {
          console.log("originalEvent", originalEvent)
          this.activeElement.type = 'line'
          this.activeElement.sourceId = conn.sourceId
          this.activeElement.targetId = conn.targetId
          this.$refs.nodeForm.lineInit({
            from: conn.sourceId,
            to: conn.targetId,
            label: conn.getLabel()
          })
        })
        this.jsPlumb.bind("dblclick", (conn) => {
          this.$confirm('确定删除所点击的线吗?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.jsPlumb.deleteConnection(conn)
            this.$message.success("删除成功")
          }).catch(() => {
            this.$message.info("取消删除")
          })
        })
        // 连线成功回调
        this.jsPlumb.bind("connection", (evt) => {
          let from = evt.source.id
          let to = evt.target.id
          if (this.loadEasyFlowFinish) {
            // 向数据列表中保存刚刚建立的连线数据
            this.data.lineList.push({ from: from, to: to })
          }
        })

        // 删除连线回调
        this.jsPlumb.bind("connectionDetached", (evt) => {
          // 向数据源更新数据,即在数据源中清除当前的关系
          this.deleteLine(evt.sourceId, evt.targetId)
        })

        // 改变线的连接节点
        this.jsPlumb.bind("connectionMoved", (evt) => {
          // 向数据源更新数据
          this.changeLine(evt.originalSourceId, evt.originalTargetId)
        })

        // 连线右击
        this.jsPlumb.bind("contextmenu", (evt) => {
          console.log('contextmenu', evt)
        })

        // 绑定建立连线前的回调事件
        this.jsPlumb.bind("beforeDrop", (evt) => {
          let from = evt.sourceId
          let to = evt.targetId
          // 连线情况判断分析
          if (from === to) {
            this.$message.error('节点不支持连接自己')
            return false
          }
          if (this.hasLine(from, to)) {
            this.$message.error('该关系已存在,不允许重复创建')
            return false
          }
          // 回环控制
          // if (this.hashOppositeLine(from, to)) {
          //   this.$message.error('不支持两个节点之间连线回环');
          //   return false
          // }
          this.$message.success('连接成功')
          return true
        })

        // beforeDetach
        this.jsPlumb.bind("beforeDetach", (evt) => {
          console.log('beforeDetach', evt)
        })
        // 建立一个容器，并将它作为默认容器,jsplumb的实例要绑定在一个div上，我们管这个div叫容器;因为容器里要放绝对定位的元素,所以容器要给相对定位属性
        this.jsPlumb.setContainer(this.$refs.efContainer)
      })
    },
    // 加载流程图
    loadEasyFlow () {
      // 初始化节点
      for (var i = 0; i < this.data.nodeList.length; i++) {
        let node = this.data.nodeList[i]
        // 设置源点，可以拖出线连接其他节点
        this.jsPlumb.makeSource(node.id, this.jsplumbSourceOptions)
        // // 设置目标点，其他源点拖出的线可以连接该节点
        this.jsPlumb.makeTarget(node.id, this.jsplumbTargetOptions)
        this.jsPlumb.draggable(node.id, {
          stop: function (el) {
            console.log('停止拖拽', el)
          }
        })
      }
      // 初始化连线
      for (var j = 0; j < this.data.lineList.length; j++) {
        let line = this.data.lineList[j]
        this.jsPlumb.connect({ source: line.from, target: line.to, label: line.label ? line.label : '' }, this.jsplumbConnectOptions)
      }
      this.$nextTick(function () {
        this.loadEasyFlowFinish = true
      })
    },
    // 设置连接线上的文字
    setLineLabel (from, to, label) {
      var conn = this.jsPlumb.getConnections({
        source: from,
        target: to
      })[0]
      console.log("conn", conn)
      if (!label || label === '') {
        // 如果准备修改的连接线上文字内容为空
        conn.removeClass('flowLabel')
        conn.addClass('emptyFlowLabel')
      } else {
        console.log("添加了非空条件")
        conn.addClass('flowLabel')
        conn.removeClass('emptyFlowLabel')
      }
      conn.setLabel({
        label: label,
      })
      this.data.lineList.forEach(function (line) {
        if (line.from == from && line.to == to) {
          line.label = label
        }
      })

    },
    // 删除激活的元素
    deleteElement () {
      if (this.activeElement.type === 'node') {
        this.deleteNode(this.activeElement.nodeId)
      } else if (this.activeElement.type === 'line') {
        this.$confirm('确定删除所点击的线吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          var conn = this.jsPlumb.getConnections({
            source: this.activeElement.sourceId,
            target: this.activeElement.targetId
          })[0]
          this.jsPlumb.deleteConnection(conn)
        }).catch(() => {
        })
      }
    },
    // 删除线
    deleteLine (from, to) {
      this.data.lineList = this.data.lineList.filter(function (line) {
        if (line.from == from && line.to == to) {
          return false
        }
        return true
      })
    },
    // 改变连线
    changeLine (oldFrom, oldTo) {
      this.deleteLine(oldFrom, oldTo)
    },
    // 改变节点的位置
    changeNodeSite (data) {
      for (var i = 0; i < this.data.nodeList.length; i++) {
        let node = this.data.nodeList[i]
        if (node.id === data.nodeId) {
          node.left = data.left
          node.top = data.top
        }
      }
    },
    changeNodeForm () {
      this.isNodeFormActive = !this.isNodeFormActive
    },
    /**
     * 拖拽结束后添加新的节点
     * @param evt
     * @param nodeMenu 被添加的节点对象
     * @param mousePosition 鼠标拖拽结束的坐标
     */
    addNode (evt, nodeMenu, mousePosition) {
      console.log(mousePosition)
      var screenX = evt.originalEvent.clientX, screenY = evt.originalEvent.clientY
      let efContainer = this.$refs.efContainer
      var containerRect = efContainer.getBoundingClientRect()
      var left = screenX, top = screenY
      // 计算是否拖入到容器中
      if (left < containerRect.x || left > containerRect.width + containerRect.x || top < containerRect.y || containerRect.y > containerRect.y + containerRect.height) {
        this.$message.error("请把节点拖入到画布中")
        return
      }
      left = left - containerRect.x + efContainer.scrollLeft
      top = top - containerRect.y + efContainer.scrollTop
      // 居中
      left -= 85
      top -= 16
      var nodeId = this.getUUID()
      // 动态生成名字
      var origName = nodeMenu.name
      var nodeName = origName
      var index = 1
      while (index < 10000) {
        var repeat = false
        for (var i = 0; i < this.data.nodeList.length; i++) {
          let node = this.data.nodeList[i]
          if (node.name === nodeName) {
            nodeName = origName + index
            repeat = true
          }
        }
        if (repeat) {
          index++
          continue
        }
        break
      }
      var node = {
        id: nodeId,
        name: nodeName,
        type: nodeMenu.type,
        left: left + 'px',
        top: top + 'px',
        ico: nodeMenu.ico,
        state: 'success'
      }
      /**
       * 这里可以进行业务判断、是否能够添加该节点
       */
      this.data.nodeList.push(node)
      this.$nextTick(function () {
        this.jsPlumb.makeSource(nodeId, this.jsplumbSourceOptions)
        this.jsPlumb.makeTarget(nodeId, this.jsplumbTargetOptions)
        this.jsPlumb.draggable(nodeId, {
          containment: 'parent'
        })
      })
    },
    /**
     * 删除节点
     * @param nodeId 被删除节点的ID
     */
    deleteNode (nodeId) {
      this.$confirm('确定要删除节点' + nodeId + '?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        closeOnClickModal: false
      }).then(() => {
        /**
         * 这里需要进行业务判断，是否可以删除
         */
        this.data.nodeList = this.data.nodeList.filter(function (node) {
          if (node.id === nodeId) {
            // 伪删除，将节点隐藏，否则会导致位置错位
            // node.show = false
            return false
          }
          return true
        })
        this.$nextTick(function () {
          this.jsPlumb.removeAllEndpoints(nodeId);
        })
      }).catch(() => {
      })
      return true
    },
    clickNode (nodeId) {
      this.activeElement.type = 'node'
      this.activeElement.nodeId = nodeId
      // 点击了节点之后,触发node_form下的nodeInit方法
      this.$refs.nodeForm.nodeInit(this.data, nodeId)
    },
    // 是否具有该线
    hasLine (from, to) {
      for (var i = 0; i < this.data.lineList.length; i++) {
        var line = this.data.lineList[i]
        if (line.from === from && line.to === to) {
          return true
        }
      }
      return false
    },
    // 是否含有相反的线
    hashOppositeLine (from, to) {
      return this.hasLine(to, from)
    },
    nodeRightMenu (nodeId, evt) {
      this.menu.show = true
      this.menu.curNodeId = nodeId
      this.menu.left = evt.x + 'px'
      this.menu.top = evt.y + 'px'
    },
    // 流程数据信息
    dataInfo () {
      this.flowInfoVisible = true
      this.$nextTick(function () {
        this.$refs.flowInfo.init()
      })
    },
    // 加载流程图
    dataReload (data) {
      this.easyFlowVisible = false
      this.data.nodeList = []
      this.data.lineList = []
      this.$nextTick(() => {
        data = lodash.cloneDeep(data)
        this.easyFlowVisible = true
        this.data = data
        this.$nextTick(() => {
          this.jsPlumb = jsPlumb.getInstance()
          this.$nextTick(() => {
            this.jsPlumbInit()
          })
        })
      })
    },
    // 模拟载入数据dataA
    dataReloadA () {
      this.currentData = data_A
      this.dataReload(data_A)
    },
    // 模拟载入数据dataB
    dataReloadB () {
      this.currentData = data_B
      this.dataReload(data_B)
    },
    // 模拟载入数据dataC
    dataReloadC () {
      this.currentData = data_C
      this.dataReload(data_C)
    },
    // 下载数据
    downloadData () {
      var conn = this.jsPlumb.getConnections({
        source: 'nodeA',
        target: 'nodeB'
      })[0]
      console.log(conn)
      this.$confirm('确定要下载该流程数据吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        closeOnClickModal: false
      }).then(() => {
        var datastr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(this.data, null, '\t'));
        var downloadAnchorNode = document.createElement('a')
        downloadAnchorNode.setAttribute("href", datastr);
        downloadAnchorNode.setAttribute("download", 'data.json')
        downloadAnchorNode.click();
        downloadAnchorNode.remove();
        this.$message.success("正在下载中,请稍后...")
      }).catch(() => {
      })
    }
  }
}
</script>
<style lang="less">
.node-form-active {
  transform: translateX(100%);
  /* width: 0 !important; */
}
/* 节点详情部分 */
.node-detail {
  width: 300px;
  border: 1px solid #dce3e8;
  background-color: #fbfbfb;
  transition: all 0.5s linear !important;
  position: fixed;
  right: 0;
  background-color: #fff;
  /* box-sizing: border-box; */
  padding: 0 0 30px;
  .ef-node-form-header {
    text-align: center;
  }
}

/* 中间画布的容器*/
.center-container {
  flex: 1;
  background-image: linear-gradient(#eee 1px, transparent 0),
    linear-gradient(90deg, #eee 1px, transparent 0),
    linear-gradient(#f5f5f5 1px, transparent 0),
    linear-gradient(90deg, #f5f5f5 1px, transparent 0);
  background-size: 75px 75px, 75px 75px, 15px 15px, 15px 15px;
  overflow: scroll;
  /*画布容器*/
  .container {
    height: 100%;
  }
  .all-nodes {
    width: 100%;
    height: 100%;
  }
  #efContainer {
    position: relative;
    // overflow: hidden;
    flex: 1;
    transform-origin: 20% 20%;
  }
}
</style>