# Vue 逻辑复用 内置组件补漏

## 逻辑复用

> 组合式函数
>
> 自定义指令
>
> 插件

### 组合式函数

> 在 Vue 应用的概念中，“组合式函数”(Composables) 是一个利用 Vue 的组合式 API 来封装和复用**有状态逻辑**的函数

#### 组合函数约定和实践

- 命名

  - 组合式函数约定用驼峰命名法命名，并以“use”作为开头。

- 输入参数

  - 尽管其响应性不依赖 ref，组合式函数仍可接收 ref 参数。如果编写的组合式函数会被其他开发者使用，你最好在处理输入参数时兼容 ref 而不只是原始的值。[`unref()`](https://cn.vuejs.org/api/reactivity-utilities.html#unref) 工具函数会对此非常有帮助：

- 返回值

- 副作用

  - 如果使用到**服务端渲染**，请确保在组件挂载后才调用的生命周期钩子中执行 DOM 相关的副作用，例如：`onMounted()`。这些钩子仅会在浏览器中被调用，因此可以确保能访问到 DOM。

  - 确保在 `onUnmounted()` 时清理副作用。举例来说，如果一个组合式函数设置了一个事件监听器，它就应该在 `onUnmounted()` 中被移除 (就像我们在 `useMouse()` 示例中看到的一样)。当然也可以像之前的 `useEventListener()` 示例那样，使用一个组合式函数来自动帮你做这些事

- 使用限制

  - 组合式函数在 `<script setup>` 或 `setup()` 钩子中，应始终被**同步地**调用。在某些场景下，你也可以在像 `onMounted()` 这样的生命周期钩子中使用他们。

### 自定义指令

> 自定义指令 可以全局注册 和局部注册
>
> - 全局注册可以 在 app.directive('focus',{/*....*/} 中)
> - 局部注册可以直接写在 setup 中  const vFocus={.....}

#### 指令钩子

~~~JavaScript
const myDirective = {
  // 在绑定元素的 attribute 前
  // 或事件监听器应用前调用
  created(el, binding, vnode, prevVnode) {
    // 下面会介绍各个参数的细节
  },
  // 在元素被插入到 DOM 前调用
  beforeMount(el, binding, vnode, prevVnode) {},
  // 在绑定元素的父组件
  // 及他自己的所有子节点都挂载完成后调用
  mounted(el, binding, vnode, prevVnode) {},
  // 绑定元素的父组件更新前调用
  beforeUpdate(el, binding, vnode, prevVnode) {},
  // 在绑定元素的父组件
  // 及他自己的所有子节点都更新后调用
  updated(el, binding, vnode, prevVnode) {},
  // 绑定元素的父组件卸载前调用
  beforeUnmount(el, binding, vnode, prevVnode) {},
  // 绑定元素的父组件卸载后调用
  unmounted(el, binding, vnode, prevVnode) {}
}
~~~

- el：指令绑定到的元素DOM
- binding：是一个对象
  - value ：是传递指令的值，如 v-focus="3" ，则这个值是2
  - oldValue：之前的值，仅在 `beforeUpdate` 和 `updated` 中可用。无论值是否更改，它都可用。
  - `arg`：传递给指令的参数 (如果有的话)。例如在 `v-my-directive:foo` 中，参数是 `"foo"`。
  - `modifiers`：一个包含修饰符的对象 (如果有的话)。例如在 `v-my-directive.foo.bar` 中，修饰符对象是 `{ foo: true, bar: true }`。
  - `instance`：使用该指令的组件实例。
- `vnode`：代表绑定元素的底层 VNode。
- `prevNode`：之前的渲染中代表指令所绑定元素的 VNode。仅在 `beforeUpdate` 和 `updated` 钩子中可用。

 #### 简化形式

对于自定义指令来说，一个很常见的情况是仅仅需要在 `mounted` 和 `updated` 上实现相同的行为，除此之外并不需要其他钩子。这种情况下我们可以直接用一个函数来定义指令

~~~html
<div v-color="color"></div>

<script>
  app.directive('color', (el, binding) => {
  // 这会在 `mounted` 和 `updated` 时都调用
  el.style.color = binding.value
})
</script>
~~~

#### 对象字面量

如果你的指令需要多个值，你可以向他传递一个js对象字面量

  ~~~html
  <div v-demo="{ color: 'white', text: 'hello!' }"></div>
  ~~~

#### 在组件上使用

- 不推荐在组件上使用自定义指令

### 插件



