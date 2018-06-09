<h1 align="center">Fastify</h1>

## 插件
Fastify 允许用户通过插件的方式扩展自身的功能.
一个插件可以是一组路由，一个服务器[装饰器](https://github.com/fastify/docs-chinese/blob/master/docs/Decorators.md)或者其他任意的东西. 在使用一个或者许多插件时，只需要一个 API `register`.<br>

默认, `register` 会创建一个 *新的作用域( Scope )*, 这意味着你能够改变 Fastify 实例(通过`装饰`), 这个改变不会反映到当前作用域, 只会影响到子作用域. 这样可以做到插件的*封装*和*继承*, 我们创建了一个*无回路有向图*(DAG), 因此不会有交叉依赖的问题.

你已经在[起步](https://github.com/fastify/docs-chinese/blob/master/docs/Getting-Started.md#register)部分很直观的看到了怎么使用这个 API.
```
fastify.register(plugin, [options])
```

<a name="plugin-options"></a>
### 插件选项
`fastify.register` 可选参数列表支持一组预定义的 Fastify 可用的参数, 除了当插件使用了 [fastify-plugin](https://github.com/fastify/fastify-plugin). 选项对象会在插件被调用传递进去, 无论这个插件是否用了 fastify-plugin. 当前支持的选项有:

+ [`日志级别`](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md#custom-log-level)
+ [`前缀`](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md#route-prefixing-options)

Fastify 有可能在将来会直接支持其他的选项. 因此为了避免冲突, 插件应该考虑给选项加入命名空间. 举个例子, 插件 `foo` 可以像以下代码一样注册:

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

<a name="route-prefixing-option"></a>
#### 路由前缀选项
如果你传入以 `prefix`为 key , `string` 为值的选项, Fastify 会自动为这个插件下所有的路由添加这个前缀, 更多信息可以查询 [这里](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md#route-prefixing).<br>
注意如果使用了 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 这个选项不会起作用.

<a name="error-handling"></a>
#### 错误处理
错误处理是由 [avvio](https://github.com/mcollina/avvio#error-handling) 解决的.<br>
一个通用的原则, 我们建议在 `register` 回调中处理错误, 不然服务器就不会启动, 并且在 `listen` 回调中有一个没有处理的错误.

<a name="create-plugin"></a>
### 创建插件
创建插件非常简单, 你只需要创建一个方法, 这个方法接收三个参数: `fastify` 实例, 选项( Options )对象 和 `next` 回调.<br>
例子:
```js
module.exports = function (fastify, opts, next) {
  fastify.decorate('utility', () => {})

  fastify.get('/', handler)

  next()
}
```
You can also use `register` inside another `register`:
```js
module.exports = function (fastify, opts, next) {
  fastify.decorate('utility', () => {})

  fastify.get('/', handler)

  fastify.register(require('./other-plugin'))

  next()
}
```
有时候, 你需要知道什么时候服务器会关闭, 举个例子你要关闭数据库的连接. 要知道什么时候服务器关闭, 你可以用 [`'onClose'`](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md#on-close) 钩子.

别忘了 `register` 会创建一个新的 Fastify 作用域, 如果你不需要, 阅读下面的章节.

<a name="handle-scope"></a>
### 处理作用域
如果你只用 `register` 通过[`装饰`](https://github.com/fastify/docs-chinese/blob/master/docs/Decorators.md)扩展服务器的功能, 你需要自己告诉 Fastify 不要创建新的上下文, 不然你的改动不会影响其他的作用域中的用户.

你有两种方式告诉 Fastify 避免创建新的上下文:
- 使用 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 模块 
- 使用 `'skip-override'` 隐藏属性

我们建议使用 `fastify-plugin` 模块, 因为它是专门用来为你解决这个问题, 并且你可以传一个能够支持的 Fastify 版本范围的参数.
```js
const fp = require('fastify-plugin')

module.exports = fp(function (fastify, opts, next) {
  fastify.decorate('utility', () => {})
  next()
}, '0.x')
```
参考 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 文档了解更多这个模块.

如果你不用 `fastify-plugin` 模块, 可以使用 `'skip-override'` 隐藏属性, 但我们不推荐这么做. 如果将来 Fastify API 改变了, 你需要去更新你的模块, 如果使用 `fastify-plugin`, 你可以对向后兼容放心.
```js
function yourPlugin (fastify, opts, next) {
  fastify.decorate('utility', () => {})
  next()
}
yourPlugin[Symbol.for('skip-override')] = true
module.exports = yourPlugin
```
