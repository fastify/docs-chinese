<h1 align="center">Fastify</h1>

## 路由

路由方法设置你程序的路由。
你可以使用简写定义与完整定义两种方式来设定路由。

- [完整定义](#full-declaration)
- [路由选项](#options)
- [简写定义](#shorthand-declaration)
- [URL 参数](#url-building)
- [使用 `async`/`await`](#async-await)
- [Promise 取舍](#promise-resolution)
- [路由前缀](#route-prefixing)
- 日志
  - [自定义日志级别](#custom-log-level)
  - [自定义日志序列化器](#custom-log-serializer)
- [配置路由的处理函数](#routes-config)
- [路由版本](#version)

<a name="full-declaration"></a>
### 完整定义

```js
fastify.route(options)
```

<a name="options"></a>
### 路由选项

* `method`：支持的 HTTP 请求方法。目前支持 `'DELETE'`、`'GET'`、`'HEAD'`、`'PATCH'`、`'POST'`、`'PUT'` 以及 `'OPTIONS'`。它还可以是一个 HTTP 方法的数组。
* `url`：路由匹配的 url 路径 (别名：`path`)。
* `schema`：用于验证请求与回复的 schema 对象。
必须符合 [JSON Schema](http://json-schema.org/) 格式。请看[这里](Validation-and-Serialization.md)了解更多信息。

  * `body`：当为 POST 或 PUT 方法时，校验请求主体。
  * `querystring` 或 `query`：校验 querystring。可以是一个完整的 JSON Schema 对象，它包括了值为 `object` 的 `type` 属性以及包含参数的 `properties` 对象，也可以仅仅是 `properties` 对象中的值 (见下文示例)。
  * `params`：校验 url 参数。
  * `response`：过滤并生成用于响应的 schema，能帮助提升 10-20% 的吞吐量。
* `attachValidation`：当 schema 校验出错时，将一个 `validationError` 对象添加到请求中，否则错误将被发送给错误处理函数。
* `onRequest(request, reply, done)`：每当接收到一个请求时触发的[函数](Hooks.md#onrequest)。可以是一个函数数组。
* `preParsing(request, reply, done)`：解析请求前调用的[函数](Hooks.md#preparsing)。可以是一个函数数组。
* `preValidation(request, reply, done)`：在共享的 `preValidation` 钩子之后执行的[函数](Hooks.md#prevalidation)，在路由层进行认证等场景中会有用处。可以是一个函数数组。
* `preHandler(request, reply, done)`：处理请求之前调用的[函数](Hooks.md#prehandler)。可以是一个函数数组。
* `preSerialization(request, reply, payload, done)`：序列化之前调用的[函数](Hooks.md#preserialization)。可以是一个函数数组。
* `onSend(request, reply, payload, done)`：响应即将发送前调用的[函数](Hooks.md#route-hooks)。可以是一个函数数组。
* `onResponse(request, reply, done)`：当响应发送后调用的[函数](Hooks.md#onresponse)。因此，在这个函数内部，不允许再向客户端发送数据。可以是一个函数数组。
* `handler(request, reply)`：处理请求的函数。函数被调用时，[Fastify server](Server.md) 将会与 `this` 进行绑定。注意，使用箭头函数会破坏这一绑定。
* `errorHandler(error, request, reply)`：在请求作用域内使用的自定义错误控制函数。覆盖默认的全局错误函数，以及由 [`setErrorHandler`](Server.md#setErrorHandler) 设置的请求错误函数。你可以通过 `instance.errorHandler` 访问默认的错误函数，在没有插件覆盖的情况下，其指向 Fastify 默认的 `errorHandler`。
* `validatorCompiler({ schema, method, url, httpPart })`：生成校验请求的 schema 的函数。详见[验证与序列化](Validation-and-Serialization.md#schema-validator)。
* `serializerCompiler({ { schema, method, url, httpStatus } })`：生成序列化响应的 schema 的函数。详见[验证与序列化](Validation-and-Serialization.md#schema-serializer)。
* `schemaErrorFormatter(errors, dataVar)`：生成一个函数，用于格式化来自 schema 校验函数的错误。详见[验证与序列化](Validation-and-Serialization.md#schema-validator)。在当前路由上会覆盖全局的 schema 错误格式化函数，以及 `setSchemaErrorFormatter` 设置的值。
* `bodyLimit`：一个以字节为单位的整形数，默认值为 `1048576` (1 MiB)，防止默认的 JSON 解析器解析超过此大小的请求主体。你也可以通过 `fastify(options)`，在首次创建 Fastify 实例时全局设置该值。
* `logLevel`：设置日志级别。详见下文。
* `logSerializers`：设置当前路由的日志序列化器。
* `config`：存放自定义配置的对象。
* `version`：一个符合[语义化版本控制规范 (semver)](http://semver.org/) 的字符串。[示例](Routes.md#version)。
`prefixTrailingSlash`：一个字符串，决定如何处理带前缀的 `/` 路由。
  * `both` (默认值)：同时注册 `/prefix` 与 `/prefix/`。
  * `slash`：只会注册 `/prefix/`。
  * `no-slash`：只会注册 `/prefix`。

  `request` 的相关内容请看[请求](Request.md)一文。

  `reply` 请看[回复](Reply.md)一文。

示例：
```js
fastify.route({
  method: 'GET',
  url: '/',
  schema: {
    querystring: {
      name: { type: 'string' },
      excitement: { type: 'integer' }
    },
    response: {
      200: {
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})
```

<a name="shorthand-declaration"></a>
### 简写定义
上文的路由定义带有 *Hapi* 的风格。要是偏好 *Express/Restify* 的写法，Fastify 也是支持的：<br>
`fastify.get(path, [options], handler)`<br>
`fastify.head(path, [options], handler)`<br>
`fastify.post(path, [options], handler)`<br>
`fastify.put(path, [options], handler)`<br>
`fastify.delete(path, [options], handler)`<br>
`fastify.options(path, [options], handler)`<br>
`fastify.patch(path, [options], handler)`

示例：
```js
const opts = {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  }
}
fastify.get('/', opts, (request, reply) => {
  reply.send({ hello: 'world' })
})
```

`fastify.all(path, [options], handler)` 会给所有支持的 HTTP 方法添加相同的处理函数。

处理函数还可以写到 `options` 对象里：
```js
const opts = {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  },
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
}
fastify.get('/', opts)
```

> 注：假如同时在 `options` 和简写方法的第三个参数里指明了处理函数，将会抛出重复的 `handler` 错误。

<a name="url-building"></a>
### Url 构建
Fastify 同时支持静态与动态的 url。<br>
要注册一个**参数命名**的路径，请在参数名前加上*冒号*。*星号*表示**通配符**。
*注意，静态路由总是在参数路由和通配符之前进行匹配。*

```js
// 参数路由
fastify.get('/example/:userId', (request, reply) => {})
fastify.get('/example/:userId/:secretToken', (request, reply) => {})

// 通配符
fastify.get('/example/*', (request, reply) => {})
```

正则表达式路由亦被支持。但要注意，正则表达式会严重拖累性能！
```js
// 正则表达的参数路由
fastify.get('/example/:file(^\\d+).png', (request, reply) => {})
```

你还可以在同一组斜杠 ("/") 里定义多个参数。就像这样：
```js
fastify.get('/example/near/:lat-:lng/radius/:r', (request, reply) => {})
```
*使用短横线 ("-") 来分隔参数。*

最后，同时使用多参数和正则表达式也是允许的。
```js
fastify.get('/example/at/:hour(^\\d{2})h:minute(^\\d{2})m', (request, reply) => {})
```
在这个例子里，任何未被正则匹配的符号均可作为参数的分隔符。

多参数的路由会影响性能，所以应该尽量使用单参数，对于高频访问的路由来说更是如此。
如果你对路由的底层感兴趣，可以查看[find-my-way](https://github.com/delvedor/find-my-way)。

<a name="async-await"></a>
### Async Await
你是 `async/await` 的使用者吗？我们为你考虑了一切！
```js
fastify.get('/', options, async function (request, reply) {
  var data = await getData()
  var processed = await processData(data)
  return processed
})
```

如你所见，我们不再使用 `reply.send` 向用户发送数据，只需返回消息主体就可以了！

当然，需要的话你还是可以使用 `reply.send` 发送数据。
```js
fastify.get('/', options, async function (request, reply) {
  var data = await getData()
  var processed = await processData(data)
  reply.send(processed)
})
```

假如在路由中，`reply.send()` 脱离了 promise 链，在一个基于回调的 API 中被调用，你可以使用 `await reply`：

```js
fastify.get('/', options, async function (request, reply) {
  setImmediate(() => {
    reply.send({ hello: 'world' })
  })
  await reply
})
```

返回回复也是可行的：

```js
fastify.get('/', options, async function (request, reply) {
  setImmediate(() => {
    reply.send({ hello: 'world' })
  })
  return reply
})
```

**警告:**
* 如果你同时使用 `return value` 与 `reply.send(value)`，那么只会发送第一次，同时还会触发警告日志，因为你试图发送两次响应。
* 不能返回 `undefined`。更多细节请看 [promise 取舍](#promise-resolution)。

<a name="promise-resolution"></a>
### Promise 取舍

假如你的处理函数是一个 `async` 函数，或返回了一个 promise，请注意一种必须支持回调函数和 promise 控制流的特殊情况：如果 promise 被 resolve 为 `undefined`，请求会被挂起，并触发一个*错误*日志。

1. 如果你想使用 `async/await` 或 promise，但通过 `reply.send` 返回值：
    - **别** `return` 任何值。
    - **别**忘了 `reply.send`。
2. 如果你想使用 `async/await` 或 promise：
    - **别**使用 `reply.send`。
    - **别**返回 `undefined`。

通过这一方法，我们便可以最小代价同时支持 `回调函数风格` 以及 `async-await`。尽管这么做十分自由，我们还是强烈建议仅使用其中的一种，因为应用的错误处理方式应当保持一致。

**注意**：每个 async 函数各自返回一个 promise 对象。

<a name="route-prefixing"></a>
### 路由前缀
有时你需要维护同一 api 的多个不同版本。一般的做法是在所有的路由之前加上版本号，例如 `/v1/user`。
Fastify 提供了一个快捷且智能的方法来解决上述问题，无需手动更改全部路由。这就是*路由前缀*。让我们来看下吧：

```js
// server.js
const fastify = require('fastify')()

fastify.register(require('./routes/v1/users'), { prefix: '/v1' })
fastify.register(require('./routes/v2/users'), { prefix: '/v2' })

fastify.listen(3000)
```

```js
// routes/v1/users.js
module.exports = function (fastify, opts, done) {
  fastify.get('/user', handler_v1)
  done()
}
```

```js
// routes/v2/users.js
module.exports = function (fastify, opts, done) {
  fastify.get('/user', handler_v2)
  done()
}
```
在编译时 Fastify 自动处理了前缀，因此两个不同路由使用相同的路径名并不会产生问题。*(这也意味着性能一点儿也不受影响！)*。

现在，你的客户端就可以访问下列路由了：
- `/v1/user`
- `/v2/user`

根据需要，你可以多次设置路由前缀，它也支持嵌套的 `register` 以及路由参数。
请注意，当使用了 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 时，这一选项是无效的。

#### 处理带前缀的 / 路由

根据前缀是否以 `/` 结束，路径为 `/` 的路由的匹配模式有所不同。举例来说，前缀为 `/something/` 的 `/` 路由只会匹配 `something`，而前缀为 `/something` 则会匹配 `/something` 和 `/something/`。

要改变这一行为，请见上文 `prefixTrailingSlash` 选项。

<a name="custom-log-level"></a>
### 自定义日志级别
在 Fastify 中为路由里设置不同的日志级别是十分容易的。<br/>
你只需在插件或路由的选项里设置 `logLevel` 为相应的[值](https://github.com/pinojs/pino/blob/master/docs/api.md#level-string)即可。

要注意的是，如果在插件层面上设置了 `logLevel`，那么 [`setNotFoundHandler`](Server.md#setnotfoundhandler) 和 [`setErrorHandler`](Server.md#seterrorhandler) 也会受到影响。

```js
// server.js
const fastify = require('fastify')({ logger: true })

fastify.register(require('./routes/user'), { logLevel: 'warn' })
fastify.register(require('./routes/events'), { logLevel: 'debug' })

fastify.listen(3000)
```

你也可以直接将其传给路由：
```js
fastify.get('/', { logLevel: 'warn' }, (request, reply) => {
  reply.send({ hello: 'world' })
})
```
*自定义的日志级别仅对路由生效，通过 `fastify.log` 访问的全局日志并不会受到影响。*

<a name="custom-log-serializer"></a>
### 自定义日志序列化器

在某些上下文里，你也许需要记录一个大型对象，但这在其他路由中是个负担。这时，你可以定义一些[`序列化器 (serializer)`](https://github.com/pinojs/pino/blob/master/docs/api.md#bindingsserializers-object)，并将它们设置在正确的上下文之上！

```js
const fastify = require('fastify')({ logger: true })
fastify.register(require('./routes/user'), { 
  logSerializers: {
    user: (value) => `My serializer one - ${value.name}`
  } 
})
fastify.register(require('./routes/events'), {
  logSerializers: {
    user: (value) => `My serializer two - ${value.name} ${value.surname}`
  }
})
fastify.listen(3000)
```

你可以通过上下文来继承序列化器：

```js
const fastify = Fastify({ 
  logger: {
    level: 'info',
    serializers: {
      user (req) {
        return {
          method: req.method,
          url: req.url,
          headers: req.headers,
          hostname: req.hostname,
          remoteAddress: req.ip,
          remotePort: req.socket.remotePort
        }
      }
    }
  } 
})
fastify.register(context1, { 
  logSerializers: {
    user: value => `My serializer father - ${value}`
  } 
})
async function context1 (fastify, opts) {
  fastify.get('/', (req, reply) => {
    req.log.info({ user: 'call father serializer', key: 'another key' })
    // 打印结果： { user: 'My serializer father - call father  serializer', key: 'another key' }
    reply.send({})
  })
}
fastify.listen(3000)
```

<a name="routes-config"></a>
### 配置
注册一个新的处理函数，你可以向其传递一个配置对象，并在其中使用它。

```js
// server.js
const fastify = require('fastify')()

function handler (req, reply) {
  reply.send(reply.context.config.output)
}

fastify.get('/en', { config: { output: 'hello world!' } }, handler)
fastify.get('/it', { config: { output: 'ciao mondo!' } }, handler)

fastify.listen(3000)
```

<a name="version"></a>
### 版本

#### 默认
需要的话，你可以提供一个版本选项，它允许你为同一个路由声明不同的版本。版本号请遵循 [semver](http://semver.org/) 规范。<br/>
Fastify 会自动检测 `Accept-Version` header，并将请求分配给相应的路由 (当前尚不支持 semver 规范中的 advanced ranges 与 pre-releases 语法)。<br/>
*请注意，这一特性会降低路由的性能。*

```js
fastify.route({
  method: 'GET',
  url: '/',
  version: '1.2.0',
  handler: function (request, reply) {
    reply.send({ hello: 'world' })
  }
})

fastify.inject({
  method: 'GET',
  url: '/',
  headers: {
    'Accept-Version': '1.x' // 也可以是 '1.2.0' 或 '1.2.x'
  }
}, (err, res) => {
  // { hello: 'world' }
})
```

> ## ⚠  安全提示
> 记得设置 [`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary) 响应头
> 为用于区分版本的值 (如 `'Accept-Version'`)，
> 来避免缓存污染攻击 (cache poisoning attacks)。你也可以在代理或 CDN 层设置该值。
> 
> ```js
> const append = require('vary').append
> fastify.addHook('onSend', async (req, reply) => {
>   if (req.headers['accept-version']) { // 或其他自定义 header
>     let value = reply.getHeader('Vary') || ''
>     const header = Array.isArray(value) ? value.join(', ') : String(value)
>     if ((value = append(header, 'Accept-Version'))) { // 或其他自定义 header
>       reply.header('Vary', value)
>     }
>   }
> })
> ```

如果你声明了多个拥有相同主版本或次版本号的版本，Fastify 总是会根据 `Accept-Version` header 的值选择最兼容的版本。<br/>
假如请求未带有 `Accept-Version` header，那么将返回一个 404 错误。

#### 自定义
新建实例时，可以通过设置 [`versioning`](Server.md#versioning) 来自定义版本号逻辑。