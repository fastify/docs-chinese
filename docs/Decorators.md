<h1 align="center">Fastify</h1>

## 装饰器

装饰器 API 允许你自定义服务器实例或请求周期中的请求/回复等对象。任意类型的属性都能通过装饰器添加到这些对象上，包括函数、普通对象 (plain object) 以及原生类型。

装饰器 API 是 *同步* 的。如果异步地添加装饰器，可能会导致在装饰器完成初始化之前， Fastify 实例就已经引导完毕了。因此，必须将 `register` 方法与 `fastify-plugin` 结合使用。详见 [Plugins](Plugins.md)。

通过装饰器 API 来自定义对象，底层的 Javascript 引擎便能对其进行优化。这是因为引擎能在所有的对象实例被初始化与使用前，定义好它们的形状 (shape)。下文的例子则是不推荐的做法，因为它在对象的生命周期中修改了它们的形状：

```js
// 不推荐的写法！请继续阅读。

// 在调用请求处理函数之前
// 将 user 属性添加到请求上。
fastify.addHook('preHandler', function (req, reply, done) {
  req.user = 'Bob Dylan'
  done()
})

// 在处理函数中使用 user 属性。
fastify.get('/', function (req, reply) {
  reply.send(`Hello, ${req.user}`)
})
```

由于上述例子在请求对象初始化完成后，还改动了它的形状，因此 JavaScript 引擎必须对该对象去优化。使用装饰器 API 能避开去优化问题：

```js
// 使用装饰器为请求对象添加 'user' 属性。
fastify.decorateRequest('user', '')

// 更新属性。
fastify.addHook('preHandler', (req, reply, done) => {
  req.user = 'Bob Dylan'
  done()
})
// 最后访问它。
fastify.get('/', (req, reply) => {
  reply.send(`Hello, ${req.user}!`)
})
```

装饰器的初始值应该尽可能地与其未来将被设置的值接近。例如，字符串类型的装饰器的初始值为 `''`，对象或函数类型的初始值为 `null`。

