<h1 align="center">Fastify</h1>

## 钩子方法

钩子 (hooks) 让你能够监听应用或请求/响应生命周期之上的特定事件。使用 `fastify.addHook` 可以注册钩子。你必须在事件被触发之前注册相应的钩子，否则，事件将得不到处理。

## 请求/响应钩子

通过钩子方法，你可以在 Fastify 的生命周期内直接进行交互。有五个可用的钩子 *(按执行顺序排序)*：
- `'onRequest'`
- `'preParsing'`
- `'preValidation'`
- `'preHandler'`
- `'preSerialization'`
- `'onError'`
- `'onSend'`
- `'onResponse'`

示例：
```js
fastify.addHook('onRequest', (request, reply, next) => {
  // 其他代码
  next()
})

fastify.addHook('preParsing', (request, reply, next) => {
  // 其他代码
  next()
})

fastify.addHook('preValidation', (request, reply, next) => {
  // 其他代码
  next()
})

fastify.addHook('preHandler', (request, reply, next) => {
  // 其他代码
  next()
})

fastify.addHook('preSerialization', (request, reply, payload, next) => {
  // 其他代码
  next()
})

fastify.addHook('onError', (request, reply, error, next) => {
  // 其他代码
  next()
})

fastify.addHook('onSend', (request, reply, payload, next) => {
  // 其他代码
  next()
})

fastify.addHook('onResponse', (request, reply, next) => {
  // 其他代码
  next()
})
```
或使用 `async/await`
```js
fastify.addHook('onRequest', async (request, reply) => {
  // 其他代码
  await asyncMethod()
  // 发生错误
  if (err) {
    throw new Error('some errors occurred.')
  }
  return
})

fastify.addHook('preParsing', async (request, reply) => {
  // 其他代码
  await asyncMethod()
  // 发生错误
  if (err) {
    throw new Error('some errors occurred.')
  }
  return
})

fastify.addHook('preValidation', async (request, reply) => {
  // 其他代码
  await asyncMethod()
  // 发生错误
  if (err) {
    throw new Error('some errors occurred.')
  }
  return
})

fastify.addHook('preHandler', async (request, reply) => {
  // 其他代码
  await asyncMethod()
  // 发生错误
  if (err) {
    throw new Error('some errors occurred.')
  }
  return
})

fastify.addHook('preSerialization', async (request, reply, payload) => {
  // 其他代码
  await asyncMethod()
  // 发生错误
  if (err) {
    throw new Error('some errors occurred.')
  }
  return payload
})

fastify.addHook('onError', async (request, reply, error) => {
  // 当自定义错误日志时有用处
  // 你不应该使用这个钩子去更新错误
})

fastify.addHook('onSend', async (request, reply, payload) => {
  // 其他代码
  await asyncMethod()
  // 发生错误
  if (err) {
    throw new Error('some errors occurred.')
  }
  return
})

fastify.addHook('onResponse', async (request, reply) => {
  // 其他代码
  await asyncMethod()
  // 发生错误
  if (err) {
    throw new Error('some errors occurred.')
  }
  return
})
```

**注意**：使用 `async`/`await` 或返回一个 `Promise` 时，`next` 回调不可用。在这种情况下，仍然使用 `next` 可能会导致难以预料的行为，例如，处理函数的重复调用。

**注意**：在 `onRequest` 与 `preValidation` 钩子中，`request.body` 的值总是 `null`，这是因为 body 的解析发生在 `preHandler` 钩子之前。

