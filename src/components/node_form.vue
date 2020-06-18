<template>
  <div>
    <div class="ee-node-form">
      <div class="ee-node-form-header" v-if="type === 'line'">连线属性</div>
      <div class="ee-node-form-header" v-if="type === 'node'">节点属性</div>
      <div class="ee-node-form-body">
        <el-form :model="node" ref="dataForm" label-width="80px" v-if="type === 'node'">
          <el-form-item label="类型">
            <el-input v-model="node.type" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="left坐标">
            <el-input disabled v-model="node.left"></el-input>
          </el-form-item>
          <el-form-item label="top坐标">
            <el-input disabled v-model="node.top"></el-input>
          </el-form-item>
          <el-form-item label="节点名称">
            <el-input v-model="node.name"></el-input>
          </el-form-item>
          <el-form-item label="ico图标">
            <el-input v-model="node.ico"></el-input>
          </el-form-item>
          <el-form-item label="节点状态">
            <el-input v-model="node.state"></el-input>
          </el-form-item>
          <el-form-item class="nodeFormButton">
            <el-button icon="el-icon-connection" class="nodeInfo" @click="showNodeInfo">节点详情</el-button>
            <el-button type="primary" icon="el-icon-check" class="save" @click="save">保存</el-button>
          </el-form-item>
        </el-form>

        <el-form :model="line" ref="dataForm" label-width="80px" v-if="type === 'line'">
          <el-form-item label="条件">
            <el-input v-model="line.label"></el-input>
          </el-form-item>
          <!-- 选择节点连接线样式 -->
          <!-- <el-form-item label="连线样式">
            <el-select @change="selectLineStyle" v-model="line.type">
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              ></el-option>
            </el-select>
          </el-form-item>-->
          <el-form-item class="nodeFormButton">
            <el-button type="primary" icon="el-icon-check" class="saveLine" @click="saveLine">保存</el-button>
          </el-form-item>
        </el-form>
      </div>
      <div class="el-node-form-tag" @click="showNodeForm">
        <i class="el-icon-arrow-right" v-if="isShowNodeForm"></i>
        <i v-else class="el-icon-arrow-left"></i>
      </div>
    </div>
  </div>
</template>

<script>
import { cloneDeep } from 'lodash'
export default {
  data () {
    return {
      visible: true,
      // node 或 line
      type: 'node',
      node: {},
      line: { label: "", from: "", to: "" },
      data: {},
      isShowNodeForm: true,
      isEmpty: false,
      // 连接线的样式值
      lineValue: '',
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

    }
  },
  methods: {
    // 设置连接线的风格
    selectLineStyle () {
      // this.jsplumbSetting.Connector = eval(this.line.type)
      // this.dataReload(this.currentData)
      // console.log(this.data)
    },
    /**
     * 表单修改，这里可以根据传入的ID进行业务信息获取
     * @param data
     * @param id
     */
    nodeInit (data, id) {
      this.type = 'node'
      // 将父组件中的所有节点数据进行浅拷贝,实现在子组件中实时修改节点数据
      this.data = data
      data.nodeList.filter((node) => {
        if (node.id === id) {
          this.node = cloneDeep(node)
        }
      })
    },
    lineInit (line) {
      this.type = 'line'
      if (this.isEmpty) {
        line.label = ''
      }
      this.line = line
      console.log(line)
    },
    // 修改连线
    saveLine () {
      console.log(this.line)
      if (this.line.label === "") {
        this.isEmpty = true
      } else {
        this.isEmpty = false
      }
      this.$emit('setLineLabel', this.line.from, this.line.to, this.line.label, this.line.type)
    },
    save () {
      this.data.nodeList.filter((node) => {
        if (node.id === this.node.id) {
          node.name = this.node.name
          node.left = this.node.left
          node.top = this.node.top
          node.icon = this.node.icon
          node.state = this.node.state
          this.$emit('repaintEverything')
        }
      })
    },
    showNodeInfo () {
      if (this.node.id) {
        this.$parent.nodeDetailVisible = true
      } else {
        this.$message.error("请先选择节点")
      }

    },
    showNodeForm () {
      this.$emit("changeNodeForm", this.isShowNodeForm)
      setTimeout(() => {
        this.isShowNodeForm = !this.isShowNodeForm
      }, 300)
    }
  }
}
</script>

<style lang="less">
.el-node-form-tag {
  position: absolute;
  top: 43%;
  margin-left: -10px;
  height: 50px;
  font-size: 10px;
  width: 10px;
  background-color: #409eff;
  display: flex;
  border-radius: 10px 0 0 10px;
  color: #fff;
  justify-content: center;
  align-items: center;
}

.nodeFormButton {
  position: relative;
  float: left;
  .el-form-item__content {
    margin-left: 40px !important;
  }
}
.nodeFormButton .nodeInfo {
  position: absolute;
  left: 40px;
}
.nodeFormButton .save {
  position: absolute;
  left: 150px;
}
.nodeFormButton .saveLine {
  position: absolute;
  left: 50%;
  transform: translateX(100%);
}
</style>
