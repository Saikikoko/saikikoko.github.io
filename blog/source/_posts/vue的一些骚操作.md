---
title: vue开发中的一些骚操作
tags:
  - null
  - null
categories:
  - null
  - null
date: 2020-09-29 15:47:40
mp3:
cover:
---
vue2开发中一些常用的技巧
<!-- more -->

## 介绍require.context的使用

使用场景：大规模重复劳动，手动引入同一个目录下的文件或资源等

`require.context`api介绍
- directory 要扫描的目录
- useSudirectories 是否需要扫描子目录
- regExp 匹配规则

~~~js
require.context(directory, useSudirectories = false, regExp =  /^\.\//) {}
~~~

## 通过Vue的自定义指令来控制按钮的权限

`Vue.directive( id, [definition\] )`

- **参数**：
  - `{string} id`
  - `{Function | Object} [definition]`

- **使用**

~~~js
// 注册
Vue.directive('my-directive', {
  bind: function () {}, // 只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置
  inserted: function () {}, // 被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)
  update: function () {}, // 所在组件的 VNode 更新时调用
  componentUpdated: function () {}, // 指令所在组件的 VNode 及其子 VNode 全部更新后调用
  unbind: function () {} // 只调用一次，指令与元素解绑时调用
})

// 注册 (指令函数)
Vue.directive('my-directive', function () {
  // 这里将会被 `bind` 和 `update` 调用
}
~~~

[更多具体的参见vue官方文档](https://cn.vuejs.org/v2/guide/custom-directive.html)

编写一个`v-permission`指令来控制按钮的显隐，有权限的显示，否则则隐藏

结合插件的写法

~~~js
// permission
const Auth = ['admin']

const checkAuth = function(authList) {
  return Auth.some(item => authList.includes(item))
}

function install(Vue) {
 	Vue.directive('permission', {
    inserted(el, binding) {
      if(!checkAuth(binding.value)) {
        el.parentNode && el.parentNode.removeChild(el)
      }
    }
  }) 
}

export default {install}
~~~

然后在入口文件引入

~~~js
//main.js
import Permission form './permission.js'
Vue.use(Permission)
~~~

之后在`.vue`文件使用

~~~vue
<button v-permission="['user']">add</button>

<!--
<button v-permission="['admin']">add</button>
-->
~~~

## 通过.sync是子组件改变父组件通过props传进来的值

~~~vue
<!-- parent -->
<child :name.sync="name"></child>
<!-- 同 -->
<child @update:name="name"></child>

<script>
  data() {
    return {
      name: '小红'
    }
  }
</script>
~~~

子组件里面通过`this.$emit('update:name', '小明')`

~~~vue
<!-- child.vue -->
this.$emit('update.name', '小明')
~~~

这样就能使子组件改变父组件传进来的props的值

## Vue中的jsx写法

jsx让我们用更接近模板的写法，编写渲染函数

~~~jsx
render() {
  return <p>hello { this.message }</p>
}
~~~

jsx里面通过`{}`单括号包裹的形式，动态传递参数

也可以引入==组件==

~~~jsx
import oneComponent from './oneComponent'

export default {
  render() {
    return <oneComponent></oneComponent>
  }
}
~~~

#### Attributes/Props

~~~jsx
render() {
  return <input type="email" />
}
~~~

动态绑定

~~~jsx
render() {
  return <input
    type="email"
    placeholder={this.placeholderText}
  />
}
~~~

使用`...`运算符

~~~jsx
render() {
  const inputAttrs = {
    type: 'email',
    placeholder: 'Enter your email'
  }

  return <input {...{ attrs: inputAttrs }} />
}
~~~

#### Slots插槽**

命名插槽

~~~jsx
render() {
  return (
    <MyComponent>
      <header slot="header">header</header>
      <footer slot="footer">footer</footer>
    </MyComponent>
  )
}
~~~

作用域插槽

~~~jsx
render() {
  const scopedSlots = {
    header: () => <header>header</header>,
    footer: () => <footer>footer</footer>,
      default: () => <h1>content</h1>
  }

  return <MyComponent scopedSlots={scopedSlots} />
}

<!-- 渲染函数写法 -->
render(h) {
  return h(Mycomponent, {
    scopedSlots: {
      header: function() {
        return h('header', ['header'])
      },
      footer: function() {
        return h('footer', ['footer'])
      }
    }
  })
}

<!-- 模板写法 -->
<MyComponent>
	<template v-slot:header>
  	<header>header</header>
  </template>
  
  <template #footer>
  	<header>header</header>
  </template>
</MyComponent>
~~~

[扩展一下Vue的插槽内容](##Vue插槽)

#### 指令

==vue官网给出的vOn并不能用==

那么jsx如何监听事件？

~~~jsx
<p on={
    {
      click: this.add,
      change: this.change
    }
  }></p>

<p {...{
    on: {
      click: this.add,
      change: this.change
    }
  }}></p>
~~~

`vModel和domPropsInnerHTML`是有效的

~~~jsx
<input vModel={this.newTodoText} />
<p domPropsInnerHTML={html} /> <!-- v-html -->
~~~

加上修饰符

~~~jsx
<input vModel_trim={this.newTodoText} />
~~~

`v-if`和`v-for`都没有直接的指令支持，但是可以通过`if`判断或者循环语句实现相同的效果

`v-if`

~~~jsx
{this.ifTrue ? <p>true</p> : <p>false</p>}
~~~

`v-for`

~~~jsx
 <ul>
  {this.pArr.map(item => {
    return  <li>{item.name}</li>
  })}
</ul>
~~~



#### 函数式组件

将返回jsx的函数默认转换成函数式组件



## 函数式组件

当一个组件只是接受一些props参数，不需要渲染时，比如包装组件时，我们可以使用函数式组件，它的渲染开销十分小，要想实现，只需要添加`functional: true`即可，函数式组件没有`this`上下文对象，所有的参数通过`context`传递



## Vue插槽

#### 插槽命名

插槽名字默认为default，如果父组件的内容没有指定插槽，所有的内容默认插入该插槽中

~~~vue
<!-- child -->
<template>
  <div>
    <slot></slot> <!-- name 为 default-->
  </div>
</template>
~~~

#### 具名插槽

~~~vue
<template>
  <div>
    <slot name="header"></slot>
    <slot name="footer"></slot>
    <slot></slot>
  </div>
</template>
~~~

~~~vue
<!-- parent -->
<template>
  <div>
    <child>
      <template v-slot:header>
        <h1>Here might be a page title</h1>
      </template>

      <p>A paragraph for the main content.</p>
      <p>And another one.</p>

      <template v-slot:footer>
        <p>Here's some contact info</p>
      </template>
  	</child>
  </div>
</template>
~~~

#### 插槽作用域scopedSlots

scopedSlots是一个对象，对象上的属性是返回每个插槽内容的函数，函数的参数则是插槽绑定的props，

~~~vue
<!-- child -->
<template>
  <div>
    <slot name="header" :page="page"></slot>
    <slot name="footer" :page="page"></slot>
    <slot></slot>
  </div>
</template>
~~~

~~~vue
<!-- parent -->
<template>
  <div>
    <child>
      <template v-slot:header="slotProps">
        <h1>Here might be a page title</h1>
        <p>{{slotProps.page.header}}</p>
      </template>

      <p>A paragraph for the main content.</p>
      <p>And another one.</p>

      <template #footer="{page}"> <!-- 缩写 + 解构-->
        <p>Here's some contact info</p>
        <p>{{page.footer}}</p>
      </template>
  	</child>
  </div>
</template>
~~~


