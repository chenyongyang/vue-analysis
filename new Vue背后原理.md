```html
<div id="app">
  {{ message }}
</div>
```
```js
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
以官网提供的例子来分析，vue是如何通过模板和数据渲染成真实DOM

### 一切从new Vue()说起
定位到src/core/instance/index.js
```js
// 引入了初始化、数据、渲染、事件、生命周期这些配置
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

// Vue其实是一个构造函数，且只能用new来调用，否则会得到警告
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  // 通过new调用后，执行_init这个内部方法，这个方法定义在Vue.prototype上
  this._init(options)
}

// 将这些配置注入到vue中
initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

