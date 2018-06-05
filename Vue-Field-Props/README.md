1. 继承原生属性

当开发表单组件时，不得不解决的问题是继承原生组件的各种属性。例如封装一个 input 组件，要有原生的 placeholder 属性，那么我们的代码可能是这样：

<template>
  <div>
    <label>{{ label }}</label>
​
    <input
      @input="$emit('input', $event.target.value)"
      :value="value"
      :placeholder="placeholder">
  </div>
</template>
​
<script>
  export default {
    props: ['value', 'placeholder', 'label']
  }
</script>
但是如果需要支持其他原生属性就需要继续写模板内容：

<template>
  <div>
    <label>{{ label }}</label>
​
    <input
      @input="$emit('input', $event.target.value)"
      :value="value"
      :placeholder="placeholder"
      :maxlength="maxlength"
      :minlength="minlength"
      :name="name"
      :form="form"
      :value="value"
      :disabled="disabled"
      :readonly="readonly"
      :autofocus="autofocus">
  </div>
</template>
​
<script>
  export default {
    props: ['label', 'placeholder', 'maxlength', 'minlength', 'name', 'form', 'value', 'disabled', 'readonly', 'autofocus']
  }
</script>
如果还要设置 type，或者是要同时支持 textarea，那么重复的代码量还是很可怕的。但是换个思路，直接用 jsx 写的话或许会轻松一些：

export default {
  props: ['label', 'type', 'placeholder', 'maxlength', 'minlength', 'name', 'form', 'value', 'disabled', 'readonly', 'autofocus'],
​
  render(h) {
    const attrs = {
      placeholder: this.placeholder,
      type: this.type
      // ...
    }
​
    return (
      <div>
        <label>{ this.label }</label>
        <input { ...{ attrs } } />
      </div>
    )
  }
}
在 Vue 的 vnode 中，原生属性是定义在 data.attrs 中，所以上面 input 部分会被编译成：

h('input', {
  attrs: attrs
})
这样就完成了原生属性的传递，同理如果需要通过 type 设置 textarea，只需要加个判断设置 tag 就好了。

h(this.type === 'textarea' ? 'textarea' : 'input', { attrs })
目前为止我们还是需要定义一个 attrs 对象，但是所需要的属性其实都已经定义在了 props 中，那么能直接从 props 里拿到值岂不是更好？我们可以简单的写一个 polyfill 完成这件事。（实际上 Vue 2.2 中不需要你引入 polyfill，默认已经支持）

import Vue from 'vue'
​
Object.defineProperty(Vue.prototype, '$props', {
  get () {
    var result = {}
    for (var key in this.$options.props) {
      result[key] = this[key]
    }
    return result
  }
})
原理很简单，从 vm.$options.props 遍历 key 后从 vm 中取值，现在我们就可以直接从 vm.$props 拿到所有 props 的值了。那么我们的代码就可以改成这样：

render(h) {
  return (
    <div>
      <label>{ this.label }</label>
      <input { ...{ attrs: this.$props } } />
    </div>
  )
}
如果你留意过 Vue 文档介绍 v-bind 是可以传对象 的话，那我们的代码用 Vue 模板写的话就更简单了：

<template>
  <div>
    <label>{{ label }}</label>
    <input v-bind="$props">
  </div>
</template>