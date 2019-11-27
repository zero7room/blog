# 前言
call, apply, bind, 大家是不是经常使用么，有了解过其实现原理么，接下就讲解一下它们的实现!
 
# call, apply, bind
首先我们思考一下函数主要实现了哪些功能呢，皮卡丘总结了一下：
- 不传入第一个参数，那么上下文默认为`window`
- 改变了`this`指向，让新的对象可以执行该函数，并能接受参数

## call
我们首先来实现call函数
### call的代码实现
```
	Function.prototype.myCall = function(context) {
	  if (typeof this !== 'function') {
		throw new TypeError('Error')
	  }
	  context = context || window
	  context.fn = this
	  const args = [...arguments].slice(1)
	  const result = context.fn(...args)
	  delete context.fn
	  return result
	}
```
###call的实现思路
- 首先`context`为可选参数，如果不传的话默认上下文为`window`
- 接下来给`context`创建一个`fn`属性，并将值设置为需要调用的函数
- 因为`call`可以传入多个参数作为调用函数的参数，所以需要将参数剥离出来
- 然后调用函数并将对象上的函数删除

## apply
### apply的代码实现
```
	Function.prototype.myApply = function(context) {
	  if (typeof this !== 'function') {
		throw new TypeError('Error')
	  }
	  context = context || window
	  context.fn = this
	  let result
	  // 处理参数和 call 有区别
	  if (arguments[1]) {
		result = context.fn(...arguments[1])
	  } else {
		result = context.fn()
	  }
	  delete context.fn
	  return result
	}

```
### apply的实现思路
- apply 的实现与call类似，区别在于对参数的处理

## bind
### bind的代码实现
- bind 的实现对比其他两个函数略微地复杂了一点，因为 bind 需要返回一个函数，需要判断一些边界问题，以下是 bind 的实现
```
	Function.prototype.myBind = function (context) {
	  if (typeof this !== 'function') {
		throw new TypeError('Error')
	  }
	  const _this = this
	  const args = [...arguments].slice(1)
	  // 返回一个函数
	  return function F() {
		// 因为返回了一个函数，我们可以 new F()，所以需要判断
		if (this instanceof F) {
		  return new _this(...args, ...arguments)
		}
		return _this.apply(context, args.concat(...arguments))
	  }
	}

```

- 前几步和之前的实现差不多，就不赘述了
- `bind`返回了一个函数，对于函数来说有两种方式调用，一种是直接调用，一种是通过 new 的方式，我们先来说直接调用的方式
- 对于直接调用来说，这里选择了`apply`的方式实现，但是对于参数需要注意以下情况：因为 bind 可以实现类似这样的代码`f.bind(obj, 1)(2)`，所以我们需要将两边的参数拼接起来，于是就有了这样的实现`args.concat(...arguments)`
- 最后来说通过`new`的方式，在之前的章节中我们学习过如何判断`this`，对于`new`的情况来说，不会被任何方式改变`this`，所以对于这种情况我们需要忽略传入的`this`

# new
## new操作符
### new的实现思路
在调用`new`的过程中会发生以上四件事情：
1.新生成了一个对象
2.链接到原型
3.绑定 this
4.返回新对象

### new的代码实现
```
	function create() {
	  let obj = {}
	  let Con = [].shift.call(arguments)
	  obj.__proto__ = Con.prototype
	  let result = Con.apply(obj, arguments)
	  return result instanceof Object ? result : obj
	}
```


- 创建一个空对象
- 获取构造函数
- 设置空对象的原型
- 绑定 this 并执行构造函数
- 确保返回值为对象

对于对象来说，其实都是通过`new`产生的，无论是`function Foo()` 还是`let a = { b : 1 }`。
对于创建一个对象来说，更推荐使用字面量的方式创建对象（无论性能上还是可读性）。因为你使用`new Object()`的方式创建对象需要通过作用域链一层层找到`Object`，但是你使用字面量的方式就没这个问题。

```
	function Foo() {}
	// function 就是个语法糖
	// 内部等同于 new Function()
	let a = { b: 1 }
	// 这个字面量内部也是使用了 new Object()
```