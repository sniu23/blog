# 构建父子组件过程的心得

**props down, events up**

**组件的意义在于，封装后复用。**

**父子组件有自己的作用域。**

Q： 如何切割父子组件的功能？

- 原则上不要破坏子组件业务功能的完整性
- 尽可能把可以共用的`数据`放在父组件中
  - 子组件如果需要使用，可以通过`props`来传入数据
    - 子组件如果需要`调整`数据并`回传`给父组件，传入`props`时使用`.sync`修饰符
      - 子组件更新这个值时，它需要显式地触发一个更新事件： `this.$emit('update:foo', newValue)`
- 在子组件内需要`用户触发`的`事件`就在子组件内`定义方法`去处理（同理，在父组件内用户触发的事件就在父组件内定义方法去处理。）
  - 有时候，子组件内的事件需要父组件`协调`处理
    - 第一种情况：子组件的事件改变了一些数据，父组件后续（在父组件中用户触发其他事件）需要这些数据。
      - 像前面一样，回传数据即可。
    - 第二种情况：子组件需要通知父组件事件发生，由父组件内的方法来（真正/协助）完成业务逻辑。
      - 参考下面例子，使用`v-on`来监听子组件触发的事件

```html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

```js
Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

- 插槽：自定义一些东西（样式）、或预留功能扩展位置
- watch & computed:两者都在描述**多个数据**之间的关系。（watch在讲一个数据影响其他数据的故事，computed在讲一个数据受到其他数据影响的故事）
  - 有时候，一个子组件内的数据要受到用户和父组件的双重影响，倒不如在子组件定义自己作用域的数据……
