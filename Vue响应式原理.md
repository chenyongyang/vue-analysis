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

