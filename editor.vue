<template>
  <div>
    <button @click="$emit('endEdit')" @change="change">Save!</button>
    <v-jsoneditor v-model="data" :plus="true" height="400px" @input="change" />
    <editor ref="editor" v-bind="$attrs" v-on="$listeners" @change="change" />
  </div>
</template>
<script>
import 'codemirror/lib/codemirror.css'
import '@toast-ui/editor/dist/toastui-editor.css'
import * as matter from 'gray-matter'

const editor = process.server ? { render () { return null } } : require('@toast-ui/vue-editor').Editor
const VJsoneditor = process.server ? { render () { return null } } : require('v-jsoneditor/src/VueJsoneditor.vue').default

export default {
  components: {
    editor,
    VJsoneditor
  },
  props: {
    value: {
      type: String,
      required: true

    }
    // isEditing: Boolean
  },
  data () {
    return {
      file: '',
      data: {},
      lock: false
    }
  },
  watch: {
    value: {
      handler: 'valueChanged',
      immediate: true
    }
  },
  methods: {
    change () {
      if (process.server) { return }
      if (!this.lock) {
        this.file = this.$refs.editor.invoke('getMarkdown')
        this.$emit('input', matter.stringify(this.file, this.data))
      }
    },
    valueChanged () {
      if (process.server) { return }
      if (!this.value) { return }

      const { data, content } = matter(this.value)

      if (content !== this.file || data !== this.data) {
        this.lock = true

        this.file = content
        this.data = data

        this.$refs.editor.invoke('setMarkdown', this.file)

        this.lock = false
      }
    }
  }
}
</script>
