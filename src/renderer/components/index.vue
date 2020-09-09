<template>
  <div class="vue-add-name">
    <el-upload
      class="upload-demo"
      drag
      action=""
      multiple
      :on-change="changFile"
      :auto-upload="false">
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处</div>
      <div class="el-upload__tip" slot="tip">支持文件或文件夹批量操作</div>
    </el-upload>
    <el-dialog
      title="提示"
      :visible.sync="dialogVisible"
      width="70%">
      <div :key="key">
        <div v-for="(value, path) in success" :key="path">
          正在写入{{ fileType }}  <span class="green">{{path}}</span>  的name属性，请稍后....
          <i :class="!value ? 'el-icon-loading' : 'el-icon-success'" :style="`color: ${!value ? 'red' : 'green'}`"></i>
        </div>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import fs from 'fs'
import path from 'path'
export default {
  data () {
    return {
      dialogVisible: false,
      fileType: '文件',
      path: '',
      hasName: 'export default { \n  name',
      name: 'ssa-web-next',
      success: {},
      key: 0
    }
  },
  methods: {
    changFile (file, fileList) {
      this.dialogVisible = true
      fs.stat(file.raw.path, (err, stats) => {
        if (err) {
          return
        }
        stats.isFile() ? this.setFile(file.raw.path) : this.setDirectory(file.raw.path)
      })
    },
    setDirectory (filePath) {
      fs.readdir(filePath, (err, files) => {
        if (err) {
          console.warn(err)
        } else {
          files.forEach((filename) => {
            var filedir = path.join(filePath, filename)
            fs.stat(filedir, (err, stats) => {
              if (err) {
                console.warn('获取文件stats失败')
              } else {
                var isFile = stats.isFile()
                var isDir = stats.isDirectory()
                if (isFile) {
                  this.setFile(filedir)
                }
                if (isDir) {
                  this.setDirectory(filedir)
                }
              }
            })
          })
        }
      })
    },
    setFile (file) {
      this.success[file] = false
      this.key++
      if (file.includes('.vue')) {
        this.setName(file)
      }
    },
    setName (file) {
      fs.readFile(file, 'utf-8', (err, data) => {
        if (err) {
          console.log(err)
        } else {
          if (data.includes(this.hasName)) {
            console.log('已经有name属性')
          } else {
            const name = this.getName(file)
            console.log(name)
            const newContent = data.replace('export default {', `export default {\n  name: '${name}',`)
            fs.writeFileSync(file, newContent)
          }
          this.success[file] = true
          this.key++
        }
      })
    },
    getName (file) {
      const index = file.indexOf(this.name)
      const str = file.substring(index + this.name.length + 1, file.length - 4)
      const newStr = str.split('\\').join('-')
      const string = newStr.replace(/-(\w)/g, (all, letter) => {
        return letter.toUpperCase()
      })
      return string.replace(string[0], string[0].toUpperCase())
    }
  }
}
</script>
<style>
html,body,#app {
  height: 100%;
  padding: 0;
  margin: 0;
}
.vue-add-name {
  height: 75%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
}
.el-upload-list{
  display: none;
}
.upload-demo {
  width: 100%;
  height: 50%;
  text-align: center;
}
.upload-demo>div{
  display: block;
  height: 100%;
  width: 70%;
  margin: 0 auto;
}
.el-upload__tip {
  height: 100px!important;
}
.upload-demo>div>div{
  width: 100%;
  height: 50%;
  height: calc(100% - 30px);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
}
.green {
  color: green;
}
/* .el-upload-dragger .el-icon-upload {
  width: 100%;
  margin-top: 20%;
}
.el-upload__text {
  margin-top: -20%;
} */
</style>