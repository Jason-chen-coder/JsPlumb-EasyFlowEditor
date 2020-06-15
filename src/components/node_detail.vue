<template>
  <el-dialog
    :title="currentNodeDetail.name"
    @closed="handleclose"
    center
    :visible.sync="nodeDialogVisible"
    width="50%"
  >
    <div id="detailContainer" ref="detailContainer" class="nodeDetailInfo">
      <div id="detailbox">
        <span
          v-for="(item, index) in currentNodeDetail.nodeList"
          :id="item.id"
          :key="index"
          :ref="item.id"
          :style="{left:item.left,top:item.top}"
          class="sub-node"
        >
          <i :class="item.ico" style="color:#409eff"></i>
          <div>{{item.name}}</div>
        </span>
      </div>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button type="primary" @click="handleclose">确 定</el-button>
    </span>
  </el-dialog>
</template>
<script>
import { node_data } from "../data/detail_data"
export default {
  props: {
    currentNode: Object,
  },
  data () {
    return {
      nodeDialogVisible: false,
      currentNodeDetail: {}
    }
  },
  mounted () {
    this.currentNodeDetail = node_data
    this.nodeDialogVisible = true
  },
  methods: {
    handleclose () {
      console.log("关闭节点详情")
      this.nodeDialogVisible = false
      this.$parent.nodeDetailVisible = false
    }
  },
}
</script>
<style lang="less">
.nodeDetailInfo {
  position: relative;
  margin: 0 auto;
  width: 99%;
  height: 376px;
  background-image: linear-gradient(#eee 1px, transparent 0),
    linear-gradient(90deg, #eee 1px, transparent 0),
    linear-gradient(#f5f5f5 1px, transparent 0),
    linear-gradient(90deg, #f5f5f5 1px, transparent 0);
  background-size: 75px 75px, 75px 75px, 15px 15px, 15px 15px;
}
.sub-node {
  // position: absolute;
  // top: 50px;
  float: left;
  width: 50px;
  height: 50px;
  // border: red 1px solid;
  text-align: center;
  margin: 20px;
  i {
    display: inline-block;
    font-size: 30px;
    margin: 20px 0px;
  }
}
#detailbox {
  position: absolute;
  left: 55%;
  top: 50%;
  width: 80%;
  height: 80%;
  transform: translate(-50%, -50%);
  margin: 0 auto;
  display: inline-block;
}
</style>