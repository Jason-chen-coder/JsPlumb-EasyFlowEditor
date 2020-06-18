<template>
  <div v-if="flowVisible" style="height: calc(100vh);over-flow:hidden" id="editor">
    <el-row>
      <!--顶部工具菜单-->
      <el-col :span="24">
        <div class="ee-tooltar">
          <el-link type="primary" :underline="false">
            <h2>Easy Editor</h2>
          </el-link>
          <span class="tooltar-center">
            <el-link type="primary" :underline="false">{{data.name}}</el-link>
            <el-divider direction="vertical"></el-divider>
            <el-popover placement="bottom" style="text-align:center" width="150" trigger="hover">
              <div style="text-align:center">生成图片</div>
              <el-button
                type="text"
                icon="el-icon-picture-outline"
                slot="reference"
                size="large"
                @click="saveAsImage"
              ></el-button>
            </el-popover>
            <el-divider direction="vertical"></el-divider>
            <el-popover placement="bottom" style="text-align:center" trigger="hover" width="150">
              <div style="text-align:center">删除节点</div>
              <el-button
                type="text"
                icon="el-icon-delete"
                slot="reference"
                size="large"
                @click="deleteElement"
                :disabled="!this.activeElement.type"
              ></el-button>
            </el-popover>
            <el-divider direction="vertical"></el-divider>
            <el-popover placement="bottom" style="text-align:center" trigger="hover" width="150">
              <div style="text-align:center">{{isShowGrid ? '隐藏网格线':'显示网格线'}}</div>
              <el-button
                type="text"
                :icon="(isShowGrid?'icon-dituleiwanggequ-copy ':'icon-fangxingweixuanzhong-copy ')+'iconfont'"
                style="vertical-align: middle;"
                slot="reference"
                size="large"
                @click="isShowGrid=!isShowGrid"
              ></el-button>
            </el-popover>
            <el-divider direction="vertical"></el-divider>
            <el-popover placement="bottom" style="text-align:center" trigger="hover" width="150">
              <div style="text-align:center">下载数据</div>
              <el-button
                slot="reference"
                type="text"
                icon="el-icon-download"
                size="large"
                @click="downloadData"
              ></el-button>
            </el-popover>
            <el-divider direction="vertical"></el-divider>
            <el-popover placement="bottom" style="text-align:center" trigger="hover" width="150">
              <div style="text-align:center">上传数据</div>
              <el-button
                slot="reference"
                type="text"
                icon="el-icon-upload2"
                size="large"
                @click="uploadDataVisible=true"
              ></el-button>
            </el-popover>
          </span>
          <div style="float: right;margin-right: 5px">
            <el-button plain round icon="el-icon-document" @click="dataInfo" size="mini">流程图数据</el-button>
            <el-button plain round @click="dataReloadA" icon="el-icon-refresh" size="mini">切换流程A</el-button>
            <el-button plain round @click="dataReloadB" icon="el-icon-refresh" size="mini">切换流程B</el-button>
            <el-button plain round @click="dataReloadC" icon="el-icon-refresh" size="mini">切换流程C</el-button>
            <el-button plain round @click="dataReloadD" icon="el-icon-refresh" size="mini">切换流程D</el-button>
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
          :step="0.001"
          :max="1.3"
          :min="0.7"
          v-model="scaleValue"
          vertical
          height="90px"
        ></el-slider>
        <i class="el-icon-minus shrink-btn"></i>
      </div>
      <!-- 中间画布部分 -->
      <div
        class="center-container"
        :class="{'background-grid':isShowGrid}"
        id="center-container"
        ref="centerContainer"
        v-flowDrag
      >
        <!-- jsplumb的容器 -->
        <div id="eeContainer" ref="eeContainer" class="container">
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
          @repaintEverything="repaintEverything"
          @changeNodeForm="changeNodeForm"
          @setLineLabel="setLineLabel"
          @changeNodeSite="changeNodeSite"
        ></flow-node-form>
      </div>
    </div>
    <!-- 流程数据详情 -->
    <flow-info v-if="flowInfoVisible" ref="flowInfo" :data="data"></flow-info>
    <!-- 节点详情 -->
    <node-detail v-if="nodeDetailVisible" :currentNode="currentNode"></node-detail>
    <!-- 上传数据 -->
    <upload-data v-if="uploadDataVisible" @uploadPaintFlow="uploadPaintFlow"></upload-data>
    <!-- 一键生成图片 -->
    <image-load v-if="imageLoadVisible" :imgUrl="imgUrl" ref="imageLoad" :isShowGrid="isShowGrid"></image-load>
  </div>
