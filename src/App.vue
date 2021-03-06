<template>
  <div id="app">
    <h1>Vue Template Explorer (Vue version: {{version}})</h1>
    <label class="with-toggle">
      <input type="checkbox" v-model="stripWith">
      Strip with?
    </label>
    <div class="main">
      <codemirror
        :value="input"
        :options="inputOptions"
        @input="onInput">
      </codemirror>
      <codemirror
        :value="output.code"
        :options="outputOptions">
      </codemirror>
      <pre class="error" v-if="output.errors.length"><div v-for="e in output.errors">{{e}}</div></pre>
    </div>
  </div>
</template>

<script>
import 'codemirror/mode/vue/vue.js'
import 'codemirror/lib/codemirror.css'
import 'codemirror/theme/base16-dark.css'
import 'codemirror/theme/base16-light.css'

import { compile } from 'vue-template-compiler/build.js'
import { codemirror } from 'vue-codemirror'
import debounce from 'lodash.debounce'
import beautify from 'js-beautify'
import stripWith from 'vue-template-es2015-compiler'

export default {
  name: 'app',
  components: { codemirror },
  data () {
    return {
      input: '',
      version: VUE_VERSION,
      stripWith: false,
      inputOptions: {
        tabSize: 2,
        mode: 'text/html',
        theme: 'base16-light',
        lineNumbers: true,
        line: true
      },
      outputOptions: {
        tabSize: 2,
        mode: 'text/javascript',
        readOnly: true,
        theme: 'base16-dark'
      }
    }
  },
  computed: {
    output () {
      if (!this.input.trim()) {
        return { errors: [], code: '' }
      }
      const res = compile(this.input, { preserveWhitespace: false })
      if (!res.errors.length) {
        let code = `function render () {${res.render.toString()}}`
        if (this.stripWith) {
          code = stripWith(code)
        }
        return {
          errors: [],
          code: beautify(code, { indent_size: 2 })
        }
      } else {
        return {
          errors: res.errors,
          code: ''
        }
      }
    }
  },
  mounted () {
    const hashInput = window.location.hash.slice(1)
    this.input = hashInput
      ? decodeURIComponent(hashInput)
      : `<div id="app">{{ msg }}</div>`
    this.setHeight()
    window.addEventListener('resize', this.setHeight)
  },
  methods: {
    onInput: debounce(function (input) {
      this.input = input
      window.location.hash = encodeURIComponent(input)
    }, 500),

    setHeight () {
      const mirrors = [...this.$el.querySelectorAll('.CodeMirror')]
      const top = mirrors[0].getBoundingClientRect().top
      const height = window.innerHeight - top
      mirrors.forEach(m => m.style.height = height + 'px')
    }
  }
}
</script>

<style>
body {
  font-family: Menlo, Consolas, monospace;
  font-size: 14px;
  color: #333;
  margin: 0;
}

h1 {
  padding: 0 10px;
  font-size: 1.2em;
}

.main {
  position: relative;
  height: 100%;
  display: flex;
}

.vue-codemirror {
  line-height: 1.4em;
  width: 50%;
}

.error {
  position: absolute;
  z-index: 999;
  top: 0;
  right: 0;
  left: 50%;
  background-color: #f33;
  font-size: .9em;
  color: #fff;
  font-family: Menlo;
  padding: 1em;
  margin: 0;
}

.with-toggle {
  position: fixed;
  top: 1em;
  right: 15px;
}
</style>