[Request](https://github.com/fastify/docs-chinese/blob/master/docs/Request.md) 与 [Reply](https://github.com/fastify/docs-chinese/blob/master/docs/Reply.md) 是 Fastify 核心的对象。<br/>
`next` 是调用[生命周期](https://github.com/fastify/docs-chinese/blob/master/docs/Lifecycle.md)下一阶段的函数。

[生命周期](https://github.com/fastify/docs-chinese/blob/master/docs/Lifecycle.md)一文清晰地展示了各个钩子执行的位置。<br>
钩子可被封装，因此可以运用在特定的路由上。更多信息请看[作用域](#scope)一节。

在钩子的执行过程中如果发生了错误，只需将错误传递给 `next()`，Fastify 就会自动关闭请求，并发送一个相应的错误码给用户。

```js
fastify.addHook('onRequest', (request, reply, next) => {
  next(new Error('some error'))
})
```

如果你想自定义发送给用户的错误码，使用 `reply.code()` 即可：
```js
fastify.addHook('preHandler', (request, reply, next) => {
  reply.code(400)
  next(new Error('some error'))
})
```

*错误最终会在 [`Reply`](https://github.com/fastify/docs-chinese/blob/master/docs/Reply.md#errors) 中得到处理*

#### `onError` 钩子

`onError` 钩子可用于自定义错误日志，或当发生错误时添加特定的 header。<br/>
该钩子并不是为了变更错误而设计的，且调用 `reply.send` 会抛出一个异常。<br/>
它只会在 `customErrorHandler` 向用户发送错误之后被执行 (要注意的是，默认的 `customErrorHandler` 总是会发送错误)。
**注意**：与其他钩子不同，`onError` 不支持向 `next` 函数传递错误。

```js
fastify.addHook('onError', (request, reply, error, next) => {
  // apm 代表应用性能监控 (Application Performance Monitoring)
  apm.sendError(error)
  next()
})

// 或使用 async
fastify.addHook('onError', async (request, reply, error) => {
  // apm 代表应用性能监控 (Application Performance Monitoring)
  apm.sendError(error)
})
```

#### `preSerialization` 钩子

`preSerialization` 钩子让你可以在 payload 被序列化之前改动它。举个例子：

 ```js
fastify.addHook('preSerialization', (request, reply, payload, next) => {
  var err = null;
  var newPayload = { wrapped: payload }
  next(err, newPayload)
})
// 或使用 async
fastify.addHook('preSerialization', async (request, reply, payload) => {
  return { wrapped: payload }
})
```

payload 为 `string`、`Buffer`、`stream` 或 `null` 时，该钩子不会被调用。

#### `onSend` 钩子

使用 `onSend` 钩子可以改变 payload。例如：

```js
fastify.addHook('onSend', (request, reply, payload, next) => {
  var err = null;
  var newPayload = payload.replace('some-text', 'some-new-text')
  next(err, newPayload)
})

// 或者使用 async
fastify.addHook('onSend', async (request, reply, payload) => {
  var newPayload = payload.replace('some-text', 'some-new-text')
  return newPayload
})
```

你也可以通过将 payload 置为 `null`，发送一个空消息主体的响应：

```js
fastify.addHook('onSend', (request, reply, payload, next) => {
  reply.code(304)
  const newPayload = null
  next(null, newPayload)
})
```

> 将 payload 设为空字符串 `''` 也可以发送空的消息主体。但要小心的是，这么做会造成 `Content-Length` header 的值为 `0`。而 payload 为 `null` 则不会设置 `Content-Length` header。

注：你只能将 payload 修改为 `string`、`Buffer`、`stream` 或 `null`。

#### `onResponse` 钩子
当响应发出后，`onResponse` 钩子即被执行，因此在该钩子中你无法再向客户端发送数据了。但是你可以在此向外部服务发送数据，或是执行一些统计工作。

### 在钩子中响应请求
需要的话，你可以在路由控制器执行前响应一个请求。一个例子便是身份验证的钩子。如果你在 `onRequest` 或 `preHandler` 中发出响应，请使用 `reply.send`。如果是在中间件中，使用 `res.end`。

```js
fastify.addHook('onRequest', (request, reply, next) => {
  reply.send('early response')
})

// 也可使用 async 函数
fastify.addHook('preHandler', async (request, reply) => {
  reply.send({ hello: 'world' })
})
```

如果你想要使用流 (stream) 来响应请求，你应该避免使用 `async` 函数。必须使用 `async` 函数的话，请参考 [test/hooks-async.js](https://github.com/fastify/fastify/blob/94ea67ef2d8dce8a955d510cd9081aabd036fa85/test/hooks-async.js#L269-L275) 中的示例来编写代码。

```js
fastify.addHook('onRequest', (request, reply, next) => {
  const stream = fs.createReadStream('some-file', 'utf8')
  reply.send(stream)
})
```

## 应用钩子

你也可以在应用的生命周期里使用钩子方法。要格外注意的是，这些钩子并未被完全封装。钩子中的 `this` 得到了封装，但处理函数可以响应封装界线外的事件。

- `'onClose'`
- `'onRoute'`
- `'onRegister''`

<a name="on-close"></a>
**'onClose'**<br>
使用 `fastify.close()` 停止服务器时被触发。当[插件](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md)需要一个 "shutdown" 事件时有用，例如一个用于连接数据库的插件。<br>
该钩子的第一个参数是 Fastify 实例，第二个为 `done` 回调函数。
```js
fastify.addHook('onClose', (instance, done) => {
  // 其他代码
  done()
})
```
<a name="on-route"></a>
**'onRoute'**<br>

当注册一个新的路由时被触发。它的监听函数拥有一个唯一的参数：`routeOptions` 对象。该函数是同步的，其本身并不接受回调作为参数。
```js
fastify.addHook('onRoute', (routeOptions) => {
  // 其他代码
  routeOptions.method
  routeOptions.schema
  routeOptions.url
  routeOptions.bodyLimit
  routeOptions.logLevel
  routeOptions.prefix
})
```
<a name="on-register"></a>
**'onRegister'**<br>
当注册一个新的插件，或创建了新的封装好的上下文后被触发。该钩子在插件的代码**之前**被执行。<br/>
当你的插件需要知晓上下文何时创建完毕，并操作它们时，可以使用这一钩子。<br/>
**注意：**被 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 所封装的插件不会触发该钩子。
```js
fastify.decorate('data', [])

fastify.register(async (instance, opts) => {
  instance.data.push('hello')
  console.log(instance.data) // ['hello']

  instance.register(async (instance, opts) => {
    instance.data.push('world')
    console.log(instance.data) // ['hello', 'world']
  })
})

fastify.register(async (instance, opts) => {
  console.log(instance.data) // []
})

fastify.addHook('onRegister', (instance) => {
  // 从旧数组浅拷贝，
  // 生成一个新数组，
  // 使用户获得一个
  // 封装好的 `data` 的实例
  instance.data = instance.data.slice()
})
```

<a name="scope"></a>
### 作用域
除了[应用钩子](#application-hooks)，所有的钩子都是封装好的。这意味着你可以通过 `register` 来决定在何处运行它们，正如[插件指南](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins-Guide.md)所述。如果你传递一个函数，那么该函数会获得 Fastify 的上下文，如此你便能使用 Fastify 的 API 了。

```js
fastify.addHook('onRequest', function (request, reply, next) {
  const self = this // Fastify 上下文
  next()
})
```
注：使用箭头函数会破坏 Fastify 实例对 this 的绑定。

<a name="route-hooks"></a>
## 路由层钩子
你可以为**单个**路由声明一个或多个自定义的 `onRequest`、`preParsing`、`preHandler`、`preValidation` 与 `preSerialization` 钩子。
如果你这么做，这些钩子总是会作为同一类钩子中的最后一个被执行。<br/>
当你需要进行认证时，这会很有用，而 `preParsing` 与 `preValidation` 钩子正是为此而生。
你也可以通过数组定义多个路由层钩子。

让我们看下范例：

```js
fastify.addHook('onRequest', (request, reply, done) => {
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
    // 操作 payload
    done(null, payload)
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})
```

**注**：两个选项都接受一个函数数组作为参数。