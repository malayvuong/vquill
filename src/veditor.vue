<template>
<div class="quill-editor">
  <slot name="toolbar"></slot>
  <div :style="{
    height: height + 'px', 'min-height': '100px'
  }" ref="editor"></div>
</div>
</template>

<script>
const Quill = require('quill')

import 'quill/dist/quill.core.css';
import 'quill/dist/quill.snow.css';
import './css/quill.bubble.css';

// export
export default {
  name: 'vquill',
  props: {
    //  v-model value
    value: {type: String, default: ''},
    //  disabled status
    disabled: { type: Boolean, default: false },
    //  Toolbar needed
    toolbar: {
      type: Array,
      default: function() {
        return [
          ['bold', 'italic', 'underline', 'strike'],
          ['blockquote', 'code-block'],
          [{ 'header': 1 }, { 'header': 2 }],
          [{ 'list': 'ordered' }, { 'list': 'bullet' }],
          [{ 'script': 'sub' }, { 'script': 'super' }],
          [{ 'indent': '-1' }, { 'indent': '+1' }],
          [{ 'direction': 'rtl' }],
          [{ 'size': ['small', false, 'large', 'huge'] }],
          [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
          [{ 'color': [] }, { 'background': [] }],
          [{ 'font': [] }],
          [{ 'align': [] }],
          ['clean'],
          ['link', 'image', 'video']
        ]
      },
    },
    //  Textarea placeholder
    placeholder: { type: String, default: 'Type your text content...' },
    //  Themes of Quill
    theme: { type: String, default: 'snow' },
    //  Container max-height
    height: { type: Number, default: 200 }
  },
  data() {
    return {
      innerValue: '',
      options: {
        theme: 'snow',
        boundary: document.body,
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'],
            ['blockquote', 'code-block'],
            [{ 'header': 1 }, { 'header': 2 }],
            [{ 'list': 'ordered' }, { 'list': 'bullet' }],
            [{ 'script': 'sub' }, { 'script': 'super' }],
            [{ 'indent': '-1' }, { 'indent': '+1' }],
            [{ 'direction': 'rtl' }],
            [{ 'size': ['small', false, 'large', 'huge'] }],
            [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
            [{ 'color': [] }, { 'background': [] }],
            [{ 'font': [] }],
            [{ 'align': [] }],
            ['clean'],
            ['link', 'image', 'video']
          ]
        },
        placeholder: '',
        readOnly: false
      }
    }
  },
  watch: {
    // Watch content change
    value(newVal, oldVal) {
      if (this.quill) {
        if (newVal && newVal !== this.innerValue) {
          this.innerValue = newVal
          this.quill.pasteHTML(newVal)
        } else if(!newVal) {
          this.quill.setText('')
        }
      }
    },
    // Watch disabled change
    disabled(newVal, oldVal) {
      if (this.quill) {
        this.quill.enable(!newVal)
      }
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    // Init Quill instance
    init() {
      let v = this
      if (v.$el) {
        // Set Toolbar
        if(v.toolbar && v.toolbar.length > 0) v.options.modules.toolbar = v.toolbar
        //  Set theme
        if(v.theme) v.options.theme = v.theme
        //  Set Placeholder
        if(v.placeholder) v.options.placeholder = v.placeholder

        // Instance
        v.quill = new Quill(v.$refs.editor, v.options)
        
        //  Enable editor
        v.quill.enable(false)

        // Set editor content
        if (v.value) {
          v.quill.pasteHTML(v.value)
        }

        // Disabled editor
        if (!v.disabled) {
          v.quill.enable(true)
        }

        // Mark model as touched if editor lost focus
        v.quill.on('selection-change', range => {
          if (!range) {
            v.$emit('blur', v.quill)
          } else {
            v.$emit('focus', v.quill)
          }
        })

        // Update model if text changes
        v.quill.on('text-change', (delta, oldDelta, source) => {
          let html = v.$refs.editor.children[0].innerHTML
          const quill = v.quill
          const text = v.quill.getText()
          if (html === '<p><br></p>') html = ''
          v.innerValue = html
          v.$emit('input', v.innerValue)
          v.$emit('change', { html, text, quill })
        })

        // Emit ready event
        v.$emit('ready', v.quill)
      }
    }
  },
  //  Remove Quill before destroy
  beforeDestroy() {
    this.quill = null
    delete this.quill
  },
}
</script>