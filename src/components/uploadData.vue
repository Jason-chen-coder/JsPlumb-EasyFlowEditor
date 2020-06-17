<template>
  <el-dialog title="数据上传" :visible.sync="centerDialogVisible" width="70%" center>
    <el-upload
      class="upload-box"
      drag
      accept=".yaml, .json, .js"
      action
      :file-list="uploadFiles"
      :on-change="fileChange"
      :auto-upload="false"
      :limit="1"
      :on-exceed="exceedFile"
    >
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">
        将文件拖到此处，或
        <em>点击上传</em>
        <div slot="tip" class="el-upload__tip">
          只能上传
          <em>yaml / json</em> 文件，且最大文件数量不超过
          <em>1</em> 个
        </div>
      </div>
    </el-upload>
    <!--codemirror插件-->
    <codemirror
      :value="currentNodeData"
      v-model="currentNodeData"
      :options="options"
      class="codemirror"
      height="500px"
    ></codemirror>
    <span slot="footer" class="dialog-footer">
      <el-button @click="cancelUpload">取 消</el-button>
      <el-button type="primary" @click="confirmUpload">确 定</el-button>
    </span>
  </el-dialog>
</template>
<script>
import 'codemirror/lib/codemirror.css'
import { codemirror } from 'vue-codemirror'
// 导入主题样式
import 'codemirror/theme/mdn-like.css'
// 导入数据类型识别文件
import "codemirror/mode/yaml/yaml.js"
import "codemirror/mode/javascript/javascript.js"
import jsyaml from "js-yaml"
export default {
  data () {
    return {
      uploadFiles: [],
      centerDialogVisible: false,
      currentNodeData: "",
      // codemirror配置项
      options: {
        tabSize: 4,
        styleActiveLine: true,
        matchBrackets: true,
        lineNumbers: true,
        lineWrapping: false,
        line: true,
        mode: 'text/yaml',
        theme: 'mdn-like'
      }
    }
  },
  components: {
    codemirror
  },
  watch: {
    currentNodeData: {
      handler (newvalue) {
        this.currentNodeData = newvalue
      },
      immediate: true,
      deep: true
    }
  },
  mounted () {
    this.centerDialogVisible = true
  },
  methods: {
    confirmUpload () {
      // 触发父组件重绘函数
      this.$emit("uploadPaintFlow", this.currentNodeData)
      this.$message.success("渲染成功")
      this.centerDialogVisible = false
      this.$parent.uploadDataVisible = false
    },
    cancelUpload () {
      console.log(this.currentNodeData)
      this.centerDialogVisible = false
      this.$parent.uploadDataVisible = false
      this.currentNodeData = ""
    },
    jsonToYaml (data) {
      // json转换为yaml
      return jsyaml.safeDump(data)
    },
    yamlToJson (data) {
      // yaml转换为json
      return jsyaml.safeLoad(data)
    },
    tirggerFile () {

    },
    fileChange (file) {
      if (file) {
        let reader = new FileReader()
        reader.readAsText(file.raw) //注意读取中文的是用这个编码，不是utf-8
        reader.onload = () => {
          this.$message.success("文件上传成功")
          var result = reader.result;
          // 获取文件类型
          var fileType = file.name.split(".").pop()
          if (fileType == 'yaml') {
            result = JSON.stringify(this.yamlToJson(result), null, "\t");
          }
          this.currentNodeData = result
        }
      }
    },
    exceedFile () {
      this.$message.error("超出文件可上传数量,请先清除文件")
    }
  },
}
</script>
<style lang="less">
.upload-box {
  .el-upload {
    display: block;
    width: 360px;
    margin: 0 auto;
  }
}
.codemirror {
  margin-top: 40px;
  border-style: solid;
  border-color: #578dbf;
  border-width: 2px 2px 2px 0;
  position: relative;
}
.codemirror::before {
  position: absolute;
  font-size: 17px;
  top: -30px;
  left: 10px;
  content: "数据预览 :";
}
</style>