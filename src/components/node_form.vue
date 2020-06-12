<template>
  <div>
    <div class="ef-node-form">
      <div class="ef-node-form-header">节点信息</div>
      <div class="ef-node-form-body">
        <el-form :model="node" ref="dataForm" label-width="80px" v-show="type === 'node'">
          <el-form-item label="类型">
            <el-input v-model="node.type" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="left坐标">
            <el-input disabled v-model="node.left"></el-input>
          </el-form-item>
          <el-form-item label="top坐标">
            <el-input disabled v-model="node.top"></el-input>
          </el-form-item>
          <el-form-item label="名称">
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

        <el-form :model="line" ref="dataForm" label-width="80px" v-show="type === 'line'">
          <el-form-item label="条件">
            <el-input v-model="line.label"></el-input>
          </el-form-item>
          <el-form-item class="nodeFormButton">
            <!-- <el-button icon="el-icon-close" class="reset">重置</el-button> -->
            <el-button type="primary" icon="el-icon-check" class="save" @click="saveLine">保存</el-button>
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
// import '../utils/jsplumb'
// import { jsPlumb } from 'jsplumb'
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
      isEmpty: false
    }
  },
  methods: {
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
      this.$emit('setLineLabel', this.line.from, this.line.to, this.line.label)
    },
    save () {
      this.data.nodeList.filter((node) => {
        if (node.id === this.node.id) {
          node.name = this.node.name
          node.left = this.node.left
          node.top = this.node.top
          node.icon = this.node.icon
          node.state = this.node.state
        }
      })
    },
    showNodeInfo () {
      this.$parent.nodeDetailVisible = true
      console.log("1111", this.$parent.nodeDetailVisible)
    },
    showNodeForm () {
      this.$emit("changeNodeForm", this.isShowNodeForm)
      setTimeout(() => {
        this.isShowNodeForm = !this.isShowNodeForm
      }, 500)
    }
  }
}
</script>

<style lang="less">
.el-node-form-tag {
  position: absolute;
  top: 50%;
  margin-left: -15px;
  height: 40px;
  width: 15px;
  background-color: #409eff;
  display: flex;
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
</style>
