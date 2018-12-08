<h1 align="center">Fastify</h1>

<a name="factory"></a>
## 工厂函数

Fastify 模块导出了一个工厂函数，可以用于创建新的<a href="https://github.com/fastify/docs-chinese/blob/master/docs/Server.md"><code><b> Fastify server</b></code></a> 实例。这个工厂函数的参数是一个配置对象，用于自定义最终生成的实例。本文描述了这一对象中可用的属性。

<a name="factory-http2"></a>
### `http2` (实验性)

设置为 `true`，则会使用 Node.js 原生的 [HTTP/2](https://nodejs.org/dist/latest-v8.x/docs/api/http2.html) 模块来绑定 socket。

+ 默认值: `false`

<a name="factory-https"></a>
### `https`

用于配置服务器的 TLS socket 的对象。其选项与 Node.js 原生的 [`createServer` 方法](https://nodejs.org/dist/latest-v8.x/docs/api/https.html#https_https_createserver_options_requestlistener)一致。
当值为 `null` 时，socket 连接将不会配置 TLS。

当 <a href="https://github.com/fastify/docs-chinese/blob/master/docs/Server.md#factory-http2">
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

用来获知请求 id 的 header 名。请看[请求 id](https://github.com/fastify/docs-chinese/blob/master/docs/Logging.md#logging-request-id) 一节。

+ 默认值: `'request-id'`

<a name="factory-gen-request-id"></a>
### `genReqId`
用于生成请求 id 的函数。参数为来访的请求对象。

+ 默认值: `'request-id' 的值 (当存在该 header 时) 或单调递增的整数`
在分布式系统中，你可能会特别想覆盖如下默认的 id 生成行为。要生成 `UUID`，请看[hyperid](https://github.com/mcollina/hyperid)。
 ```js
let i = 0
const fastify = require('fastify')({
  genReqId: function (req) { return i++ }
})

<a name="factory-trust-proxy"></a>
### `trustProxy`

通过开启 `trustProxy` 选项，Fastify 会认为使用了代理服务，且 `X-Forwarded-*` header 是可信的，否则该值被认为是极具欺骗性的。

```js
const fastify = Fastify({ trustProxy: true })
```

+ 默认值: `false`
+ `true/false`: 信任所有代理 (`true`) 或不信任任意的代理 (`false`)。
+ `string`: 只信任给定的 IP/CIDR (例如 `'127.0.0.1'`)。可以是一组用英文逗号分隔的地址 (例如 `'127.0.0.1,192.168.1.1/24'`)。
+ `Array<string>`: 只信任给定的 IP/CIDR 列表 (例如 `['127.0.0.1']`)。
+ `number`: 信任来自前置代理服务器的第n跳 (hop) 地址作为客户端。
+ `Function`: 自定义的信任函数，第一个参数为 `address`
    ```js
    function myTrustFn(address, hop) {
      return address === '1.2.3.4' || hop === 1
    }
    ```

更多示例详见 [proxy-addr](https://www.npmjs.com/package/proxy-addr)。

你还可以通过原始的 `request` 对象获取 `ip` 与 `hostname` 的值。

```js
fastify.get('/', (request, reply) => {
  console.log(request.raw.ip)
  console.log(request.raw.hostname)
})
```

<a name="plugin-timeout"></a>
### `pluginTimeout`

单个插件允许加载的最长时间，以毫秒计。如果某个插件加载超时，则 [`ready`](https://github.com/fastify/fastify/blob/master/docs/Server.md#ready) 会抛出一个含有 `'ERR_AVVIO_PLUGIN_TIMEOUT'` 代码的 `Error` 对象。

+ 默认值: `10000`

## 实例

### 服务器方法

<a name="server"></a>
#### 服务器
`fastify.server`：由 [**`Fastify 的工厂函数`**](https://github.com/fastify/docs-chinese/blob/master/docs/Server.md) 生成的 Node 原生 [server](https://nodejs.org/api/http.html#http_class_http_server) 对象。

<a name="after"></a>
#### after
当前插件及在其中注册的所有插件加载完毕后调用。总在 `fastify.ready` 之前执行。

```js
fastify
  .register((instance, opts, next) => {
    console.log('当前插件')
    next()
  })
  .after(err => {
    console.log('当前插件之后')
  })
  .register((instance, opts, next) => {
    console.log('下一个插件')
    next()
  })
  .ready(err => {
    console.log('万事俱备')
  })
```

<a name="ready"></a>
#### ready
当所有插件的加载都完成时调用。如有错误发生，它会传递一个 `error` 参数。
```js
fastify.ready(err => {
  if (err) throw err
})
```
调用时不加参数，它会返回一个 `Promise` 对象：

```js
fastify.ready().then(() => {
  console.log('successfully booted!')
}, (err) => {
  console.log('an error happened', err)
})
```

<a name="listen"></a>
#### listen
所有的插件加载完毕、`ready` 事件触发后，在指定的端口启动服务器。它的回调函数与 Node 原生方法的回调相同。默认情况下，服务器监听 `localhost` 所决定的地址 (`127.0.0.1` 或 `::1`，取决于操作系统)。将地址设置为 `0.0.0.0` 可监听所有的 IPV4 地址。设置为 `::` 则可监听所有的 IPV6 地址，在某些系统中，这么做亦可同时监听所有 IPV4 地址。监听所有的接口要格外谨慎，因为这种方式存在着固有的[安全风险](https://web.archive.org/web/20170831174611/https://snyk.io/blog/mongodb-hack-and-secure-defaults/)。

```js
fastify.listen(3000, (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

指定监听的地址：

```js
fastify.listen(3000, '127.0.0.1', (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

指定积压队列 (backlog queue size) 的大小：

```js
fastify.listen(3000, '127.0.0.1', 511, (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

没有提供回调函数时，它会返回一个 Promise 对象：

```js
fastify.listen(3000)
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

你还可以在使用 Promise 的同时指定地址：

```js
fastify.listen(3000, '127.0.0.1')
  .then((address) => console.log(`server listening on ${address}`))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

当部署在 Docker 或其它容器上时，明智的做法是监听 `0.0.0.0`。因为默认情况下，这些容器并未将映射的端口暴露在 `127.0.0.1`：

```js
fastify.listen(3000, '0.0.0.0', (err, address) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

假如未设置 `port` (或设为 0)，则会自动选择一个随机可用的端口 (之后可通过 `fastify.server.address().port` 获知)。

<a name="route"></a>
#### route
将路由添加到服务器的方法，支持简写。请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md)。

<a name="close"></a>
#### close
`fastify.close(callback)`：调用这个函数来关闭服务器实例，并触发 [`'onClose'`](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md#on-close) 钩子。<br>
服务器会向所有新的请求发送 `503` 错误，并销毁它们。

<a name="decorate"></a>
#### decorate*
向 Fastify 实例、响应或请求添加装饰器函数。参阅[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Decorators.md)了解更多。

<a name="register"></a>
#### register
Fastify 允许用户通过插件扩展功能。插件可以是一组路由、装饰器或其他。请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md)。

<a name="use"></a>
#### use
向 Fastify 添加中间件，请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Middlewares.md)。

<a name="addHook"></a>
#### addHook
向 Fastify 添加特定的生命周期钩子函数，请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md)。

<a name="prefix"></a>
#### prefix
添加在路由前的完整路径。

示例：

```js
fastify.register(function (instance, opts, next) {
  instance.get('/foo', function (request, reply) {
    // 输出："prefix: /v1"
    request.log.info('prefix: %s', instance.prefix)
    reply.send({prefix: instance.prefix})
  })

  instance.register(function (instance, opts, next) {
    instance.get('/bar', function (request, reply) {
      // 输出："prefix: /v1/v2"
      request.log.info('prefix: %s', instance.prefix)
      reply.send({prefix: instance.prefix})
    })

    next()
  }, { prefix: '/v2' })

  next()
}, { prefix: '/v1' })
```

<a name="log"></a>
#### log
日志的实例，详见[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Logging.md)。

<a name="inject"></a>
#### inject
伪造 http 注入 (作为测试之用) 。请看[更多内容](https://github.com/fastify/docs-chinese/blob/master/docs/Testing.md#inject)。

<a name="add-schema"></a>
#### addSchema
`fastify.addSchema(schemaObj)`，向 Fastify 实例添加可共用的 schema，用于验证数据。你可以通过该 schema 的 id 在应用的任意位置使用它。<br/>
请看[验证和序列化](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md)一文中的[范例](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md#shared-schema)。
<a name="set-schema-compiler"></a>
#### setSchemaCompiler
为所有的路由设置 schema 编译器 (schema compiler)，请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md#schema-compiler)了解更多信息。

<a name="set-not-found-handler"></a>
#### setNotFoundHandler

`fastify.setNotFoundHandler(handler(request, reply))`：为 404 状态 (not found) 设置处理器 (handler) 函数。向 `fastify.register()` 传递不同的 [`prefix` 选项](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md#route-prefixing-option)，就可以为不同的插件设置不同的处理器。这些处理器被视为常规的路由处理器，因此它们的请求会经历一个完整的 [Fastify 生命周期](https://github.com/fastify/docs-chinese/blob/master/docs/Lifecycle.md#lifecycle)。

你也可以为 404 处理器注册一个 [beforeHandler](https://www.fastify.io/docs/latest/Hooks/#beforehandler) 钩子。

```js
fastify.setNotFoundHandler({
  beforeHandler: (req, reply, next) => {
    req.body.beforeHandler = true
    next()
  }  
}, function (request, reply) {
    // 设置了 beforeHandler 钩子的默认 not found 处理器
})

fastify.register(function (instance, options, next) {
  instance.setNotFoundHandler(function (request, reply) {
    // '/v1' 开头的 URL 的 not found 处理器，
    // 未设置 beforeHandler 钩子
  })
  next()
}, { prefix: '/v1' })
```

<a name="set-error-handler"></a>
#### setErrorHandler

`fastify.setErrorHandler(handler(error, request, reply))`：设置任意时刻的错误处理器函数。错误处理器是完全封装 (fully encapsulated) 的，因此不同插件的处理器可以不同。支持 *async-await* 语法。

```js
fastify.setErrorHandler(function (error, request, reply) {
  // 发送错误响应
})
```

<a name="print-routes"></a>
#### printRoutes

`fastify.printRoutes()`：打印路由的基数树 (radix tree)，可作调试之用。<br/>
*记得在 `ready` 函数的内部或之后调用它。*

```js
fastify.get('/test', () => {})
fastify.get('/test/hello', () => {})
fastify.get('/hello/world', () => {})

fastify.ready(() => {
  console.log(fastify.printRoutes())
  // └── /
  //   ├── test (GET)
  //   │   └── /hello (GET)
  //   └── hello/world (GET)
})
```
