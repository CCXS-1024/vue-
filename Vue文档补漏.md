# Vue文档补漏



### 事件处理

- 事件修饰符

  - .stop 用来防止事件冒泡
  - prevent 来防止标签默认行为
  - self 仅当 event.target 是元素本身时才会触发事件处理器

  ~~~html
  <!-- 修饰语可以使用链式书写 -->
  <a @click.stop.prevent="doThat"></a>
  
  
  <!-- 仅当 event.target 是元素本身时才会触发事件处理器 -->
  <!-- 例如：事件处理器不来自子元素 -->
  <div @click.self="doThat">...</div>
  
  <!-- 滚动事件的默认行为 (scrolling) 将立即发生而非等待 `onScroll` 完成 -->
  <!-- 以防其中包含 `event.preventDefault()` -->
  <div @scroll.passive="onScroll">...</div>
  ~~~

  - capture 事件捕获
  - once 事件触发一次
  - passive 饰符一般用于触摸事件的监听器，可以用来[改善移动端设备的滚屏性能](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener#使用_passive_改善滚屏性能)

- 按键修饰符
  - 在监听键盘事件时，我们经常需要检查特定的按键。Vue 允许在 `v-on` 或 `@` 监听按键事件时添加按键修饰符。
  - [请查看文档](https://cn.vuejs.org/guide/essentials/event-handling.html#accessing-event-argument-in-inline-handlers)
- 系统按键修饰符
- .exact修饰符
- 鼠标按键修饰符

### 表单绑定

- v-model不会再拼写中午的时候触发更新。如果你想在拼字的时候也出发更新需要自己监听 input 事件 和绑定value 不要使用v-model

~~~html
<input
  :value="text"
  @input="event => text = event.target.value">
~~~

- V-model 修饰符

  - .lazy 默认情况下v-model会在每次input事件后更新 加了以后则会在change后更新数据
  - .number 自动转换字符串
  - .trim 清除两端空格

 ### 生命周期

> 所用声明周期钩子都应该在 setup 中同步调用

### 监听器

> 不能直接监听响应式对象的属性值，

~~~JavaScript
watch(
  () => state.someObject,
  () => {
    // 仅当 state.someObject 被替换时触发
  }
)
~~~

- 默认情况下，用户创建监听回调，都会在Vue组件更新之前被调用。这就意味着你在监听回调中访问的DOM将是被Vue更新之前的状态
- 你需要使用flue:'post' 来访问Vue更新后的DOM

~~~JavaScript
watch(source, callback, {
  flush: 'post'
})

watchEffect(callback, {
  flush: 'post'
})
~~~

- 停止监听

  - 监听器必须是同步语句创建：如果用异步回调创建一个监听器，那么他不会绑定到组件上，你必须手动停止它防止内存泄露

  - 手动停止监听器 watch函数会返回一个函数执行函数即可

## 深入组件

> 组件分为全局组件和局部组件
>
> - 全局注册，但并没有被使用的组件无法在生产打包时被自动移除 (也叫“tree-shaking”)。如果你全局注册了一个组件，即使它并没有被实际使用，它仍然会出现在打包后的 JS 文件中。
> - 全局注册在大型项目中使项目的依赖关系变得不那么明确。在父组件中使用子组件时，不太容易定位子组件的实现。和使用过多的全局变量一样，这可能会影响应用长期的可维护性

#### Props的细节

- Props名字的格式

  - 如果一个props的名字很长，应该使用小驼峰的形式，因为它们是合法的js标识符可以在模板表达式使用
  - 但是在父向子传递参数的时候一般使用  kebab-case的 形式

  ~~~html
  <MyComponent greeting-message="hello" />
  ~~~

  - 如果你想使用一个对象绑定多个prop可以使用 v-bind不带如下

  ~~~html
  <BlogPost v-bind="post" />
  
  <BlogPost :id="post.id" :title="post.title" />
  ~~~

- 单项数据流

- props 的校验  

- Boolean类型转换

  - 为了更贴近原生 boolean attributes 的行为，声明为 `Boolean` 类型的 props 有特别的类型转换规则。以带有如下声明的 `<MyComponent>` 组件为例：

  - ~~~html
    defineProps({
      disabled: Boolean
    })
    
    <!-- 等同于传入 :disabled="true" -->
    <MyComponent disabled />
    
    <!-- 等同于传入 :disabled="false" -->
    <MyComponent />
    ~~~

### 组件事件

> 在子组件触发的事件使用驼峰命名 但是在父组件中需要使用  kebab-case 的格式监听和props的格式差不多

触发事件不仅可以传入一个函数还可以传入一个对象来进行事假校验

~~~Vue
<script setup>
const emit = defineEmits({
  // 没有校验
  click: null,

  // 校验 submit 事件  返回为true则通过 false 不通过
  submit: ({ email, password }) => {
    if (email && password) {
      return true
    } else {
      console.warn('Invalid submit event payload!')
      return false
    }
  }
})

function submitForm(email, password) {
  emit('submit', { email, password })
}
</script>
~~~

### 在组件中使用v-model

 - 默认情况下会展开成

~~~html
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
~~~

你需要再对应的子组件中手动绑定到对应的input 中

- 还用一种方式 通过计算属性来实现 子组件的绑定

~~~html
<!-- CustomInput.vue -->
<script setup>
import { computed } from 'vue'

const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])

