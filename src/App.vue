<template>
  <div id="app">
    <center><h2>喜为软著代码生成</h2></center>
    <FormPage @render="renderFile" />
  </div>
</template>

<script>
import FormPage from './components/FormPage.vue'
import Docxtemplater from "docxtemplater";
import PizZip from "pizzip";
import PizZipUtils from "pizzip/utils/index.js";
import { saveAs } from "file-saver";


function loadFile(url, callback) {
  PizZipUtils.getBinaryContent(url, callback);
}

function replaceErrors(key, value) {
  if (value instanceof Error) {
    return Object.getOwnPropertyNames(value).reduce(function(
      error, key
    ) {
      error[key] = value[key];
      return error;
    }, {});
  }
  return value;
}

/* eslint-disable */
// 匹配注释和空行
var regs = /(\/\*([^*]|[\r\n]|(\*+([^*\/]|[\r\n])))*\*+\/)|(\/\/.*)|(#.*)|(<!--(.|[\r\n])*?-->)|(\n\s*\n)/g

export default {
  name: 'App',
  components: {
    FormPage
  },
  data () {
    return {
      gencode: '',
      fsList: []
    }
  },
  created () {
    
  },
  methods: {
    /**
     * 合成代码
     */
    async renderCode (data) {
      const loading = this.$loading({
        lock: true,
        text: '读取代码',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      });
      let dirs = await this.tt.fs.readDir(data.path, {recursive: true})
      this.gencode = ''
      this.fsList = []
      await this.readCodeFile(dirs)
      
      for (let path of this.fsList) {
        let ext = path.substr(path.lastIndexOf('.'))
        if (data.extname.indexOf(ext) !== -1) {
          let code = await this.tt.fs.readBinaryFile(path)
          // let code = await this.tt.fs.readTextFile(path)
          // code = code.replace(regs, '')
          // this.gencode += '\n\r' + code
          let arrayBuffer = Buffer.from(code);
          code = arrayBuffer.toString().replace(regs, '')
          code = code.replaceAll('\r\n', '\n')
          code = code.replaceAll('\n\r', '\n')
          code = code.replaceAll('\r', '')
          this.gencode += '\n' + code
        }
      }
      loading.close();
      this.genDocx({
        title: data.title,
        contents: this.gencode
      })
    },
    async readCodeFile (dirs) {
      for (let item of dirs) {
        if (item.children) {
          await this.readCodeFile(item.children)
        } else {
          this.fsList.push(item.path)
        }
      }
    },
    async renderFile (data) {
      let _data = Object.assign({}, data)
      _data.extname = _data.extname.trim().split(',')
      await this.renderCode(_data)
    },
    genDocx (data) {
      const loading = this.$loading({
        lock: true,
        text: '生成文档',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      });
      loadFile('/temp.docx', function (error, content) {
        if (error) throw error;
        var zip = new PizZip(content);
        var doc = new Docxtemplater(zip);

        let text = data.contents.split('\n').filter(item => item.trim())

        doc.setData({
          title: data.title,
          contents: text
        });

        try {
          doc.render();
        } catch (error) {
            loading.close();
            console.log(JSON.stringify({ error: error }, replaceErrors));

            if (error.properties && error.properties.errors instanceof Array) {
              const errorMessages = error.properties.errors
                .map(function(error) {
                  return error.properties.explanation;
                })
                .join("\n");
              console.log("errorMessages", errorMessages);
              // errorMessages is a humanly readable message looking like this :
              // 'The tag beginning with "foobar" is unopened'
            }
            alert('生成报错')
            throw error;
        }
        var out = doc.getZip().generate({
          type: "blob",
          mimeType: "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        })
        loading.close();
        saveAs(out, "output.docx");
      })
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
