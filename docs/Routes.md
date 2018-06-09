<h1 align="center">Fastify</h1>

## 路由
Fastify 支持简写定义与完整定义两种方式来设定路由。让我们先从第二种开始吧：
<a name="full-declaration"></a>
### 完整定义
```js
fastify.route(options)
```
* `method`：支持的 HTTP 请求方法。目前支持 `'DELETE'`、`'GET'`、`'HEAD'`、`'PATCH'`、`'POST'`、`'PUT'` 以及 `'OPTIONS'`。它还可以是一个 HTTP 方法的数组。

* `url`：路由匹配的 url 路径 (别名：`path`)。
* `schema`：用于验证请求与回复的 schema 对象。
必须符合 [JSON Schema](http://json-schema.org/) 格式。请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md)了解更多信息。

  * `body`：当为 POST 或 PUT 方法时，校验请求主体。
  * `querystring`: 校验 querystring。可以是一个完整的 JSON Schema 对象，它包括了值为 `object` 的 `type` 属性以及包含参数的 `properties` 对象，也可以仅仅是 `properties` 对象中的值 (见下文示例)。
  * `params`: 校验 url 参数。
  * `response`：过滤并生成用于响应的 schema，能帮助提升 10-20% 的吞吐量。
* `beforeHandler(request, reply, done)`：处理请求之前调用的[钩子函数](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md#before-handler)，当需要在路由层面进行身份验证等操作时能派上用场。它还可以是一个函数数组。
* `handler(request, reply)`：处理请求的函数。
* `schemaCompiler(schema)`：生成校验 schema 的函数。请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md#schema-compiler)。
* `bodyLimit`：一个以字节为单位的整形数，默认值为 `1048576` (1 MiB)，防止默认的 JSON 解析器解析超过此大小的请求主体。你也可以通过 `fastify(options)`，在首次创建 Fastify 实例时全局设置该值。
* `logLevel`：设置日志级别。详见下文。
* `config`：存放自定义配置的对象。

  `request` 的相关内容请看[请求](https://github.com/fastify/docs-chinese/blob/master/docs/Request.md)一文。

  `reply` 请看[回复](https://github.com/fastify/docs-chinese/blob/master/docs/Reply.md)一文。


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

`fastify.all(path, [options], handler)` 会给所有支持的 HTTP 方法添加相同的处理器。

处理器还可以写到 `options` 对象里：
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
  handler (request, reply) {
    reply.send({ hello: 'world' })
  }
}
fastify.get('/', opts)
```

> 注：假如同时在 `options` 和简写方法的第三个参数里指明了处理器，将会抛出重复的 `handler` 错误。

<a name="url-building"></a>
### Url 构建
Fastify 同时支持静态与动态的 url。<br>
要注册一个**参数命名**的路径，请在参数名前加上*冒号*。*星号*表示**通配符**。
*注意，静态路由总是在参数路由和通配符之前进行匹配。*

```js
// 参数路由
fastify.get('/example/:userId', (request, reply) => {}))
fastify.get('/example/:userId/:secretToken', (request, reply) => {}))

// 通配符
fastify.get('/example/*', (request, reply) => {}))
```

正则表达式路由亦被支持。但要注意，正则表达式会严重拖累性能！
```js
// 正则表达的参数路由
fastify.get('/example/:file(^\\d+).png', (request, reply) => {}))
```

你还可以在同一组斜杠 ("/") 里定义多个参数。就像这样：
```js
fastify.get('/example/near/:lat-:lng/radius/:r', (request, reply) => {}))
```
*使用短横线 ("-") 来分隔参数。*

最后，同时使用多参数和正则表达式也是允许的。
```js
fastify.get('/example/at/:hour(^\\d{2})h:minute(^\\d{2})m', (request, reply) => {}))
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
**警告:** 不能返回 `undefined`。更多细节请看 [promise 取舍](#promise-resolution)。

如你所见，我们不再使用 `reply.send` 向用户发送数据，只需返回消息主体就可以了！

当然，需要的话你还是可以使用 `reply.send` 发送数据。
```js
fastify.get('/', options, async function (request, reply) {
  var data = await getData()
  var processed = await processData(data)
  reply.send(processed)
})
```
**警告:**
* 如果你同时使用 `return` 与 `reply.send`，那么只会发送第一次，同时还会触发警告日志，因为你试图发送两次响应。
* 不能返回 `undefined`。更多细节请看 [promise 取舍]。

<a name="promise-resolution"></a>
### Promise 取舍

假如你的处理器是一个 `async` 函数，或返回了一个 promise，请注意一种必须支持回调函数和 promise 控制流的特殊情况：如果 promise 被 resolve 为 `undefined`，请求会被挂起，并触发一个*错误*日志。

1. 如果你想使用 `async/await` 或 promise，但通过 `reply.send` 返回值：
    - **别** `return` 任何值。
    - **别** 忘了 `reply.send`。
2. 如果你想使用 `async/await` 或 promise：
    - **别** 使用 `reply.send`。
    - **别** 返回 `undefined`。

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
module.exports = function (fastify, opts, next) {
  fastify.get('/user', handler_v1)
  next()
}
```
```js
// routes/v2/users.js
module.exports = function (fastify, opts, next) {
  fastify.get('/user', handler_v2)
  next()
}
```
在编译时 Fastify 自动处理了前缀，因此两个不同路由使用相同的路径名并不会产生问题。*(这也意味着性能一点儿也不受影响！)*。

现在，你的客户端就可以访问下列路由了：
- `/v1/user`
- `/v2/user`

根据需要，你可以多次设置路由前缀，它也支持嵌套的 `register` 以及路由参数。
请注意，当使用了 [`fastify-plugin`](https://github.com/fastify/fastify-plugin) 时，这一选项是无效的。

<a name="custom-log-level"></a>
### 自定义日志级别
在 Fastify 中为路由里设置不同的日志级别是十分容易的。<br/>
你只需在插件或路由的选项里设置 `logLevel` 为相应的[值](https://github.com/pinojs/pino/blob/master/docs/API.md#discussion-3)即可。

要注意的是，如果在插件层面上设置了 `logLevel`，那么 [`setNotFoundHandler`](https://github.com/fastify/docs-chinese/blob/master/docs/Server-Methods.md#setnotfoundhandler) 和 [`setErrorHandler`](https://github.com/fastify/docs-chinese/blob/master/docs/Server-Methods.md#seterrorhandler) 也会受到影响。

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


<a name="routes-config"></a>
### 配置
注册一个新的处理器，你可以向其传递一个配置对象，并在其中使用它。

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
