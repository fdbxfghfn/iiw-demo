<template lang="pug">
  q-page(padding)
    q-window-resize-observable(@resize="onResize")
    .row
      .col-12(style="padding-bottom: 5px;")
        q-btn-group(push)
          q-btn(push label="运行" icon="fa fa-play" size="xs" @click="run")
          q-btn(push label="复制" icon="fa fa-copy" size="xs")
          q-btn(push label="撤销" icon="fa fa-undo" size="xs")
          q-btn(push label="重做" icon="fa fa-redo" size="xs")
          q-btn(push label="保存" icon="fa fa-save" size="xs")
    .row
      .col-8
        codemirror(ref="myCm" :value="codemirrorData" :options="cmOptions" @changes="changes" style="")
      //- .col-8
        textarea(ref="editbox").scriptbox
      .col-4
        .android-preview
          editorvalue(v-if="running").previewframe
</template>
<script>
import Vue from 'vue'
import Cookies from 'js-cookie'
import { codemirror } from 'vue-codemirror'
require('codemirror/mode/python/python.js')
require('codemirror/addon/fold/foldcode.js')
require('codemirror/addon/fold/foldgutter.js')
require('codemirror/addon/fold/brace-fold.js')
require('codemirror/addon/fold/xml-fold.js')
require('codemirror/addon/fold/indent-fold.js')
require('codemirror/addon/fold/markdown-fold.js')
require('codemirror/addon/fold/comment-fold.js')
export default {
  data() {
    return {
      editor: null,
      running: false,
      item: {},
      tempFalse: false,
      code: 'const a = 10',
      cmOptions: {
        mode: {
          name: 'vue',
          globalVars: true
        },
        extraKeys: { 'Alt-/': 'autocomplete' },
        selectionPointer: true,
        theme: 'idea',
        indentUnit: 2,
        lineWrapping: true,
        matchBrackets: true,
        styleActiveLine: true,
        smartIndent: true,
        tabSize: 2,
        indentWithTabs: true,
        lineNumbers: true,
        autofocus: true
      },
      codemirrorData: ''
    }
  },
  component: {
    codemirror
  },
  created() {},
  methods: {
    changes(data) {},
    onResize() {
      if (this.editor) {
        this.editor.setSize('99%', '100%')
        this.editor.setSize('99%', this.$el.clientHeight - 165 + 'px')
      }
    },
    run() {
      let self = this
      let value = this.editor.getValue()
      self.running = false
      Vue.component('editorvalue', function(resolve, reject) {
        let dom = document.createElement('div')
        dom.innerHTML = value
        let scripx = 'scripx=' + dom.getElementsByTagName('script')[0].innerText
        let fun = eval
        try {
          scripx = fun(scripx.replace('export default()', ''))
        } catch (e) {
          self.$q.notify({
            timeout: 2000,
            icon: 'fas fa-warning',
            type: 'negative',
            message: '代码有误: ' + e.message,
            position: 'top'
          })
          console.error(e)
          console.dir(e.stack)
          return
        }

        self.$q.notify({
          timeout: 2000,
          icon: 'fas fa-angle-right',
          type: 'positive',
          message: '运行成功',
          position: 'top'
        })
        dom = dom.getElementsByTagName('template')[0].innerHTML.trim()
        try {
          resolve(
            Object.assign(
              {
                template: dom
              },
              scripx
            )
          )
        } catch (e) {
          self.$q.notify({
            timeout: 2000,
            icon: 'fas fa-warning',
            type: 'negative',
            message: '代码有误: ' + e.message,
            position: 'top'
          })
          console.error(e)
          console.dir(e.stack)
        }
      })
      self.running = true
      // Vue.component('editorvalue', () => {
      //   import('statics/editor/value/editing.vue')
      // })
    }
  },
  mounted() {
    this.editor = this.$refs.myCm.codemirror
    this.onResize()
    // this.$store.state.code.editingitem
    // 读取如果是新的就读取模板或者读取cookie// 防止刷新后数据丢失
    let editingitem = this.$store.state.code.editingitem
    if (editingitem) {
      Cookies.set('editingitem', editingitem)
      // 在数据库读取
    } else {
      // 在数据库读取默认模板
      if (this.tempFalse /* Cookies.get('editingitem') */) {
        // editor.setValue(Cookies.get('editingitem'))
      } else {
        let template = '<template>\n  <div></div>\n</template>\n\n<script>\n {\n\tdata() {\n\t\treturn {}\n\t},\n\tmethods:{}\n }\n</' + 'script>\n\n<style lang="less">\n</style>'
        this.codemirrorData = template
        // editor.setValue(template)
        Cookies.set('editingitem', {
          content: template
        })
      }
    }
  }
}
</script>

<style lang="less" >
.scriptbox,
.CodeMirror {
  border-radius: 4px;
  font-size: 12pt;
  box-shadow: 0px 0px 16px #bbb;
}
.android-preview {
  width: 467px;
  height: 750px;
  background: center top no-repeat url(/statics/images/devices-sprite.jpg);
  background-size: 467px;
  margin: 5px auto 0;
  position: relative;
  top: -48px;
  //top: 60px;
  //left: 66%;
  .previewframe {
    background: rgba(255, 255, 255, 1);
    position: relative;
    top: 80px;
    left: 82px;
    height: 72%;
    border-radius: 5px;
    width: 307px;
  }
}
</style>
