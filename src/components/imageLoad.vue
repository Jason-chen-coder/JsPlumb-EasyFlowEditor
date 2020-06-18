<template>
  <el-dialog
    title="流程图"
    :visible.sync="dialogVisible"
    width="60%"
    class="image-load-dialog"
    top="10vh"
    center
    @close="closeImageLoad"
  >
    <div
      class="load-image-box"
      :class="{'show-grid':isShowGrid}"
      v-loading="loading"
      element-loading-text="流程图生成中"
    >
      <img :src="imgUrl" alt class="load-image" v-if="!loading" />
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  props: {
    imgUrl: String,
    isShowGrid: Boolean
  },
  mounted () {
    this.dialogVisible = true
    setTimeout(() => { }, 1000)
  },
  data () {
    return {
      dialogVisible: false,
      loading: true
    }
  },
  components: {

  },
  methods: {
    closeImageLoad () {
      this.dialogVisible = false
      this.$parent.imageLoadVisible = false
      this.isloading = false
    }
  }
}
</script>
<style lang="less">
.image-load-dialog {
  .load-image-box {
    width: 1102px;
    height: 580px;
    .load-image {
      width: 100%;
      height: 100%;
      // overflow: hidden;
      display: block;
      transition: all 0.5s linear;
    }
    .load-image:hover {
      box-shadow: 0px 0px 10px 4px #dddddd;
    }
    .el-loading-mask {
      background-color: rgba(255, 255, 255, 1);
    }
  }
  .show-grid {
    background-color: #fff;
    background-image: linear-gradient(#eee 1px, transparent 0),
      linear-gradient(90deg, #eee 1px, transparent 0),
      linear-gradient(#f5f5f5 1px, transparent 0),
      linear-gradient(90deg, #f5f5f5 1px, transparent 0);
    background-size: 75px 75px, 75px 75px, 15px 15px, 15px 15px;
  }
}
</style>