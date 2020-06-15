<template>
  <el-dialog title="流程数据信息" :visible.sync="dialogVisible" width="70%">
    <!--一个高亮显示的插件-->
    <!-- <codemirror :value="flowJsonData" :options="options" class="code"></codemirror> -->
    <json-viewer :value="data" :expand-depth="5" copyable boxed sort class="code"></json-viewer>
  </el-dialog>
</template>

<script>
// import 'codemirror/lib/codemirror.css'
// import { codemirror } from 'vue-codemirror'
// require("codemirror/mode/javascript/javascript.js")
import JsonViewer from 'vue-json-viewer'

export default {
  props: {
    data: Object,
  },
  data () {
    return {
      dialogVisible: false,
      flowJsonData: {},
      options: {
        mode: { name: "javascript", json: true },
        lineNumbers: true
      }
    }
  },
  components: {
    // codemirror,
    JsonViewer
  },
  methods: {
    init () {
      this.dialogVisible = true
      // this.flowJsonData = JSON.stringify(this.data, null, 4).toString()
    }
  }
}
</script>
