---
title: 一文彻底搞懂slot
tags:
  - null
  - null
categories:
  - null
  - null
date: 2021-06-03 10:09:29
mp3:
cover:
---
slot具体用法，以及渲染函数和jsx语法
<!-- more -->

什么时候会用到插槽？

有了`props`，为什么还需要`slot`插槽?

插槽更灵活，即插即用，不占空间

## 模板语法

#### 具名插槽

假如有个子组件如下

~~~HTML
<div>
  <slot></slot>
</div>
~~~

基础用法，在父组件中使用子组件`child`的时候，在子组件标签内部的内容会被视为插槽的内容。如下，child开始标签和结束标签之间所有内容，会被当做默认插槽的内容，插入到`<slot></slot>`中

~~~HTML
<child>
  <h1>这是一个插槽</h1>
  好的
</child>
~~~

但是，我们也可以给插槽命名，这时候在父组件中，通过`slot="xxx"`，就可以将内容插入对应名字的插槽内，子组件对应如下

~~~HTML
<div>
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
~~~

此时，子组件有三个插槽，两个具名插槽`header`和`footer`，还有一个默认插槽。对应父组件的如何运用这些插槽，如下写法：

~~~HTML
<child>
  <template slot="header">
    <h1>这是头部</h1>
  </template>
  <template>这是内容</template>
  <template slot="footer">
    <h1>这是尾部</h1>
  </template>
</child>
~~~

`vue 2.6`版本开始语法有所更新，通过`v-slot:xxx`的方式，对插槽进行命名

~~~HTML
<template v-slot:header>
  <h1>这是一个头部</h1>
</template>
<template v-slot> <!-- 可以使用v-slot:default 也可以省略v-slot，三种方式都代表默认插槽 -->
  <p>这是内容</p>
</template>
<template v-slot:footer>
  <h1>这是一个尾部</h1>
</template>
~~~

> 注意 **`v-slot` 只能添加在 `<template>` 上**（有一种特殊情况）

> 当被提供的内容只有默认插槽时，组建的标签才可以被当作插槽的模板来使用。这样我们可以把`v-slot`直接用在组件上：

~~~HTML
<current-user v-slot:default="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
~~~



通过上面的写法，父级可以非常自由地在子组件中插入对应的内容，但是如果父级想要在插槽中访问子级的作用域时，该如何操作呢？`vue`给我们提供了作用域插槽

#### 插槽作用域

假如有如下子组件，通过`v-bind:foo`在插槽上绑定了一个`foo`的值

~~~HTML
<div>
  <slot v-bind:foo="foo" :user="user"></slot>
</div>
~~~

**废弃的`slot-scope`语法**

在父组件中，则可以通过任意一个取名的参数接受插槽传递的`props`，比如使用`slotProps`接受子组件默认插槽传递的所有props属性，所以`slotPoops`包括两个属性`foo`和`user`

~~~html
<template slot-scope="slotProps">
  <p>{{slotProps.foo}}</p>
  <p>{{slotProps.user}}</p>
</template>
~~~

也支持解构的方式来读取子级传递过来的参数

~~~HTML
<template slot-scope="{foo, user}">
  <p>{{foo}}</p>
  <p>{{user}}</p>
</template>
~~~

**`vue 2.6.x`新语法**

新语法，不再将具名插槽和插槽作用域，分成两个不同的属性，而是通过指令`v-slot`将二者合二为一，具体如何使用，见下面的示例

~~~HTML
<slot name="footer" :user="fullName" :foo="foo1"></slot> <!-- child的插槽 -->

<!-- 父级 -->
<template v-slot:footer="{foo, user}">
  <h1>{{foo.bar}}</h1>
  <h1>{{user}}</h1>
</template>
~~~

新语法，看上去比废弃的属性更为直观

`v-slot`缩写为`#`

~~~html
<template #footer="{foo, user}">
  <h1>{{foo.bar}}</h1>
  <h1>{{user}}</h1>
</template>
~~~

#### 具体示例

> **插槽 prop 允许我们将插槽转换为可复用的模板，这些模板可以基于输入的 prop 渲染出不同的内容。**这在设计封装数据逻辑同时允许父级组件自定义部分布局的可复用组件时是最有用的。

