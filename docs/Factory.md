<h1 align="center">Fastify</h1>

<a name="factory"></a>
## 工厂函数

Fastify 模块导出了一个工厂函数，可以用于创建新的<a href="https://github.com/fastify/docs-chinese/blob/master/docs/Server-Methods.md"><code><b> Fastify server</b></code></a> 实例。这个工厂函数的参数是一个配置对象，用于自定义最终生成的实例。本文描述了这一对象中可用的属性。

<a name="factory-http2"></a>
### `http2` (实验性)

设置为 `true`，则会使用 Node.js 原生的 [HTTP/2](https://nodejs.org/dist/latest-v8.x/docs/api/http2.html) 模块来绑定 socket。

+ 默认值: `false`

<a name="factory-https"></a>
### `https`

用于配置服务器的 TLS socket 的对象。其选项与 Node.js 原生的 [`createServer` 方法](https://nodejs.org/dist/latest-v8.x/docs/api/https.html#https_https_createserver_options_requestlistener)一致。
当值为 `null` 时，socket 连接将不会配置 TLS。

当 <a href="https://github.com/fastify/docs-chinese/blob/master/docs/Factory.md#factory-http2">
<code><b>http2</b></code>
</a> 选项设置时，`https` 选项也会被应用。

+ 默认值: `null`

<a name="factory-ignore-slash"></a>
### `ignoreTrailingSlash`

Fastify 使用 [find-my-way](https://github.com/delvedor/find-my-way) 处理路由。该选项为 `true` 时，尾斜杠将被省略。
这一选项应用于 server 实例上注册的*所有*路由。

+ 默认值: `false`

```js
const fastify = require('fastify')({
  ignoreTrailingSlash: true
})

// 同时注册 "/foo" 与 "/foo/"
fastify.get('/foo/', function (req, reply) {
  res.send('foo')
})

// 同时注册 "/bar" 与 "/bar/"
fastify.get('/bar', function (req, reply) {
  res.send('bar')
})
```

<a name="factory-max-param-length"></a>
### `maxParamLength`
你可以为通过 `maxParamLength` 选项为带参路由 (无论是标准的、正则匹配的，还是复数的) 设置最大参数长度。选项的默认值为 100 字符。<br>
当使用正则匹配的路由时，这非常有用，可以帮你抵御 [DoS 攻击](https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS)。<br>
*当达到长度限制时，将触发 not found 路由。*

<a name="factory-body-limit"></a>
### `bodyLimit`

定义服务器可接受的最大 payload，以字节为单位。

+ 默认值: `1048576` (1MiB)

<a name="factory-logger"></a>
### `logger`

Fastify 依托 [Pino](https://getpino.io/) 内建了一个日志工具。该属性用于配置日志实例。

属性可用的值为：

+ 默认: `false`。禁用日志。所有记录日志的方法将会指向一个空日志工具 [abstract-logging](https://npm.im/abstract-logging) 的实例。

+ `pinoInstance`: 一个已被实例化的 Pino 实例。内建的日志工具将指向这个实例。

+ `object`: 标准的 Pino [选项对象](https://github.com/pinojs/pino/blob/c77d8ec5ce/docs/API.md#constructor)。
它会被直接传递进 Pino 的构造函数。如果下列属性未在该对象中定义，它们将被相应地添加：
    * `genReqId`: 一个同步函数，用于生成请求的标识符。默认生成按次序排列的标识符。
    * `level`: 最低的日志级别。若未被设置，则默认为 `'info'`。
    * `serializers`: 序列化函数的哈希。默认情况下，序列化函数应用在 `req` (来访的请求对象)、`res` (发送的响应对象) 以及 `err` (标准的 `Error` 对象) 之上。当一个日志方法接收到含有上述任意属性的对象时，对应的序列化器将会作用于该属性。举例如下：
        ```js
        fastify.get('/foo', function (req, res) {
          req.log.info({req}) // 日志输出经过序列化的请求对象
          res.send('foo')
        })
        ```
      用户提供的序列化函数将会覆盖对应属性默认的序列化函数。

<a name="custom-http-server"></a>
### `serverFactory`
通过 `serverFactory` 选项，你可以向 Fastify 传递一个自定义的 http server。<br/>
`serverFactory` 函数的参数为 `handler` 函数及一个选项对象。`handler` 函数的参数为 `request` 和 `response` 对象，选项对象则与你传递给 Fastify 的一致。

```js
const serverFactory = (handler, opts) => {
  const server = http.createServer((req, res) => {
    handler(req, res)
  })

  return server
}

const fastify = Fastify({ serverFactory })

fastify.get('/', (req, reply) => {
  reply.send({ hello: 'world' })
})

fastify.listen(3000)
```
Fastify 内在地使用 Node 原生 http server 的 API。因此，如果你使用一个自定义的 server，你必须保证暴露了相同的 API。不这么做的话，你可以在 `serverFactory` 函数内部 `return` 语句之前，向 server 实例添加新的属性。

<a name="factory-case-sensitive"></a>
### `caseSensitive`

默认值为 `true`，此时路由对大小写敏感。这就意味着 `/foo` 与 `/Foo` 是两个不同的路由。当该选项为 `false` 时，路由大小写不敏感，`/foo`、`/Foo` 以及 `/FOO` 都是一样的。

将 `caseSensitive` 设置为 `false` 也会导致所有路由参数 (包括正则匹配的值) 变为小写。

```js
fastify.get('/user/:username', (request, reply) => {
  // 原 URL: /user/NodeJS
  console.log(request.params.username) // -> 'nodejs'
})
```

要注意的是，将该选项设为 `false` 与 [RFC3986](https://tools.ietf.org/html/rfc3986#section-6.2.2.1) 相悖。

<a name="factory-request-id-header"></a>
### `requestIdHeader`

用来获知请求 id 的 header 名。请看[请求 id](./Logging.md#logging-request-id) 一节。

+ 默认值: `'request-id'`
