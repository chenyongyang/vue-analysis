想要实现这样的功能：
```js
let price = 5
let quantity = 2
let total = price * quantity
console.log(total) // 10

price = 10 // 修改data
console.log(total) // 如果是响应式，就应该是20
```

现在就是来实现这样的响应式功能

既然要实现改变单价或质量，总价格就会更新，本质上就是当数据改变时，再次执行计价

因此我们应该把这个计价封装成一个函数，以便之后调用
```js
target = () => {
    total = price * quantity
}
```

可能有多个对数据的处理函数，因此我们定义一个数组来存储
```js
storge = []
storge.push(target)
```
以后我们想要调用计较函数，就可以这样
```js
storge.forEach(run => run())
```
完整代码如下
```js
let price = 5
let quantity = 2
let total = 0
let target = null
let storge = [] // 存储target

function recode(){ // 将target记录起来
    storge.push(target)
}

function replay(){
    storge.forEach(run => run())
}

target = () => { total = price * quantity}

recode()
target()

price = 10
console.log(total) // 10
replay()
console.log(total) // 20
```

我们现在有了一个target，同时将target存储在storge列表中，通过replay方法通知target。这一切都可以封装成一个类，这是依赖类就出现了。
```js
class Dep{
    constructor(){
        this.subscribers = []; // 代替storge
    }
    depend(){ // 代替recode
        if(target && !this.subscribers.includes(target)){
            this.subscribers.push(target)
        }
    }
    notify(){ // 代替replay
        this.subscribers.forEach(sub => sub()) // run target or observer
    }
}

let dep = new Dep()

let price = 5
let quantity = 2
let total = 0
let target = () => { total = price * quantity}
dep.depend() // 添加依赖（这个依赖指的是target）
target()

console.log(total) // 10
price = 10
console.log(total) // 10
dep.notify()
console.log(total) // 20
```

现在还有一点不足：还需要去定义和调用target；我们完全可以把target的定义和调用都封装起来

我们要封装的就是创建监听更新的匿名函数的行为；这时候可以引进Watcher观察者函数
```js
function watcher(func){
    target = func;
    dep.depend();
    target();
    target = null
}
```
这样我们就把创建监听更新的匿名函数、依赖的收集都封装到一起。

将代码整合一下
```js
class Dep{
    constructor(func){
        this.subscribers = []; // 代替storge
        this.target = null;
        this.watcher(func);
    }
    depend(){ // 代替recode
        if(this.target && !this.subscribers.includes(this.target)){
            this.subscribers.push(this.target)
        }
    }
    notify(){ // 代替replay
        this.subscribers.forEach(sub => sub()) // run target or observer
    }
    watcher(func){
        this.target = func
        this.depend()
        this.target()
        this.target = null
    }
}


let price = 5;
let quantity = 2
let total = 0
let dep = new Dep(()=>{total = price * quantity});

price = 10 // data数据修改
console.log(total) // 10
dep.notify() // dep通知target更新
console.log(total) // 20
```

到此为止，当我们修改数据一修改，只要通知存储在依赖类实例的target，即可实现依赖数据的修改

但我们还需要手动执行dep.notify，我们希望数据一修改就自动执行dep.notify，而且应该是每个数据都有这个能力

我们希望当访问数据对象属性时，将target所指向的更新函数添加到subscribers中；当修改数据对象属性时，通知其依赖数据

我们需要一些方式来挂钩数据属性，使得当访问它时，能够将target推送到subscribers数组中；当它被修改时，运行存储在subscribers数组中的函数

这时候就需要Object.defineProperty()

最终代码如下：
```js
let data = {
    price: 5,
    quantity: 2
}
let target = null
class Dep{
    constructor(){
        this.subscribers = []
    }
    depend(){
        if(target && !this.subscribers.includes(target)){
            this.subscribers.push(target)
        }
    }
    notify(){
        this.subscribers.forEach(sub=>sub())
    }
}
Object.keys(data).forEach(key => {
    let value = data[key]
    let dep = new Dep()
    Object.defineProperty(data, key, {
        get(){
            dep.depend()
            return value;
        },
        set(newValue){
            value = newValue
            dep.notify()
        }
    })
})

function watcher(func){
    target = func
    target()
    target = null
}

watcher(() => {
    data.total = data.price * data.quantity
})
```

至此，我们实现了price quantity的响应式，一旦数据更改，依赖的数据也随之更改