实现一个官方的例子`todo-list`组件

~~~HTML
<template>
 <ul>
  <li
    v-for="todo in filteredTodos"
    v-bind:key="todo.id"
  >
    <!--
    我们为每个 todo 准备了一个插槽，
    将 `todo` 对象作为一个插槽的 prop 传入。
    -->
    <slot name="todo" v-bind:todo="todo">
      <!-- 后备内容 -->
      {{ todo.text }}
    </slot>
  </li>
</ul>
</template>

<script>
  export default {
    props: {
      todos: {
        type: Array,
        default: () => [],
      }
    },
    computed: {
      filteredTodos() {
        return this.todos
      }
    }
  }
</script>

<style lang="scss" scoped>

</style>
~~~

在父级引用

~~~HTML
<template>
<todo-list v-bind:todos="todos">
  <template v-slot:todo="{ todo }">
    <span v-if="todo.isComplete">✓</span>
    {{ todo.text }}
  </template>
</todo-list>
</template>

<script>
import TodoList from './TodoList'
export default {
  components: {
    TodoList
  },
  data() {
    return {
      todos: [
        {
          text: '吃饭',
          isComplete: false,
        },
        {
          text: '洗澡',
          isComplete: true
        }
      ]
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

~~~

## 渲染函数中的插槽

组件的本质是渲染函数，渲染函数产出vnode，插槽的本质实际上也是产出vnode的一个函数

可以通过`this.$slots`访问静态插槽的内容，每个插槽都是一个VNode数组：

~~~js
render: function (createElement) {
  // `<div><slot></slot></div>`
  return createElement('div', this.$slots.default)
}
~~~

也可以通过`this.$scopedSlots`访问作用域插槽，每个作用域插槽都是一个返回若干VNode的函数

~~~js
props: ['message'],
render: function (createElement) {
  // `<div><slot :text="message"></slot></div>`
  return createElement('div', [
    this.$scopedSlots.default({
      text: this.message
    })
  ])
}
~~~

如果要用渲染函数向子组件中传递作用域插槽，可以利用 VNode 数据对象中的 `scopedSlots` 字段

~~~js
render: function (createElement) {
  // `<div><child v-slot="props"><span>{{ props.text }}</span></child></div>`
  return createElement('div', [
    createElement('child', {
      // 在数据对象中传递 `scopedSlots`
      // 格式为 { name: props => VNode | Array<VNode> }
      scopedSlots: {
        default: function (props) {
          return createElement('span', props.text)
        }
      }
    })
  ])
}
~~~

将上面的`todo-list`组件改写成渲染函数形式

首先是子组件

~~~js
render(h) {
  return h('ul', [
    ...this.filteredTodos(i => {
      return h('li', {
        key: i.id,
      }, [
        this.$scopedSlots.todo ? this.$scopedSlots.todo({
          todo: i
        }) : i.text
      ])
    })
  ])
}
~~~

然后是父组件

~~~js
render(h) {
  return h('todo-list', {
    props: {
      todos: this.todos,
    },
    scopedSlots: {
      todo: ({todo}) => {
        return [
          todo.isComplete ? h('span', '✓') : '',
          todo.text
        ]
      }
    }
  })
}
~~~

## Jsx语法

可以看到使用渲染函数来书写十分繁琐，那有没有简便一点的方法呢？有，那就是`Jsx`语法

同样地，我们将`todoList`用`Jsx`改造一下

子组件

~~~js
render() {
  return (
    <ul>
    	{
        this.filteredTodos.map(todo => {
         return <li key={todo.id}>
          {this.$scopedSlots.todo ? this.$scopedSlots.todo({todo}) : todo.text }
      	</li>})
			}
  	</ul>
)}
~~~

父组件

~~~js
render() {
    return (
      <todo-list todos={this.todos} scopedSlots={{
        todo: ({todo}) => {
          return <div>
            {todo.isComplete ? <span>✓</span> : ''}
            {todo.text}
          </div>   
        }
      }}></todo-list>
    )
  }
~~~



[如何理解Vue.js的组件中的slot? - HcySunYang的回答 - 知乎 ](https://www.zhihu.com/question/37548226/answer/609289491)