</template>

<script>
import { jsPlumb } from 'jsplumb'
//  2.使用domtoimage 转换图片
import domtoimage from 'dom-to-image';
import flowNode from '@/components/node'
import nodeMenu from '@/components/node_menu'
import FlowInfo from '@/components/info'
import nodeDetail from '@/components/node_detail'
import uploadData from '@/components/uploadData'
import imageLoad from '@/components/imageLoad'
import $ from "jquery"
import FlowNodeForm from './node_form'
import lodash from 'lodash'
import { data_A } from '../data/data_A'
import { data_B } from '../data/data_B'
import { data_C } from '../data/data_C'
import { data_D } from '../data/data_D'

export default {
  data () {
    return {
      // jsPlumb 实例
      jsPlumb: null,
      // 控制画布销毁
      flowVisible: true,
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
        Container: 'eeContainer',
        // 连线的样式 StateMachine、Flowchart，Bezier、Straight
        // Connector: ['Bezier', {curviness: 100}],
        // Connector: ['Straight', {stub: 20, gap: 1}],
        Connector: ['Flowchart', { stub: 30, gap: 1, alwaysRespectStubs: false, midpoint: 0.5, cornerRadius: 10 }],
        // Connector: ['StateMachine', {margin: 5, curviness: 10, proximityLimit: 80}],
        // 鼠标是否拖动删除线
        ConnectionsDetachable: true,
        // 删除线的时候节点不删除
        DeleteEndpointsOnDetach: false,
        // 连线的两端端点类型：矩形 eight: 矩形的高 ，idth: 矩形的宽
        // Endpoint: ['Rectangle', {height: 20, width: 20, cssClass: 'ee-rectangle', hoverClass: 'ee-rectangle-hover'}],
        //  空白端点
        Endpoint: ['Blank', { Overlays: '' }],
        // Endpoints: [['Dot', {radius: 5, cssClass: 'ee-dot', hoverClass: 'ee-dot-hover'}], ['Rectangle', {height: 20, width: 20, cssClass: 'ee-rectangle', hoverClass: 'ee-rectangle-hover'}]],
        // 线端点的样式
        EndpointStyle: { fill: '#1879ffa1', outlineWidth: 1 },
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
        Overlays: [// 箭头叠加
          ['Arrow', {
            width: 10, // 箭头尾部的宽度
            length: 8, // 从箭头的尾部到头部的距离
            location: 1, // 位置，建议使用0～1之间
            direction: 1, // 方向，默认值为1（表示向前），可选-1（表示向后）
            foldback: 0.623 // 折回，也就是尾翼的角度，默认0.623，当为1时，为正三角
          }],
          ['Label', {
            label: '',
            location: 2.2,
            cssClass: 'aLabel'
          }]
        ],
        // 绘制图的模式 svg、canvas
        RenderMode: 'svg',
        DragOptions: { cursor: 'pointer', zIndex: 2000 },
        // 鼠标滑过线的样式
        HoverPaintStyle: { stroke: '#1879ff', strokeWidth: 3, cursor: 'pointer' },
        // 滑过锚点效果
        // EndpointHoverStyle: { fill: 'red' }
        Scope: 'jsPlumb_DefaultScope' // 范围，具有相同scope的点才可连接
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
        // 设置可以拖拽的类名，只要鼠标移动到该类名上的DOM，就可以拖拽连线
        filter: '.flow-node-drag',
        filterExclude: false,
        anchor: 'Continuous',
        allowLoopback: false,
        maxConnections: -1,
        onMaxConnections: function (info) {
          console.log(`超过了最大值连线: ${info.maxConnections}`)
        }
      },
      // 参考 https://www.cnblogs.com/mq0036/p/7942139.html
      jsplumbSourceOptions2: {
        // 设置可以拖拽的类名，只要鼠标移动到该类名上的DOM，就可以拖拽连线
        filter: '.flow-node-drag',
        filterExclude: false,
        // anchor: 'Continuous',
        allowLoopback: false,
        connector: ['Flowchart', { curviness: 50 }],
        connectorStyle: {
          // 线的颜色
          stroke: 'red',
          // 线的粗细，值越大线越粗
          strokeWidth: 1,
          // 设置外边线的颜色，默认设置透明，这样别人就看不见了，点击线的时候可以不用精确点击，参考 https://blog.csdn.net/roymno2/article/details/72717101
          outlineStroke: 'transparent',
          // 线外边的宽，值越大，线的点击范围越大
          outlineWidth: 10
        },
        connectorHoverStyle: { stroke: 'red', strokeWidth: 2 }
      },
      jsplumbTargetOptions: {
        // 设置可以拖拽的类名，只要鼠标移动到该类名上的DOM，就可以拖拽连线
        filter: '.flow-node-drag',
        filterExclude: false,
        anchor: 'Continuous',
        allowLoopback: false,
        dropOptions: { hoverClass: 'ee-drop-hover' }
      },
      isNodeFormActive: false,
      // 当前的数据
      currentData: {},
      scaleValue: 1,
      // 鼠标点击的当前节点
      currentNode: {},
      // 节点详情显示
      nodeDetailVisible: false,
      // 数据文件上传显示
      uploadDataVisible: false,
      imageLoadVisible: false,
      // 图片预览
      imgUrl: "../assets/images/2.5D-2.jpg",
      // 网格线显示控制 
      isShowGrid: true
    }
  },
  components: {
    uploadData,
    imageLoad,
    flowNode,
    nodeMenu,
    FlowInfo,
    FlowNodeForm,
    nodeDetail
  },
  directives: {
    // 鼠标在画布中进行拖拽时，动态修改流程图在画布的位置
    'flowDrag': {
      bind (el, binding, vnode, oldNode) {
        console.log("flowDrag", vnode, oldNode)
        if (!binding) {
          return
        }
        el.onmousedown = (e) => {
          if (e.button == 2) {
            // 右键需求待完成
            console.log("右键需求待完成")
            return
          }
          //  鼠标按下，计算鼠标在当前可视区的坐标
          let disX = e.clientX
          let disY = e.clientY
          // 移除被选中节点的样式
          $(".ee-node-active").removeClass("ee-node-active")
          el.style.cursor = 'move'
          document.onmousemove = function (e) {
            // 移动时禁止默认事件
            e.preventDefault()
            // e.clientX/e.clientY为在鼠标移动中的最新坐标
            // left为触发一次移动事件横向移动的距离
            const left = e.clientX - disX
            el.scrollLeft += -left
            // top为触发一次移动事件竖直移动的距离
            const top = e.clientY - disY
            el.scrollTop += -top
            // 更新当前鼠标在可视区域的坐标,便于下次触发移动事件计算
            disX = e.clientX
            disY = e.clientY
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
      this.currentData = data_A
      this.dataReload(this.currentData)
    })
    //  加载版本提示框
    this.$notify({
      title: '欢迎使用 Easy Editor',
      dangerouslyUseHTMLString: true,
      message: "<p> 当前版本：1.0.0 <br /> Powered by Jason chen<br /> Gitee:<a target='blank' href='https://gitee.com/save_money/Easy-Editor'>  Easy Editor  </a> </p>",
      showClose: true,
      iconClass: "icon-workflow-copy iconfont",
      position: 'bottom-right',
      offset: 100,
      customClass: "notify-tip"
    });
  },
  methods: {
    saveAsImage () {
      // 将流程图保存为图片并将图片地址传递给子组件（使用dom2image）
      // var node = document.getElementById('center-container');
      var node = document.getElementById('eeContainer');
      domtoimage.toPng(node)
        .then((dataUrl) => {
          this.imgUrl = dataUrl
          this.$refs.imageLoad.loading = false
        })
        .catch(function (error) {
          console.error('oops, something went wrong!', error);
        });
      this.imageLoadVisible = true;

    },
    //点击了某个节点
    showNodeDetail (node) {
      this.nodeDetailVisible = true
      this.currentNode = Object.create(node)
    },
    //开始放大缩小
    startScale () {
      this.$refs.eeContainer.style.transform = `scale(${this.scaleValue})`
      this.jsPlumb.setZoom(`${this.scaleValue}`);
      // 获取center-container的宽高作为原始宽高,然后以放大/缩小倍数计算后给eecontainer
      var containWidth = $("#center-container").width()
      var containHeight = $("#center-container").height()
      if (this.scaleValue < 1) {
        $("#eeContainer").width(containWidth / this.scaleValue)
        $("#eeContainer").height(containHeight / this.scaleValue)
      } else {
        $("#eeContainer").width(containWidth * this.scaleValue)
        $("#eeContainer").height(containHeight * this.scaleValue)
      }
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
          console.log(originalEvent)
          this.activeElement.type = 'line'
          this.activeElement.sourceId = conn.sourceId
          this.activeElement.targetId = conn.targetId
          this.$refs.nodeForm.lineInit({
            from: conn.sourceId,
            to: conn.targetId,
            label: conn.getLabel(),
            type: conn.connector.type
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
        this.jsPlumb.setContainer(this.$refs.eeContainer)
      })
    },
    // 加载流程图
    loadEasyFlow () {
      // 初始化节点
      for (var i = 0; i < this.data.nodeList.length; i++) {
        let node = this.data.nodeList[i]
        let _this = this
        // 设置源点，可以拖出线连接其他节点
        this.jsPlumb.makeSource(node.id, this.jsplumbSourceOptions)
        // 设置目标点，其他源点拖出的线可以连接该节点
        this.jsPlumb.makeTarget(node.id, this.jsplumbTargetOptions)
        this.jsPlumb.draggable(node.id, {
          containment: 'parent',
          stop: (el) => {
            _this.currentData = _this.data
            console.log('停止拖拽', el)
          }
        })
        this.jsPlumb.bind("click", function (conn, originalEvent) {
          console.log(conn, originalEvent)
          // this.showNodeDetail(conn)
        });
      }
      // 初始化连线
      for (var j = 0; j < this.data.lineList.length; j++) {
        let line = this.data.lineList[j]
        var connParam = {
          source: line.from,
          target: line.to,
          label: line.label ? line.label : '',
          connector: line.connector ? line.connector : '',
          anchors: line.anchors ? line.anchors : undefined,
          paintStyle: line.paintStyle ? line.paintStyle : undefined,
        }
        this.jsPlumb.connect(connParam, this.jsplumbConnectOptions)
      }
      this.$nextTick(function () {
        this.loadEasyFlowFinish = true
      })
    },
    // 子组件上触发的设置连接线属性
    setLineLabel (from, to, label, type) {
      var conn = this.jsPlumb.getConnections({
        source: from,
        target: to
      })[0]
      if (!label || label === '' || label == null) {
        // 如果准备修改的连接线上文字内容为空
        //如果原来就没有文字就直接返回
        if (!conn.getLabel()) {
          return
        }
        // 原来就有文字就添加类名
        conn.removeClass('flowLabel')
        conn.addClass('emptyFlowLabel')
      } else {
        conn.addClass('flowLabel')
        conn.removeClass('emptyFlowLabel')
        console.log(type)
        // 修改连接线的样式
        // conn.setConnector(eval(type));
        //  修改连接线上的文字
        conn.setLabel({
          label: label,
        })
      }
    },
    // 删除激活的元素
    deleteElement () {
      if (this.activeElement.type === 'node') {
        // 当我们点击节点时,会将this.activeElement.type设置为node
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
      var screenX = evt.originalEvent.clientX
      let screenY = evt.originalEvent.clientY
      let eeContainer = this.$refs.eeContainer
      var containerRect = eeContainer.getBoundingClientRect()// 获取eeContainer的大小及其相对于视口的位置
      var left = screenX
      var top = screenY
      // 计算是否拖入到容器中
      if (left < containerRect.x || left > containerRect.width + containerRect.x || top < containerRect.y || containerRect.y > containerRect.y + containerRect.height) {
        this.$message.error("请把节点拖入到画布中")
        return
      }
      left = left - containerRect.x + eeContainer.scrollLeft
      top = top - containerRect.y + eeContainer.scrollTop
      // 让节点放在画布上时让鼠标在节点中间
      left -= 85
      top -= 16
      var nodeId = this.getUUID() // 准备唯一的id给节点
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
        state: nodeMenu.state
      }
      /**
       * 这里可以进行业务判断、是否能够添加该节点
       */
      //向节点数据列表添加节点
      this.data.nodeList.push(node)
      this.$nextTick(function () {
        // 当节点为源点时，可以拖出线连接其他节点
        this.jsPlumb.makeSource(nodeId, this.jsplumbSourceOptions)
        // 当节点目标点时，其他源点拖出的线可以连接该节点
        this.jsPlumb.makeTarget(nodeId, this.jsplumbTargetOptions)
        // 使当前节点可以被拖拽
        this.jsPlumb.draggable(nodeId)
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
            // 将当前被删除的节点过滤掉
            return false
          }
          return true
        })
        this.$nextTick(function () {
          //  在jsplumb实例中删除当前节点
          this.jsPlumb.removeAllEndpoints(nodeId);
        })
      }).catch(() => {
      })
      return true
    },
    // 点击了子组件node后,子组件触发的事件
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
    repaintEverything () {
      console.log('重绘')
      this.jsPlumb.repaint()
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
      this.flowVisible = false
      this.data.nodeList = []
      this.data.lineList = []
      this.$nextTick(() => {
        data = lodash.cloneDeep(data)
        this.flowVisible = true
        this.data = data
        this.currentData = data
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
      console.log(data_C)
      this.dataReload(data_C)
    },
    // 模拟载入数据dataD
    dataReloadD () {
      this.currentData = data_D
      this.dataReload(data_D)
    },
    // 文件上传子组件触发的重绘事件
    uploadPaintFlow (data) {
      try {
        console.log(JSON.parse(data))
        this.currentData = JSON.parse(data)
        this.dataReload(this.currentData)
      } catch (error) {
        console.log("出错啦", error)
      }
    },
    // 下载数据
    downloadData () {
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
    },
  }
}
</script>
<style lang="less">
/*顶部工具栏*/
.ee-tooltar {
  padding-left: 30px;
  box-sizing: border-box;
  height: 45px;
  line-height: 40px;
  z-index: 3;
  background-color: #fff;
  border-bottom: 1px solid #dadce0;
}

