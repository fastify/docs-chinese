<h1 align="center">Fastify</h1>

## 钩子方法

钩子 (hooks) 让你能够监听应用或请求/响应生命周期之上的特定事件。使用 `fastify.addHook` 可以注册钩子。你必须在事件被触发之前注册相应的钩子，否则，事件将得不到处理。

通过钩子方法，你可以与 Fastify 的生命周期直接进行交互。有用于请求/响应的钩子，也有应用级钩子：

- [请求/响应钩子](#requestreply-hooks)
  - [onRequest](#onrequest)
  - [preParsing](#preparsing)
  - [preValidation](#prevalidation)
  - [preHandler](#prehandler)
  - [preSerialization](#preserialization)
  - [onError](#onerror)
  - [onSend](#onsend)
  - [onResponse](#onresponse)
  - [onTimeout](#ontimeout)
  - [在钩子中管理错误](#manage-errors-from-a-hook)
  - [在钩子中响应请求](#respond-to-a-request-from-a-hook)
- [应用钩子](#application-hooks)
  - [onReady](#onready)
  - [onClose](#onclose)
  - [onRoute](#onroute)
  - [onRegister](#onregister)
- [作用域](#scope)
- [路由层钩子](#route-level-hooks)

**注意**：使用 `async`/`await` 或返回一个 `Promise` 时，`done` 回调不可用。在这种情况下，仍然使用 `done` 可能会导致难以预料的行为，例如，处理函数的重复调用。

## 请求/响应钩子

[Request](Request.md) 与 [Reply](Reply.md) 是 Fastify 核心的对象。<br/>
`done` 是调用[生命周期](Lifecycle.md)下一阶段的函数。

[生命周期](Lifecycle.md)一文清晰地展示了各个钩子执行的位置。<br>
钩子可被封装，因此可以运用在特定的路由上。更多信息请看[作用域](#scope)一节。

在请求/响应中，有八个可用的钩子 *(按执行顺序排序)*：

### onRequest
```js
fastify.addHook('onRequest', (request, reply, done) => {
  // 其他代码
  done()
})
```
或使用 `async/await`：
```js
fastify.addHook('onRequest', async (request, reply) => {
  // 其他代码
  await asyncMethod()
})
```

**注意**：在 [onRequest](#onrequest) 钩子中，`request.body` 的值总是 `null`，这是因为 body 的解析发生在 [preValidation](#prevalidation) 钩子之前。

### preParsing

`preParsing` 钩子让你能在解析请求之前转换它们。它的参数除了和其他钩子一样的请求与响应对象外，还多了一个当前请求 payload 的 stream。

需要通过 `return` 或回调函数返回值的话，必须返回一个 stream。

例如，你可以解压请求的 body：

```js
fastify.addHook('preParsing', (request, reply, payload, done) => {
  // 其他代码
  done(null, newPayload)
})
```
或使用 `async/await`：
```js
fastify.addHook('preParsing', async (request, reply, payload) => {
  // 其他代码
  await asyncMethod()
  return newPayload
})
```

**注意**：在 [preParsing](#preparsing) 钩子中，`request.body` 的值总是 `null`，这是因为 body 的解析发生在 [preValidation](#prevalidation) 钩子之前。

**注意**：你应当给返回的 stream 添加 `receivedEncodedLength` 属性。这是为了通过比对请求头的 `Content-Length`，来精确匹配请求的 payload。理想情况下，每收到一块数据都应该更新该属性。

**注意**：早先的写法 `function(request, reply, done)` 与 `function(request, reply)` 仍被支持，但不推荐使用。

### preValidation

使用 `preValidation` 钩子时，你可以在校验前修改 payload。示例如下：

```js
fastify.addHook('preValidation', (request, reply, done) => {
  req.body = { ...req.body, importantKey: 'randomString' }
  done()
})
```
或使用 `async/await`：
```js
fastify.addHook('preValidation', async (request, reply) => {
  const importantKey = await generateRandomString()
  req.body = { ...req.body, importantKey }
})
```

### preHandler
```js
fastify.addHook('preHandler', (request, reply, done) => {
  // 其他代码
  done()
})
```
或使用 `async/await`：
```js
fastify.addHook('preHandler', async (request, reply) => {
  // 其他代码
  await asyncMethod()
})
```
### preSerialization

`preSerialization` 钩子让你可以在 payload 被序列化之前改动 (或替换) 它。举个例子：

```js
fastify.addHook('preSerialization', (request, reply, payload, done) => {
  const err = null
  const newPayload = { wrapped: payload }
  done(err, newPayload)
})
```
或使用 `async/await`：
```js
fastify.addHook('preSerialization', async (request, reply, payload) => {
  return { wrapped: payload }
})
```

注：payload 为 `string`、`Buffer`、`stream` 或 `null` 时，该钩子不会被调用。

### onError
```js
fastify.addHook('onError', (request, reply, error, done) => {
  // 其他代码
  done()
})
```
或使用 `async/await`：
```js
fastify.addHook('onError', async (request, reply, error) => {
  // 当自定义错误日志时有用处
  // 你不应该使用这个钩子去更新错误
})
```
`onError` 钩子可用于自定义错误日志，或当发生错误时添加特定的 header。<br/>
该钩子并不是为了变更错误而设计的，且调用 `reply.send` 会抛出一个异常。<br/>
它只会在 `customErrorHandler` 向用户发送错误之后被执行 (要注意的是，默认的 `customErrorHandler` 总是会发送错误)。
**注意**：与其他钩子不同，`onError` 不支持向 `done` 函数传递错误。

### onSend
使用 `onSend` 钩子可以改变 payload。例如：

```js
fastify.addHook('onSend', (request, reply, payload, done) => {
  const err = null;
  const newPayload = payload.replace('some-text', 'some-new-text')
  done(err, newPayload)
})
```
或使用 `async/await`：
```js
fastify.addHook('onSend', async (request, reply, payload) => {
  const newPayload = payload.replace('some-text', 'some-new-text')
  return newPayload
})
```

你也可以通过将 payload 置为 `null`，发送一个空消息主体的响应：

```js
fastify.addHook('onSend', (request, reply, payload, done) => {
  reply.code(304)
  const newPayload = null
  done(null, newPayload)
})
```

> 将 payload 设为空字符串 `''` 也可以发送空的消息主体。但要小心的是，这么做会造成 `Content-Length` header 的值为 `0`。而 payload 为 `null` 则不会设置 `Content-Length` header。

注：你只能将 payload 修改为 `string`、`Buffer`、`stream` 或 `null`。


### onResponse
```js

fastify.addHook('onResponse', (request, reply, done) => {
  // 其他代码
  done()
})
```
或使用 `async/await`：
```js
fastify.addHook('onResponse', async (request, reply) => {
  // 其他代码
  await asyncMethod()
})
```

`onResponse` 钩子在响应发出后被执行，因此在该钩子中你无法再向客户端发送数据了。但是你可以在此向外部服务发送数据，比如收集数据。

### onTimeout

```js
fastify.addHook('onTimeout', (request, reply, done) => {
  // 其他代码
  done()
})
```
Or `async/await`:
```js
fastify.addHook('onTimeout', async (request, reply) => {
  // 其他代码
  await asyncMethod()
})
```
`onTimeout` 用于监测请求超时，需要在 Fastify 实例上设置 `connectionTimeout` 属性。当请求超时，socket 挂起 (hang up) 时，该钩子执行。因此，在这个钩子里不能再向客户端发送数据了。

### 在钩子中管理错误
在钩子的执行过程中如果发生了错误，只需将错误传递给 `done()`，Fastify 就会自动关闭请求，并发送一个相应的错误码给用户。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  done(new Error('Some error'))
})
```

如果你想自定义发送给用户的错误码，使用 `reply.code()` 即可：
```js
fastify.addHook('preHandler', (request, reply, done) => {
  reply.code(400)
  done(new Error('Some error'))
})
```
*错误最终会在 [`Reply`](Reply.md#errors) 中得到处理。*

或者在 `async/await` 函数中抛出错误：
```js
fastify.addHook('onResponse', async (request, reply) => {
  throw new Error('Some error')
})
```

### 在钩子中响应请求

需要的话，你可以在路由函数执行前响应一个请求，例如进行身份验证。在钩子中响应暗示着钩子的调用链被 __终止__，剩余的钩子将不会执行。假如钩子使用回调的方式，意即不是 `async` 函数，也没有返回 `Promise`，那么只需要调用 `reply.send()`，并且避免触发回调便可。假如钩子是 `async` 函数，那么 `reply.send()` __必须__ 发生在函数返回或 promise resolve 之前，否则请求将会继续下去。当 `reply.send()` 在 promise 调用链之外被调用时，需要  `return reply`，不然请求将被执行两次。

__不应当混用回调与 `async`/`Promise`__，否则钩子的调用链会被执行两次。

如果你在 `onRequest` 或 `preHandler` 中发出响应，请使用 `reply.send`。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  reply.send('Early response')
})

// 也可使用 async 函数
fastify.addHook('preHandler', async (request, reply) => {
  await something()
  reply.send({ hello: 'world' })
  return reply // 在这里是可选的，但这是好的实践
})
```

如果你想要使用流 (stream) 来响应请求，你应该避免使用 `async` 函数。必须使用 `async` 函数的话，请参考 [test/hooks-async.js](https://github.com/fastify/fastify/blob/94ea67ef2d8dce8a955d510cd9081aabd036fa85/test/hooks-async.js#L269-L275) 中的示例来编写代码。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  const stream = fs.createReadStream('some-file', 'utf8')
  reply.send(stream)
})
```

如果发出响应但没有 `await` 关键字，请确保总是 `return reply`：

```js
fastify.addHook('preHandler', async (request, reply) => {
  setImmediate(() => { reply.send('hello') })
  // 让处理函数等待 promise 链之外发出的响应
  return reply
})
fastify.addHook('preHandler', async (request, reply) => {
  // fastify-static 插件异步地发送文件，因此需要 return reply
  reply.sendFile('myfile')
  return reply
})
```

## 应用钩子

你也可以在应用的生命周期里使用钩子方法。要格外注意的是，这些钩子并未被完全封装。钩子中的 `this` 得到了封装，但处理函数可以响应封装界线外的事件。

- [onReady](#onready)
- [onClose](#onclose)
- [onRoute](#onroute)
- [onRegister](#onregister)

### onReady
在服务器开始监听请求之前，或调用 `.ready()` 方法时触发。在此你无法更改路由，或添加新的钩子。注册的 `onReady` 钩子函数串行执行，只有全部执行完毕时，服务器才会开始监听请求。钩子接受一个回调函数作为参数：`done`，在钩子函数完成后调用。钩子的 `this` 为 Fastify 实例。

```js
// 回调写法
fastify.addHook('onReady', function (done) {
  // 其他代码
  const err = null;
  done(err)
})

// 或 async/await
fastify.addHook('onReady', async function () {
  // 异步代码
  await loadCacheFromDatabase()
})
```

<a name="on-close"></a>
### onClose
使用 `fastify.close()` 停止服务器时被触发。当[插件](Plugins.md)需要一个 "shutdown" 事件时有用，例如关闭一个数据库连接。<br>
该钩子的第一个参数是 Fastify 实例，第二个为 `done` 回调函数。
```js
fastify.addHook('onClose', (instance, done) => {
  // 其他代码
  done()
})
```

<a name="on-route"></a>
### onRoute
当注册一个新的路由时被触发。它的监听函数拥有一个唯一的参数：`routeOptions` 对象。该函数是同步的，其本身并不接受回调作为参数。
```js
fastify.addHook('onRoute', (routeOptions) => {
  // 其他代码
  routeOptions.method
  routeOptions.schema
  routeOptions.url // 路由的完整 URL，包括前缀
  routeOptions.path // `url` 的别名
  routeOptions.routePath // 无前缀的 URL
  routeOptions.bodyLimit
  routeOptions.logLevel
  routeOptions.logSerializers
  routeOptions.prefix
})
```

如果在编写插件时，需要自定义程序的路由，比如修改选项或添加新的路由层钩子，你可以在这里添加。

```js
fastify.addHook('onRoute', (routeOptions) => {
  function onPreSerialization(request, reply, payload, done) {
    // 其他代码
    done(null, payload)
  }
  
  // preSerialization 可以是数组或 undefined
  routeOptions.preSerialization = [...(routeOptions.preSerialization || []), onPreSerialization]
})
```

<a name="on-register"></a>
### onRegister
当注册一个新的插件，或创建了新的封装好的上下文后被触发。该钩子在注册的代码**之前**被执行。<br/>
当你的插件需要知晓上下文何时创建完毕，并操作它们时，可以使用这一钩子。<br/>
**注意**：被 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 所封装的插件不会触发该钩子。
```js
fastify.decorate('data', [])

fastify.register(async (instance, opts) => {
  instance.data.push('hello')
  console.log(instance.data) // ['hello']

  instance.register(async (instance, opts) => {
    instance.data.push('world')
    console.log(instance.data) // ['hello', 'world']
  }), { prefix: '/hola' })
}), { prefix: '/ciao' })

fastify.register(async (instance, opts) => {
  console.log(instance.data) // []
}), { prefix: '/hello' })

fastify.addHook('onRegister', (instance, opts) => {
  // 从旧数组浅拷贝，
  // 生成一个新数组，
  // 使用户获得一个
  // 封装好的 `data` 的实例
  instance.data = instance.data.slice()

  // 新注册实例的选项
  console.log(opts.prefix)
})
```

<a name="scope"></a>
## 作用域
除了[应用钩子](#application-hooks)，所有的钩子都是封装好的。这意味着你可以通过 `register` 来决定在何处运行它们，正如[插件指南](Plugins-Guide.md)所述。如果你传递一个函数，那么该函数会获得 Fastify 的上下文，如此你便能使用 Fastify 的 API 了。

```js
fastify.addHook('onRequest', function (request, reply, done) {
  const self = this // Fastify 上下文
  done()
})
```

要注意的是，每个钩子内的 Fastify 上下文都和注册路由时的插件一样，举例如下：

```js
fastify.addHook('onRequest', async function (req, reply) {
  if (req.raw.url === '/nested') {
    assert.strictEqual(this.foo, 'bar')
  } else {
    assert.strictEqual(this.foo, undefined)
  }
})

fastify.get('/', async function (req, reply) {
  assert.strictEqual(this.foo, undefined)
  return { hello: 'world' }
})

fastify.register(async function plugin (fastify, opts) {
  fastify.decorate('foo', 'bar')

  fastify.get('/nested', async function (req, reply) {
    assert.strictEqual(this.foo, 'bar')
    return { hello: 'world' }
  })
})
```

提醒：使用[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)的话，`this` 将不会是 Fastify，而是当前的作用域。

<a name="route-hooks"></a>

## 路由层钩子
你可以为**单个**路由声明一个或多个自定义的生命周期钩子 ([onRequest](#onrequest)、[onResponse](#onresponse)、[preParsing](#preparsing)、[preValidation](#prevalidation)、[preHandler](#prehandler)、[preSerialization](#preserialization)、[onSend](#onsend)、[onTimeout](#ontimeout) 与 [onError](#onerror))。
如果你这么做，这些钩子总是会作为同一类钩子中的最后一个被执行。<br/>
当你需要进行认证时，这会很有用，而 [preParsing](#preparsing) 与 [preValidation](#prevalidation) 钩子正是为此而生。
你也可以通过数组定义多个路由层钩子。

```js
fastify.addHook('onRequest', (request, reply, done) => {
  // 你的代码
  done()
})

fastify.addHook('onResponse', (request, reply, done) => {
  // 你的代码
  done()
})

fastify.addHook('preParsing', (request, reply, done) => {
  // 你的代码
  done()
})

fastify.addHook('preValidation', (request, reply, done) => {
  // 你的代码
  done()
})

fastify.addHook('preHandler', (request, reply, done) => {
  // 你的代码
  done()
})

fastify.addHook('preSerialization', (request, reply, payload, done) => {
  // 你的代码
  done(null, payload)
})

fastify.addHook('onSend', (request, reply, payload, done) => {
  // 你的代码
  done(null, payload)
})

fastify.addHook('onTimeout', (request, reply, done) => {
  // 你的代码
  done()
})

fastify.addHook('onError', (request, reply, error, done) => {
  // 你的代码
  done()
})

fastify.route({
  method: 'GET',
  url: '/',
  schema: { ... },
  onRequest: function (request, reply, done) {
    // 该钩子总是在共享的 `onRequest` 钩子后被执行
    done()
  },
  onResponse: function (request, reply, done) {
    // 该钩子总是在共享的 `onResponse` 钩子后被执行
    done()
  },
  preParsing: function (request, reply, done) {
    // 该钩子总是在共享的 `preParsing` 钩子后被执行
    done()
  },
  preValidation: function (request, reply, done) {
    // 该钩子总是在共享的 `preValidation` 钩子后被执行
    done()
  },
  preHandler: function (request, reply, done) {
    // 该钩子总是在共享的 `preHandler` 钩子后被执行
    done()
  },
  // // 使用数组的例子。所有钩子都支持这一语法。
  //
  // preHandler: [function (request, reply, done) {
  //   // 该钩子总是在共享的 `preHandler` 钩子后被执行
  //   done()
  // }],
  preSerialization: (request, reply, payload, done) => {
    // 该钩子总是在共享的 `preSerialization` 钩子后被执行
    done(null, payload)
  },
  onSend: (request, reply, payload, done) => {
    // 该钩子总是在共享的 `onSend` 钩子后被执行
    done(null, payload)
  },
  onTimeout: (request, reply, done) => {
    // 该钩子总是在共享的 `onTimeout` 钩子后被执行
    done()
  },
  onError: (request, reply, error, done) => {
    // 该钩子总是在共享的 `onError` 钩子后被执行
    done()
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})
```

**注**：两个选项都接受一个函数数组作为参数。

## 诊断通道钩子

> **注：** `诊断通道` (diagnostics_channel) 是当前 Node.js 的试验特性，
> 因此，其 API 即便在补丁版本中也可能会发生变动。
> 对于 Fastify 支持的 Node.js 版本，不兼容 `诊断通道` 的，
> 将会使用 [polyfill](https://www.npmjs.com/package/diagnostics_channel)。
> 而 polyfill 都不支持的版本将无法使用该特性。

当前版本在初始化时，会有一个[`诊断通道`](https://nodejs.org/api/diagnostics_channel.html)发布 `'fastify.initialization'` 事件。此时，Fastify 的实例将会作为回调函数参数的一个属性，该实例可以添加钩子、插件、路由及其他任意内容。

举例来说，某个监控工具可以如下使用（当然这是一个简化的例子）。在典型的“探测工具优先加载 (require instrumentation
tools first)”风格下，这段代码会在被监控的应用初始化时加载。

```js
const tracer = /* 某个监控工具 */
const dc = require('diagnostics_channel')
const channel = dc.channel('fastify.initialization')
const spans = new WeakMap()

channel.subscribe(function ({ fastify }) {
  fastify.addHook('onRequest', (request, reply, done) => {
    const span = tracer.startSpan('fastify.request')
    spans.set(request, span)
    done()
  })

  fastify.addHook('onResponse', (request, reply, done) => {
    const span = spans.get(request)
    span.finish()
    done()
  })
})
```