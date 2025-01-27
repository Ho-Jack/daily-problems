Vue的生命周期

- **beforeCreate（初始化界面前） **
- **created（初始化界面后）**
- **beforeMount（渲染dom前）**
- **mounted（渲染dom后）**
- **beforeUpdate（更新数据前）**
- **updated（更新数据后）**
- **beforeDestroy（卸载组件前）**
- **destroyed（卸载组件后）**

`beforeCreate` ：初始化了部分参数，如果有相同的参数，做了参数合并，执行 `beforeCreate` ；this指向被创建的实例

`created` ：初始化了 `Inject` 、`Provide` 、 `props` 、`methods` 、`data` 、`computed` 和 `watch`，执行 `created` ；
                    未挂载dom，可对data进行操作，操作dom需放到nextTick中

`beforeMount` ：检查是否存在 `el` 属性，存在的话进行渲染 `dom` 操作，执行 `beforeMount` ；

`mounted` ：实例化 `Watcher` ，渲染 `dom`，执行 `mounted` ；
     完成挂载dom和渲染，可以对dom进行操作，并获取到dom节点，可以发起后端请求拿到数据

`beforeUpdate` ：在渲染 `dom` 后，执行了 `mounted` 钩子后，在数据更新的时候，执行 `beforeUpdate` ；

`updated` ：检查当前的 `watcher` 列表中，是否存在当前要更新数据的 `watcher` ，如果存在就执行 `updated` ；

`beforeDestroy` ：检查是否已经被卸载，如果已经被卸载，就直接 `return` 出去，否则执行 `beforeDestroy` ；								可以销毁定时器

`destroyed` ：把所有有关自己痕迹的地方，都给删除掉


```js
let vm = new Vue({
  el: '#app',
  data: {
    message: 1
  },
  template: '<div id="app"><p>{{message}}</p></div>',
  beforeCreate() {
    console.log('调用了beforeCreate')
    console.log(this.message)
    console.log(this.$el)
  },
  created() {
    console.log('调用了created')
    console.log(this.message)
    console.log(this.$el)
  },
  beforeMount() {
    console.log('调用了beforeMount')
    console.log(this.message)
    console.log(this.$el)
  },
  mounted() {
    console.log('调用了mounted')
    console.log(this.message)
    console.log(this.$el)
  },
  beforeUpdate() {
    console.log('调用了beforeUpdate')
    console.log(this.message)
    console.log(this.$el)
  },
  updated() {
    console.log('调用了updated')
    console.log(this.message)
    console.log(this.$el)
  },
  beforeDestory() {
    console.log('调用了beforeDestory')
    console.log(this.message)
    console.log(this.$el)
  },
  destoryed() {
    console.log('调用了Destoryed')
    console.log(this.message)
    console.log(this.$el)
  }
})

vm.message = 2
```

输出结果：

```js
// 调用了beforeCreate
// undefined
// undefined


// 调用了created
// 1
// undefined


// 调用了beforeMount
// 1
// <div></div>


// 调用了mounted
// 1
// <div id="app"><p>1</p></div>


// 调用了beforeUpdate
// 2
// <div id="app"><p>2</p></div>


// 调用了updated
// 2
// <div id="app"><p>2</p></div>
```

### 第一次页面加载会触发哪几个钩子

- 会触发beforeCreate, created, beforeMount, mounted

### created和mounted的区别

- created:在模板渲染成html前调用，即通常初始化某些属性值，然后再渲染成视图。
- mounted:在模板渲染成html后调用，通常是初始化页面完成后，再对html的dom节点进行一些需要的操作