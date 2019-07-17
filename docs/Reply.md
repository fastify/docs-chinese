<h1 align="center">Fastify</h1>

## 回复
- [回复](#reply)
  - [简介](#introduction)
  - [.code(statusCode)](#codestatuscode)
  - [.header(key, value)](#headerkey-value)
  - [.getHeader(key)](#getheaderkey)
  - [.removeHeader(key)](#removeheaderkey)
  - [.hasHeader(key)](#hasheaderkey)
  - [.redirect(dest)](#redirectdest)
  - [.callNotFound()](#callnotfound)
  - [.getResponseTime()](#getresponsetime)
  - [.type(contentType)](#typecontenttype)
  - [.serializer(func)](#serializerfunc)
  - [.sent](#sent)
  - [.send(data)](#senddata)
    - [对象](#objects)
    - [字符串](#strings)
    - [Streams](#streams)
    - [Buffers](#buffers)
    - [Errors](#errors)
    - [最终 payload 的类型](#type-of-the-final-payload)
    - [Async-Await 与 Promise](#async-await-and-promises)

<a name="introduction"></a>
### 简介
处理函数的第二个参数为 `Reply`。
Reply 是 Fastify 的一个核心对象。它暴露了以下函数及属性：

- `.code(statusCode)` - 设置状态码。
- `.status(statusCode)` - `.code(statusCode)` 的别名。
- `.header(name, value)` - 设置响应 header。
- `.getHeader(name)` - 获取某个 header 的值。
- `.removeHeader(key)` - 清除已设置的 header 的值。
- `.hasHeader(name)` - 检查某个 header 是否设置。
- `.type(value)` - 设置 `Content-Type` header。
- `.redirect([code,] url)` - 重定向至指定的 url，状态码可选 (默认为 `302`)。
- `.callNotFound()` - 调用自定义的 not found 处理函数。
- `.serialize(payload)` - 使用默认的或自定义的 json 序列化工具序列化指定的 payload，并返回处理后的结果。
- `.serializer(function)` - 设置自定义的 payload 序列化工具。
- `.send(payload)` - 向用户发送 payload。类型可以是纯文本、buffer、JSON、stream，或一个 Error 对象。
- `.sent` - 一个 boolean，检查 `send` 是否已被调用。
- `.res` - Node 原生的 [`http.ServerResponse`](https://nodejs.org/dist/latest/docs/api/http.html#http_class_http_serverresponse) 对象。
- `.log` - 请求的日志实例

```js
fastify.get('/', options, function (request, reply) {
  // 你的代码
  reply
    .code(200)
    .header('Content-Type', 'application/json; charset=utf-8')
    .send({ hello: 'world' })
})
```

另外，`Reply` 能够访问请求的上下文：

```js
fastify.get('/', {config: {foo: 'bar'}}, function (request, reply) {
  reply.send('handler config.foo = ' + reply.context.config.foo)
})
```

<a name="code"></a>
### .code(statusCode)
如果没有设置 `reply.code`，`statusCode` 会是 `200`。

<a name="header"></a>
### .header(key, value)
设置响应 header。如果值被省略或为 undefined，将被强制设成 `''`。

更多信息，请看 [`http.ServerResponse#setHeader`](https://nodejs.org/dist/latest/docs/api/http.html#http_response_setheader_name_value)。

<a name="getHeader"></a>
### .getHeader(key)
获取已设置的 header 的值。
```js
reply.header('x-foo', 'foo') // 设置 x-foo header 的值为 foo
reply.getHeader('x-foo') // 'foo'
```

<a name="getHeader"></a>
### .removeHeader(key)

清除已设置的 header 的值。
```js
reply.header('x-foo', 'foo')
reply.removeHeader('x-foo')
reply.getHeader('x-foo') // undefined
```

<a name="hasHeader"></a>
### .hasHeader(key)
返回一个 boolean，用于检查是否设置了某个 header。

<a name="redirect"></a>
### .redirect(dest)
重定向请求至指定的 url，状态码可选，当未通过 `code` 方法设置时，默认为 `302`。
```js
reply.redirect('/home')
```

<a name="call-not-found"></a>
### .callNotFound()
调用自定义的 not found 处理函数。
```js
reply.callNotFound()
```

<a name="getResponseTime"></a>
### .getResponseTime()
调用自定义响应时间获取函数，来计算自收到请求起的时间。

```js
const milliseconds = reply.getResponseTime()
```

<a name="type"></a>
### .type(contentType, type)
设置响应的 content type。
这是 `reply.header('Content-Type', 'the/type')` 的简写。

```js
reply.type('text/html')
```

<a name="serializer"></a>
### .serializer(func)
`.send()` 方法会默认将 `Buffer`、`stream`、`string`、`undefined`、`Error` 之外类型的值 JSON-序列化。假如你需要在特定的请求上使用自定义的序列化工具，你可以通过 `.serializer()` 来实现。要注意的是，如果使用了自定义的序列化工具，你必须同时设置 `'Content-Type'` header。

```js
reply
  .header('Content-Type', 'application/x-protobuf')
  .serializer(protoBuf.serialize)
```

注意，你并不需要在一个 `handler` 内部使用这一工具，因为 Buffers、streams 以及字符串 (除非已经设置了序列化工具) 被认为是已序列化过的。

```js
reply
  .header('Content-Type', 'application/x-protobuf')
  .send(protoBuf.serialize(data))
```

请看 [`.send()`](#send) 了解更多关于发送不同类型值的信息。

<a name="sent"></a>
### .sent

如你所见，`.sent` 属性表明是否已通过 `reply.send()` 发送了一个响应。

当控制器是一个 async 函数或返回一个 promise 时，可以手动设置 `reply.sent = true`，以防 promise resolve 时自动调用 `reply.send()`。通过设置 `reply.sent =
true`，程序能完全掌控底层的请求，且相关钩子不会被触发。

请看范例：

```js
app.get('/', (req, reply) => {
  reply.sent = true
  reply.res.end('hello world')

  return Promise.resolve('this will be skipped') // 译注：该处会被跳过
})
```

如果处理函数 reject，将会记录一个错误。

<a name="send"></a>
### .send(data)
顾名思义，`.send()` 是向用户发送 payload 的函数。

<a name="send-object"></a>
#### 对象
如上文所述，如果你发送 JSON 对象时，设置了输出的 schema，那么 `send` 会使用 [fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) 来序列化对象。否则，将使用 `JSON.stringify()`。
```js
fastify.get('/json', options, function (request, reply) {
  reply.send({ hello: 'world' })
})
```

<a name="send-string"></a>
#### 字符串
在未设置 `Content-Type` 的时候，字符串会以 `text/plain; charset=utf-8` 类型发送。如果设置了 `Content-Type`，且使用自定义序列化工具，那么 `send` 发出的字符串会被序列化。否则，字符串不会有任何改动 (除非 `Content-Type` 的值为 `application/json; charset=utf-8`，这时，字符串会像对象一样被 JSON-序列化，正如上一节所述)。
```js
fastify.get('/json', options, function (request, reply) {
  reply.send('plain string')
})
```

<a name="send-streams"></a>
#### Streams
*send* 开箱即用地支持 stream。它使用 [pump](https://www.npmjs.com/package/pump) 来避免文件描述符 (file descriptors) 的泄露。如果在未设置 `'Content-Type'` header 的情况下发送 stream，它会被设定为 `'application/octet-stream'`。
```js
fastify.get('/streams', function (request, reply) {
  const fs = require('fs')
  const stream = fs.createReadStream('some-file', 'utf8')
  reply.send(stream)
})
```

<a name="send-buffers"></a>
#### Buffers
未设置 `'Content-Type'` header 的情况下发送 buffer，*send* 会将其设置为 `'application/octet-stream'`。
```js
const fs = require('fs')
fastify.get('/streams', function (request, reply) {
  fs.readFile('some-file', (err, fileBuffer) => {
    reply.send(err || fileBuffer)
  })
})
```

<a name="errors"></a>
#### Errors
若使用 *send* 发送一个 *Error* 的实例，Fastify 会自动创建一个如下的错误结构：
```js
{
  error: String        // http 错误信息
  message: String      // 用户错误信息
  statusCode: Number   // http 状态码
}
```
你可以向 Error 对象添加自定义属性，例如 `statusCode` 或 `headers`，这可以用来增强 http 响应。<br>
*注意：如果 `send` 一个错误，但状态码小于 400，Fastify 会自动将其设为 500。*

贴士：你可以通过 [`http-errors`](https://npm.im/http-errors) 或 [`fastify-sensible`](https://github.com/fastify/fastify-sensible) 来简化生成的错误：

```js
fastify.get('/', function (request, reply) {
  reply.send(httpErrors.Gone())
})
```

如果你想完全自定义错误处理，请看 [`setErrorHandler`](https://github.com/fastify/docs-chinese/blob/master/docs/Server.md#seterrorhandler) API。<br>
*注：当自定义错误处理时，你需要自行记录日志*

API:

 ```js
fastify.setErrorHandler(function (error, request, reply) {
  request.log.warn(error)
  var statusCode = error.statusCode >= 400 ? error.statusCode : 500
  reply
    .code(statusCode)
    .type('text/plain')
    .send(statusCode >= 500 ? 'Internal server error' : error.message)
})
```

路由生成的 not found 错误会使用 [`setNotFoundHandler`](https://github.com/fastify/docs-chinese/blob/master/docs/Server.md#setnotfoundhandler)。
API：

```js
fastify.setNotFoundHandler(function (request, reply) {
  reply
    .code(404)
    .type('text/plain')
    .send('a custom not found')
})
```

<a name="payload-type"></a>
#### 最终 payload 的类型
发送的 payload (序列化之后、经过任意的 [`onSend` 钩子](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md#the-onsend-hook)) 必须为下列类型之一，否则将会抛出一个错误：

- `string`
- `Buffer`
- `stream`
- `undefined`
- `null`

<a name="async-await-promise"></a>
#### Async-Await 与 Promise
Fastify 原生地处理 promise 并支持 async-await。<br>
*请注意，在下面的例子中我们没有使用 reply.send。*
```js
const delay = promisify(setTimeout)

fastify.get('/promises', options, function (request, reply) {
  return delay(200).then(() => { return { hello: 'world' }})
})

fastify.get('/async-await', options, async function (request, reply) {
  await delay(200)
  return { hello: 'world' }
})
```

被 reject 的 promise 默认发送 `500` 状态码。要修改回复，可以 reject 一个 promise，或在 `async 函数` 中进行 `throw` 操作，同时附带上一个有 `statusCode` (或 `status`) 与 `message` 属性的对象。

```js
fastify.get('/teapot', async function (request, reply) => {
  const err = new Error()
  err.statusCode = 418
  err.message = 'short and stout'
  throw err
})
```

想要了解更多？请看 [Routes#async-await](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md#async-await)。
