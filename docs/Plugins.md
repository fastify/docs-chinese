<h1 align="center">Fastify</h1>

## 插件
Fastify 允许用户通过插件的方式扩展自身的功能。
一个插件可以是一组路由，一个服务器[装饰器](https://github.com/fastify/docs-chinese/blob/master/docs/Decorators.md)或者其他任意的东西。 在使用一个或者许多插件时，只需要一个 API `register`。<br>

默认, `register` 会创建一个 *新的作用域( Scope )*, 这意味着你能够改变 Fastify 实例(通过`decorate`), 这个改变不会反映到当前作用域, 只会影响到子作用域。 这样可以做到插件的*封装*和*继承*, 我们创建了一个*无回路有向图*(DAG), 因此不会有交叉依赖的问题。

你已经在[起步](https://github.com/fastify/docs-chinese/blob/master/docs/Getting-Started.md#register)部分很直观的看到了怎么使用这个 API。
```
fastify.register(plugin, [options])
```

<a name="plugin-options"></a>
### 插件选项
`fastify.register` 可选参数列表支持一组预定义的 Fastify 可用的参数, 除了当插件使用了 [fastify-plugin](https://github.com/fastify/fastify-plugin)。 选项对象会在插件被调用传递进去, 无论这个插件是否用了 fastify-plugin。 当前支持的选项有:

+ [`日志级别`](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md#custom-log-level)
+ [`日志序列化器`](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md#custom-log-serializer)
+ [`前缀`](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md#route-prefixing-options)

**注意：当使用 fastify-plugin 时，这些选项会被忽略**

Fastify 有可能在将来会直接支持其他的选项。 因此为了避免冲突, 插件应该考虑给选项加入命名空间。 举个例子, 插件 `foo` 可以像以下代码一样注册:

```js
fastify.register(require('fastify-foo'), {
  prefix: '/foo',
  foo: {
    fooOption1: 'value',
    fooOption2: 'value'
  }
})
```

如果不考虑冲突, 插件可以简化成直接接收对象参数:

```js
fastify.register(require('fastify-foo'), {
  prefix: '/foo',
  fooOption1: 'value',
  fooOption2: 'value'
})
```

`options` 参数还可以是一个在插件注册时确定的 `函数`，这个函数的第一位参数是 fastify 实例：

```js
const fp = require('fastify-plugin')

fastify.register(fp((fastify, opts, done) => {
  fastify.decorate('foo_bar', { hello: 'world' })

  done()
}))

// fastify-foo 的 options 参数会是 { hello: 'world' }
fastify.register(require('fastify-foo'), parent => parent.foo_bar)
```

传给函数的 fastify 实例是插件声明时**外部 fastify 实例**的最新状态，允许你访问**注册顺序**在前的插件通过 [`decorate`](https://github.com/fastify/docs-chinese/blob/master/docs/Decorators.md) 注入的变量。这在需要依赖前置插件对于 Fastify 实例的改动时派得上用场，比如，使用已存在的数据库连接来包装你的插件。

请记住，传给函数的 fastify 实例和传给插件的实例是一样的，不是外部 fastify 实例的引用，而是拷贝。任何对函数的实例参数的操作结果，都会和在插件函数中操作的结果一致。也就是说，如果调用了 `decorate`，被注入的变量在插件函数中也是可用的，除非你使用 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 包装了这个插件。

<a name="route-prefixing-option"></a>
#### 路由前缀选项
如果你传入以 `prefix`为 key , `string` 为值的选项, Fastify 会自动为这个插件下所有的路由添加这个前缀, 更多信息可以查询 [这里](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md#route-prefixing).<br>
注意如果使用了 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 这个选项不会起作用。

<a name="error-handling"></a>
#### 错误处理
错误处理是由 [avvio](https://github.com/mcollina/avvio#error-handling) 解决的。<br>
一个通用的原则, 我们建议在下一个 `after` 或 `ready` 代码块中处理错误, 否则错误将出现在 `listen` 回调里。

```js
fastify.register(require('my-plugin'))

// `after` 将在上一个 `register` 结束后执行
fastify.after(err => console.log(err))

// `ready` 将在所有 `register` 结束后执行
fastify.ready(err => console.log(err))

// `listen` 是一个特殊的 `ready`,
// 因此它的执行时机与 `ready` 一致
fastify.listen(3000, (err, address) => {
  if (err) console.log(err)
})
```

*async-await* 只被 `ready` 与 `listen` 支持。
```js
fastify.register(require('my-plugin'))

await fastify.ready()

await fastify.listen(3000)
```
<a name="create-plugin"></a>
### 创建插件
创建插件非常简单, 你只需要创建一个方法, 这个方法接收三个参数: `fastify` 实例、`options` 选项和 `done` 回调。<br>
例子:
```js
module.exports = function (fastify, opts, done) {
  fastify.decorate('utility', () => {})

  fastify.get('/', handler)

  done()
}
```
你也可以在一个 `register` 内部添加其他 `register`:
```js
module.exports = function (fastify, opts, done) {
  fastify.decorate('utility', () => {})

  fastify.get('/', handler)

  fastify.register(require('./other-plugin'))

  done()
}
```
有时候, 你需要知道这个服务器何时即将关闭, 例如在你必须关闭数据库连接的时候。 要知道什么时候发生这种情况, 你可以用 [`'onClose'`](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md#on-close) 钩子。

别忘了 `register` 会创建一个新的 Fastify 作用域, 如果你不需要, 阅读下面的章节。

<a name="handle-scope"></a>
### 处理作用域
如果你使用 `register` 仅仅是为了通过[`decorate`](https://github.com/fastify/docs-chinese/blob/master/docs/Decorators.md)扩展服务器的功能, 你需要告诉 Fastify 不要创建新的上下文, 不然你的改动不会影响其他作用域中的用户。

你有两种方式告诉 Fastify 避免创建新的上下文:
- 使用 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 模块 
- 使用 `'skip-override'` 隐藏属性

我们建议使用 `fastify-plugin` 模块, 因为它是专门用来为你解决这个问题, 并且你可以传一个能够支持的 Fastify 版本范围的参数。
```js
const fp = require('fastify-plugin')

module.exports = fp(function (fastify, opts, done) {
  fastify.decorate('utility', () => {})
  done()
}, '0.x')
```
参考 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 文档了解更多这个模块。

如果你不用 `fastify-plugin` 模块, 可以使用 `'skip-override'` 隐藏属性, 但我们不推荐这么做。 如果将来 Fastify API 改变了, 你需要去更新你的模块, 如果使用 `fastify-plugin`, 你可以对向后兼容放心。
```js
function yourPlugin (fastify, opts, done) {
  fastify.decorate('utility', () => {})
  done()
}
yourPlugin[Symbol.for('skip-override')] = true
module.exports = yourPlugin
```