请注意，上述例子仅可用于基本类型的值，因为引用类型会在所有请求中共享。详见 [decorateRequest](#decorate-request)。

更多此话题的内容，请见 [JavaScript engine fundamentals: Shapes and Inline Caches](https://mathiasbynens.be/notes/shapes-ics)。

### 使用方法
<a name="usage"></a>

#### `decorate(name, value, [dependencies])`
<a name="decorate"></a>

该方法用于自定义 Fastify [server](Server.md) 实例。

例如，为其添加一个新方法：

```js
fastify.decorate('utility', function () {
  // 新功能的代码
})
```

正如上文所述，还可以传递非函数的值：

```js
fastify.decorate('conf', {
  db: 'some.db',
  port: 3000
})
```

通过装饰属性的名称便可访问值：

```js
fastify.utility()

console.log(fastify.conf.db)
```

[路由](Routes.md)函数的 `this` 指向 [Fastify server](Server.md)：

```js
fastify.decorate('db', new DbConnection())

fastify.get('/', async function (request, reply) {
  reply({hello: await this.db.query('world')})
})
```

可选的 `dependencies` 参数用于指定当前装饰器所依赖的其他装饰器列表。这个列表包含了其他装饰器的名称字符串。在下面的例子里，装饰器 "utility" 依赖于 "greet" 和 "log"：

```js
fastify.decorate('utility', fn, ['greet', 'log'])
```

一旦有依赖项不满足，`decorate` 方法便会抛出异常。依赖项检查是在服务器实例启动前进行的，因此，在运行时不会发生异常。

#### `decorateReply(name, value, [dependencies])`
<a name="decorate-reply"></a>

顾名思义，`decorateReply` 向 `Reply` 核心对象添加新的方法或属性：

```js
fastify.decorateReply('utility', function () {
  // 新功能的代码
})
```

注：使用箭头函数会破坏 `this` 和 Fastify `Reply` 实例的绑定。

注：使用 `decorateReply` 装饰引用类型，会触发警示：

```js
// 反面教材
fastify.decorateReply('foo', { bar: 'fizz'})
```
在这个例子里，该对象的引用存在于所有请求中，导致**任何更改都会影响到所有请求，并可能触发安全漏洞或内存泄露**。要合适地封装请求对象，请在 [`'onRequest'` 钩子](Hooks.md#onrequest)里设置新的值。示例如下：

```js
const fp = require('fastify-plugin')

async function myPlugin (app) {
  app.decorateRequest('foo', null)
  app.addHook('onRequest', async (req, reply) => {
    req.foo = { bar: 42 }
  }) 
}

module.exports = fp(myPlugin)
```

关于 `dependencies` 参数，请见 [`decorate`](#decorate)。

#### `decorateRequest(name, value, [dependencies])`
<a name="decorate-request"></a>

同理，`decorateRequest` 向 `Request` 核心对象添加新的方法或属性：

```js
fastify.decorateRequest('utility', function () {
  // 新功能的代码
})
```

注：使用箭头函数会破坏 `this` 和 Fastify `Request` 实例的绑定。

注：使用 `decorateRequest` 装饰引用类型，会触发警示：

```js
// 反面教材
fastify.decorateRequest('foo', { bar: 'fizz'})
```
在这个例子里，该对象的引用存在于所有请求中，导致**任何更改都会影响到所有请求，并可能触发安全漏洞或内存泄露**。要合适地封装请求对象，请在 [`'onRequest'` 钩子](Hooks.md#onrequest)里设置新的值。示例如下：

```js
const fp = require('fastify-plugin')

async function myPlugin (app) {
  app.decorateRequest('foo', null)
  app.addHook('onRequest', async (req, reply) => {
    req.foo = { bar: 42 }
  }) 
}

module.exports = fp(myPlugin)
```

关于 `dependencies` 参数，请见 [`decorate`](#decorate)。

#### `hasDecorator(name)`
<a name="has-decorator"></a>

用于检查服务器实例上是否存在某个装饰器：

```js
fastify.hasDecorator('utility')
```

#### hasRequestDecorator
<a name="has-request-decorator"></a>

用于检查 Request 实例上是否存在某个装饰器：

```js
fastify.hasRequestDecorator('utility')
```

#### hasReplyDecorator
<a name="has-reply-decorator"></a>

用于检查 Reply 实例上是否存在某个装饰器：

```js
fastify.hasReplyDecorator('utility')
```

### 装饰器与封装
<a name="decorators-encapsulation"></a>

在 **封装** 的同一个上下文中，如果通过 `decorate`、`decorateRequest` 以及 `decorateReply` 多次定义了一个同名的的装饰器，将会抛出一个异常。

下面的示例会抛出异常：
 ```js
const server = require('fastify')()
server.decorateReply('view', function (template, args) {
  // 页面渲染引擎的代码。
})
server.get('/', (req, reply) => {
  reply.view('/index.html', { hello: 'world' })
})
// 当在其他地方定义
// view 装饰器时，抛出异常。
server.decorateReply('view', function (template, args) {
  // 另一个渲染引擎。
})
server.listen(3000)
```

但下面这个例子不会抛异常：

```js
const server = require('fastify')()
  server.decorateReply('view', function (template, args) {
  // 页面渲染引擎的代码。
})
server.register(async function (server, opts) {
  // 我们在当前封装的插件内添加了一个 view 装饰器。
  // 这么做不会抛出异常。
  // 因为插件外部和内部的 view 装饰器是不一样的。
  server.decorateReply('view', function (template, args) {
    // another rendering engine
  })
  server.get('/', (req, reply) => {
    reply.view('/index.page', { hello: 'world' })
  })
}, { prefix: '/bar' })
server.listen(3000)
```

### Getter 和 Setter
<a name="getters-setters"></a>

装饰器接受特别的 "getter/setter" 对象。这些对象拥有着名为 `getter` 与 `setter` 的函数 (尽管 `setter` 是可选的)。这么做便可以通过装饰器来定义属性。例如：

```js
fastify.decorate('foo', {
  getter () {
    return 'a getter'
  }
})
```

上例会在 Fastify 实例中定义一个 `foo` 属性：

```js
console.log(fastify.foo) // 'a getter'
```
