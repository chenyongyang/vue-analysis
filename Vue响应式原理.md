### Vue响应式原理

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


- getter
  - 依赖收集
    - Dep
      - dep中定义了一些方法，例如addSub removeSub depend notify
      - Dep.target
      - dep是对watcher的管理，无论是target，还是方法所要处理的对象，都是Watcher类型

- Watcher
  - 定义的属性：deps newDeps depIds newDepIds(用了Set这一数据结构来存储，保证不可重复)
  - 定义的方法：get（将所有打上target标记的属性收集起来） addDep cleanupDeps

当mount组件时，执行mountComponent函数，同时将updateComponent传入Watcher，实例化一个Watcher

watcher在构造函数阶段执行get方法，get方法主要将当前的Watcher实例作为参数传递给pushTarget

pushTarget是dep的一个方法，它会将当前的Watcher设置为唯一的Watcher，也就是Dep.target

接下来就是针对当前这个Watcher，进行一些dep操作

执行Watcher的getter方法，这个方法是外界传递过来的，也就是它的第二个参数，那哪里调用了new Watcher呢？

就是挂载组件时，此时传入的getter方法是
```js
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```
可以看到执行了_render方法，这个方法返回VNode，同时对数据对象进行访问，也就会触发getter

在数据对象的getter中，主要做了dep.depend,依赖收集这件事，这个方法定义在Dep中，执行的是当前Dep.target.addDep方法，之前Watcher初始化时，已经将Dep.target赋值为当前这个Watcher了，addDep中也执行了addSub，让其子对象属性也能订阅到数据的变化