/* 缩放工具 */
.zoom-tool {
  opacity: 0.3;
  position: fixed;
  left: 270px;
  /* top: 74px; */
  bottom: 60px;
  z-index: 1;
  .enlarged-btn {
    position: absolute;
    top: -30px;
    left: 50%;
    transform: translateX(-50%);
  }
  .shrink-btn {
    position: absolute;
    bottom: -30px;
    left: 50%;
    transform: translateX(-50%);
  }
}
.zoom-tool:hover {
  opacity: 1;
}
.zoom-tool::before {
  content: "";
}
.jtk-overlay {
  cursor: pointer;
  color: #4a4a4a;
}

.node-form-active {
  transform: translateX(100%);
  /* width: 0 !important; */
}
.tooltar-center {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}

/* 中间画布的容器*/
.center-container {
  flex: 1;
  background-color: #fff;
  overflow: hidden;
  /*画布容器*/
  .container {
    height: 100%;
  }
  .all-nodes {
    width: 100%;
    height: 100%;
  }
  #eeContainer {
    position: relative;
    overflow: hidden;
    flex: 1;
    transform-origin: 0% 0%;
  }
}
// 背景网格
.background-grid {
  // background-color: #fff !important;
  background-image: linear-gradient(#ddd 1px, transparent 0),
    linear-gradient(90deg, #ddd 1px, transparent 0),
    linear-gradient(#f6f6f6 1px, transparent 0),
    linear-gradient(90deg, #f6f6f6 1px, transparent 0);
  background-size: 75px 75px, 75px 75px, 15px 15px, 15px 15px;
}
#center-container::-webkit-scrollbar,
#eeContainer::-webkit-scrollbar,
#editor::-webkit-scrollbar {
  display: none;
}

/* 节点详情部分 */
.node-detail {
  width: 300px;
  height: 100%;
  border: 1px solid #dce3e8;
  background-color: #fbfbfb;
  transition: all 0.3s linear !important;
  position: fixed;
  // position: absolute;
  right: 0;
  background-color: #fff;
  /* box-sizing: border-box; */
  .ee-node-form-header {
    text-align: center;
  }
}
.notify-tip {
  margin-right: 300px;
}
// .imagebackground {
//   background-color: #fff;
//   background-image: linear-gradient(#eee 1px, transparent 0),
//     linear-gradient(90deg, #eee 1px, transparent 0),
//     linear-gradient(#f5f5f5 1px, transparent 0),
//     linear-gradient(90deg, #f5f5f5 1px, transparent 0);
//   background-size: 75px 75px, 75px 75px, 15px 15px, 15px 15px;
// }
</style>