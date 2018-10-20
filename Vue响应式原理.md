### Vue响应式原理

想要实现这样的功能：
```js
let price = 5
let quantity = 2
let totalPrice = price * quantity
console.log(totalPrice) // 10

price = 10 // 修改data
console.log(totalPrice) // 如果是响应式，就应该是20
```

现在就是来实现这样的响应式功能

Observer类
- 作用：给对象的属性添加getter setter，依赖收集(dep)、派发更新(notify)
- 如果属性值是数组，走observerArray
  - 循环数组每一项，执行observer
- 如果属性值是对象，走walk
  - Object.key
  - 循环每个key，执行defineReactive
- defineReactive
  - 递归给每个属性添加getter setter，在基础功能之上添加一些逻辑
  - getter中执行dep.depend()
  - setter中执行dep.notify()
    - 给newVal执行observe()，添加观察者
      - observe函数通过判断属性是否有__ob__，有则返回；没有则执行new Observer(newVal)
- 这个类主要就是通过Object.definePropety给对象属性添加getter setter，在其中进行依赖收集和派发更新