const value = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    emit('update:modelValue', value)
  }
})
</script>
<template>
  <input v-model="value" />
</template>
~~~

#### v-model 的参数

> 默认情况下，v-model在组件上都是使用modelValue作为prop，并以update:modelValue作为对应的事件。我们可以通过给v-model指定一个参数来更改这些名字

~~~html
<MyComponent v-model:title="bookTitle" />
<!-- 默认情况下-->

子组件会拆分成
<script setup>
defineProps(['title'])
defineEmits(['update:title'])
</script>
<template>
  <input
    type="text"
    :value="title"
    @input="$emit('update:title', $event.target.value)"
  />
</template>
~~~

- 多个v-model绑定
  - 利用上面的v-model的参数可以在一子组件中绑定多个v-model 其组件上的v-model都会不同
  - 看文档上面的解释

- 处理v-model修饰符
  - 自定义修饰符
    - 可以在子组件 监听props 中 modelModifiers 这个属性默认是一个对象
    - [文档](https://cn.vuejs.org/guide/components/v-model.html)

### 事件穿透

> 事件穿透默认是 子组件的根元素上

- 事件穿透 class 和 style

  - 这两个会合并

- 禁用事件穿透

  - 如果你使用了 `<script setup>`，你需要一个额外的 `<script>` 块来书写这个选项声明

  - ~~~html
    <script>
    // 使用普通的 <script> 来声明选项
    export default {
      inheritAttrs: false
    }
    </script>
    
    <script setup>
    // ...setup 部分逻辑
    </script>
    ~~~

- 默认情况下所有穿透的信息会保存在$attrs 中

- 这个 `$attrs` 对象包含了除组件所声明的 `props` 和 `emits` 之外的所有其他 attribute，例如 `class`，`style`，`v-on` 监听器等等。
  

  - 和 props 有所不同，透传 attributes 在 JavaScript 中保留了它们原始的大小写，所以像 `foo-bar` 这样的一个 attribute 需要通过 `$attrs['foo-bar']` 来访问。
    
  - 像 `@click` 这样的一个 `v-on` 事件监听器将在此对象下被暴露为一个函数 `$attrs.onClick`。

#### 多跟节点的穿透

> 和单根节点的穿透不同 多节点 需要手动绑定对应的事件 否则控制台会给警告

不需要禁用事件穿透 多接点穿透

可以使用 useAttrs 来在js 中访问 Attributes的值

### 插槽Slots

> 当一个组件同时接收默认插槽和具名插槽时，所有位于顶级的非 `<template>` 节点都被隐式地视为默认插槽的内容 

#### 动态域名插槽

~~~ html
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>

  <!-- 缩写为 -->
  <template #[dynamicSlotName]>
    ...
  </template>
</base-layout>
~~~

#### 作用域插槽

[文档](https://cn.vuejs.org/guide/components/slots.html)

### 依赖注入

> 注入分为全局注入和全局注入
>
> 全局注册  直接在app.provide('message','hello')
