<h1 align="center">Fastify</h1>

# 插件漫游指南
首先, `不要恐慌`!

Fastify 从一开始就搭建成非常模块化的系统. 我们搭建了非常强健的 API 来允许你创建命名空间, 来添加工具方法. Fastify 创建的封装模型可以让你在任何时候将你的应用分割成不同的微服务, 而无需重构整个应用.

**内容清单**
- [注册器](#register)
- [装饰器](#decorators)
- [钩子方法](#hooks)
- [如何处理封装与分发](#distribution)
- [ESM 的支持](#esm-support)
- [错误处理](#handle-errors)
- [自定义错误](#custom-errors)
- [发布提醒](#emit-warnings)
- [开始!](#start)

<a name="register"></a>
## 注册器
就像在 JavaScript 万物都是对象, 在 Fastify 万物都是插件.<br>
你的路由, 你的工具方法等等都是插件. 无论添加什么功能的插件, 你都可以使用 Fastify 优秀又独一无二的 API: [`register`](Plugins.md).
```js
fastify.register(
  require('./my-plugin'),
  { options }
)
```
`register` 创建一个新的 Fastify 上下文, 这意味着如果你对 Fastify 的实例做任何改动, 这些改动不会反映到上下文的父级上. 换句话说, 封装!

*为什么封装这么重要?*<br>
那么, 假设你创建了一个具有开创性的初创公司, 你会怎么做? 你创建了一个包含所有东西的 API 服务器, 所有东西都在同一个地方, 一个庞然大物!<br>
现在, 你增长得非常迅速, 想要改变架构去尝试微服务. 通常这意味着非常多的工作, 因为交叉依赖和缺少关注点的分离.<br>
Fastify 在这个层面上可以帮助你很多, 多亏了封装模型, 它完全避免了交叉依赖, 并且帮助你将组织成高聚合的代码块.

*让我们回到如何正确地使用 `register`.*<br>
插件必须输出一个有以下参数的方法
```js
module.exports = function (fastify, options, done) {}
```
`fastify` 就是封装的 Fastify 实例, `options` 就是选项对象, 而 `done` 是一个在插件准备好了之后**必须**要调用的方法.

Fastify 的插件模型是完全可重入的和基于图(数据结构)的, 它能够处理任何异步代码并且保证插件的加载顺序, 甚至是关闭顺序! *如何做到的?* 很高兴你发问了, 查看下 [`avvio`](https://github.com/mcollina/avvio)! Fastify 在 `.listen()`, `.inject()` 或者 `.ready()` 被调用了之后开始加载插件.

在插件里面你可以做任何想要做的事情, 注册路由, 工具方法 (我们马上会看到这个) 和进行嵌套的注册, 只要记住当所有都设置好了后调用 `done`!
```js
module.exports = function (fastify, options, done) {
  fastify.get('/plugin', (request, reply) => {
    reply.send({ hello: 'world' })
  })

  done()
}
```

那么现在你已经知道了如何使用 `register` API 并且知道它是怎么工作的, 但我们如何给 Fastify 添加新的功能, 并且分享给其他的开发者?

<a name="decorators"></a>
## 装饰器
好了, 假设你写了一个非常好的工具方法, 因此你决定在你所有的代码里都能够用这个方法. 你改怎么做? 可能是像以下代码一样:
```js
// your-awesome-utility.js
module.exports = function (a, b) {
  return a + b
}
```
```js
const util = require('./your-awesome-utility')
console.log(util('that is ', 'awesome'))
```
现在你需要在所有需要这个方法的文件中引入它. (别忘了你可能在测试中也需要它).

Fastify 提供了一个更优雅的方法, *装饰器*.
创建一个装饰器非常简单, 只要使用 [`decorate`](Decorators.md) API:
```js
fastify.decorate('util', (a, b) => a + b)
```
现在你可以在任意地方通过 `fastify.util` 调用你的方法, 甚至在你的测试中.<br>
这里神奇的是: 你还记得之前我们讨论的封装? 同时使用 `register` 和 `decorate` 可以实现, 让我用例子来阐明这个事情:
```js
fastify.register((instance, opts, done) => {
  instance.decorate('util', (a, b) => a + b)
  console.log(instance.util('that is ', 'awesome'))

  done()
})

fastify.register((instance, opts, done) => {
  console.log(instance.util('that is ', 'awesome')) // 这里会抛错

  done()
})
```
在第二个注册器中调用 `instance.util` 会抛错, 因为 `util` 只存在第一个注册器的上下文中.<br>
让我们更深入地看一下: 当使用 `register` API 每次都会创建一个新的上下文而且这避免了上文提到的这个状况.

但是注意, 封装只会在父级和同级中有效, 不会在子级中有效.
```js
fastify.register((instance, opts, done) => {
  instance.decorate('util', (a, b) => a + b)
  console.log(instance.util('that is ', 'awesome'))

  fastify.register((instance, opts, done) => {
    console.log(instance.util('that is ', 'awesome')) // 这里不会抛错
    done()
  })

  done()
})

fastify.register((instance, opts, done) => {
  console.log(instance.util('that is ', 'awesome')) // 这里会抛错 

  done()
})
```
*PS: 如果你需要全局的工具方法, 请注意要声明在应用根作用域上. 或者你可以使用 `fastify-plugin` 工具, [参考](#distribution).*

`decorate` 不是唯一可以用来扩展服务器的功能的 API, 你还可以使用 `decorateRequest` 和 `decorateReply`.

*`decorateRequest` 和 `decorateReply`? 为什么我们已经有了 `decorate` 还需要它们?*<br>
好问题, 是为了让开发者更方便地使用 Fastify. 让我们看看这个例子:
```js
fastify.decorate('html', payload => {
  return generateHtml(payload)
})

fastify.get('/html', (request, reply) => {
  reply
    .type('text/html')
    .send(fastify.html({ hello: 'world' }))
})
```
这个可行, 但可以变得更好!
```js
fastify.decorateReply('html', function (payload) {
  this.type('text/html') // this 是 'Reply' 对象
  this.send(generateHtml(payload))
})

fastify.get('/html', (request, reply) => {
  reply.html({ hello: 'world' })
})
```

你可以对 `request` 对象做同样的事:
```js
fastify.decorate('getHeader', (req, header) => {
  return req.headers[header]
})

fastify.addHook('preHandler', (request, reply, done) => {
  request.isHappy = fastify.getHeader(request.raw, 'happy')
  done()
})

fastify.get('/happiness', (request, reply) => {
  reply.send({ happy: request.isHappy })
})
```
这个也可行, 但可以变得更好!
```js
fastify.decorateRequest('setHeader', function (header) {
  this.isHappy = this.headers[header]
})

fastify.decorateRequest('isHappy', false) // 这会添加到 Request 对象的原型中, 好快!

fastify.addHook('preHandler', (request, reply, done) => {
  request.setHeader('happy')
  done()
})

fastify.get('/happiness', (request, reply) => {
  reply.send({ happy: request.isHappy })
})
```

我们见识了如何扩展服务器的功能并且如何处理封装系统, 但是假如你需要加一个方法, 每次在服务器 "[emits](Lifecycle.md)" 事件的时候执行这个方法, 该怎么做?

<a name="hooks"></a>
## 钩子方法
你刚刚构建了工具方法, 现在你需要在每个请求的时候都执行这个方法, 你大概会这样做:
```js
fastify.decorate('util', (request, key, value) => { request[key] = value })

fastify.get('/plugin1', (request, reply) => {
  fastify.util(request, 'timestamp', new Date())
  reply.send(request)
})

fastify.get('/plugin2', (request, reply) => {
  fastify.util(request, 'timestamp', new Date())
  reply.send(request)
})
```
我想大家都同意这个代码是很糟的. 代码重复, 可读性差并且不能扩展.

那么你该怎么消除这个问题呢? 是的, 使用[钩子方法](Hooks.md)!<br>
```js
fastify.decorate('util', (request, key, value) => { request[key] = value })

fastify.addHook('preHandler', (request, reply, done) => {
  fastify.util(request, 'timestamp', new Date())
  done()
})

fastify.get('/plugin1', (request, reply) => {
  reply.send(request)
})

fastify.get('/plugin2', (request, reply) => {
  reply.send(request)
})
```
现在每个请求都会运行工具方法, 很显然你可以注册任意多的需要的钩子方法.<br>
有时, 你希望只在一个路由子集中执行钩子方法, 这个怎么做到?  对了, 封装!

```js
fastify.register((instance, opts, done) => {
  instance.decorate('util', (request, key, value) => { request[key] = value })

  instance.addHook('preHandler', (request, reply, done) => {
    instance.util(request, 'timestamp', new Date())
    done()
  })

  instance.get('/plugin1', (request, reply) => {
    reply.send(request)
  })

  done()
})

fastify.get('/plugin2', (request, reply) => {
  reply.send(request)
})
```
现在你的钩子方法只会在第一个路由中运行!

你可能已经注意到, `request` and `reply` 不是标准的 Nodejs *request* 和 *response* 对象, 而是 Fastify 对象.<br>

<a name="distribution"></a>
## 如何处理封装与分发
完美, 现在你知道了(几乎)所有的扩展 Fastify 的工具. 但可能你遇到了一个大问题: 如何分发你的代码?

我们推荐将所有代码包裹在一个`注册器`中分发, 这样你的插件可以支持异步启动 *(`decorate` 是一个同步 API)*, 例如建立数据库链接.

*等等? 你不是告诉我 `register` 会创建封装的上下文, 那么我创建的不是就外层不可见了?*<br>
是的, 我是说过. 但我没告诉你的是, 你可以通过 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 模块告诉 Fastify 不要进行封装.
```js
const fp = require('fastify-plugin')
const dbClient = require('db-client')

function dbPlugin (fastify, opts, done) {
  dbClient.connect(opts.url, (err, conn) => {
    fastify.decorate('db', conn)
    done()
  })
}

module.exports = fp(dbPlugin)
```
你还可以告诉 `fastify-plugin` 去检查安装的 Fastify 版本, 万一你需要特定的 API.

正如前面所述，Fastify 在 `.listen()`、`.inject()` 以及 `.ready()` 被调用，也即插件被声明 __之后__ 才开始加载插件。这么一来，即使插件通过 [`decorate`](Decorators.md) 向外部的 fastify 实例注入了变量，在调用 `.listen()`、`.inject()` 和 `.ready()` 之前，这些变量是获取不到的。

当你需要在 `register` 方法的 `options` 参数里使用另一个插件注入的变量时，你可以向 `options` 传递一个函数参数，而不是对象：
```js
const fastify = require('fastify')()
const fp = require('fastify-plugin')
const dbClient = require('db-client')

function dbPlugin (fastify, opts, done) {
  dbClient.connect(opts.url, (err, conn) => {
    fastify.decorate('db', conn)
    done()
  })
}

fastify.register(fp(dbPlugin), { url: 'https://example.com' })
fastify.register(require('your-plugin'), parent => {
  return { connection: parent.db, otherOption: 'foo-bar' }
})
```
在上面的例子中，`register` 方法的第二个参数的 `parent` 变量是注册了插件的**外部 fastify 实例**的一份拷贝。这就意味着我们可以获取到之前声明的插件所注入的变量了。

<a name="esm-support"></a>
## ESM 的支持

自 [Node.js `v13.3.0`](https://nodejs.org/api/esm.html) 开始， ESM 也被支持了！写插件时，你只需要将其作为 ESM 模块导出即可！

```js
// plugin.mjs
async function plugin (fastify, opts) {
  fastify.get('/', async (req, reply) => {
    return { hello: 'world' }
  })
}

export default plugin
```

<a name="handle-errors"></a>
## 错误处理
你的插件也可能在启动的时候失败. 或许你预料到这个并且在这种情况下有特定的处理逻辑. 你该怎么实现呢?
`after` API 就是你需要的. `after` 注册一个回调, 在注册之后就会调用这个回调, 它可以有三个参数.<br>
回调会基于不同的参数而变化:

1. 如果没有参数并且有个错误, 这个错误会传递到下一个错误处理.
1. 如果有一个参数, 这个参数就是错误对象.
1. 如果有两个参数, 第一个是错误对象, 第二个是完成回调.
1. 如果有三个参数, 第一个是错误对象, 第二个是顶级上下文(除非你同时指定了服务器和复写, 在这个情况下将会是那个复写的返回), 第三个是完成回调.

让我们看看如何使用它:
```js
fastify
  .register(require('./database-connector'))
  .after(err => {
    if (err) throw err
  })
```

<a name="custom-errors"></a>
## 自定义错误
假如你的插件需要暴露自定义的错误，[`fastify-error`](https://github.com/fastify/fastify-error) 能帮助你轻松地在代码或插件中生成一致的错误对象。

```js
const createError = require('fastify-error')
const CustomError = createError('ERROR_CODE', 'message')
console.log(new CustomError())
```

<a name="emit-warnings"></a>
## 发布提醒
假如你要提示用户某个 API 不被推荐，或某个特殊场景需要注意，你可以使用 [`fastify-warning`](https://github.com/fastify/fastify-warning)。

```js
const warning = require('fastify-warning')()
warning.create('FastifyDeprecation', 'FST_ERROR_CODE', 'message')
warning.emit('FST_ERROR_CODE')
```

<a name="start"></a>
## 开始!
太棒了, 现在你已经知道了所有创建插件需要的关于 Fastify 和它的插件系统的知识, 如果你写了插件请告诉我们! 我们会将它加入到 [*生态*](https://github.com/fastify/fastify#ecosystem) 章节中!

如果你想要看看真正的插件例子, 查看:
- [`point-of-view`](https://github.com/fastify/point-of-view)
给 Fastify 提供模版 (*ejs, pug, handlebars, marko*) 支持.
- [`fastify-mongodb`](https://github.com/fastify/fastify-mongodb)
Fastify MongoDB 连接插件, 可以在全局使用同一 MongoDb 连接池.
- [`fastify-multipart`](https://github.com/fastify/fastify-multipart)
Multipart 支持 
- [`fastify-helmet`](https://github.com/fastify/fastify-helmet) 重要的安全头部支持


*如果感觉还差什么? 告诉我们! :)*
