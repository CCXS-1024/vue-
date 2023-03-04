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

  我的束河