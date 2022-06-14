<h1 align="center">Fastify</h1>

<a name="factory"></a>
## 工厂函数

Fastify 模块导出了一个工厂函数，可以用于创建新的 <code><b>Fastify server</b></code> 实例。这个工厂函数的参数是一个配置对象，用于自定义最终生成的实例。本文描述了这一对象中可用的属性。

- [http2](./Server.md#http2)
- [https](./Server.md#https)
- [connectionTimeout](./Server.md#connectiontimeout)
- [keepAliveTimeout](./Server.md#keepalivetimeout)
- [maxRequestsPerSocket](./Server.md#maxRequestsPerSocket)
- [requestTimeout](./Server.md#requestTimeout)
- [ignoreTrailingSlash](./Server.md#ignoretrailingslash)
- [maxParamLength](./Server.md#maxparamlength)
- [onProtoPoisoning](./Server.md#onprotopoisoning)
- [onConstructorPoisoning](./Server.md#onconstructorpoisoning)
- [logger](./Server.md#logger)
- [serverFactory](./Server.md#serverfactory)
- [jsonShorthand](./Server.md#jsonshorthand)
- [caseSensitive](./Server.md#casesensitive)
- [requestIdHeader](./Server.md#requestidheader)
- [requestIdLogLabel](./Server.md#requestidloglabel)
- [genReqId](./Server.md#genreqid)
- [trustProxy](./Server.md#trustProxy)
- [pluginTimeout](./Server.md#plugintimeout)
- [querystringParser](./Server.md#querystringparser)
- [exposeHeadRoutes](./Server.md#exposeheadroutes)
- [constraints](./Server.md#constraints)
- [return503OnClosing](./Server.md#return503onclosing)
- [ajv](./Server.md#ajv)
- [serializerOpts](./Server.md#serializeropts)
- [http2SessionTimeout](./Server.md#http2sessiontimeout)
- [frameworkErrors](./Server.md#frameworkerrors)
- [clientErrorHandler](./Server.md#clienterrorhandler)
- [rewriteUrl](./Server.md#rewriteurl)
- [实例](./Server.md#instance)
- [服务器方法](./Server.md#server-methods)
- [initialConfig](./Server.md#initialConfig)

<a name="factory-http2"></a>
### `http2`

设置为 `true`，则会使用 Node.js 原生的 [HTTP/2](https://nodejs.org/dist/latest-v14.x/docs/api/http2.html) 模块来绑定 socket。

+ 默认值：`false`

<a name="factory-https"></a>
### `https`

用于配置服务器的 TLS socket 的对象。其选项与 Node.js 原生的 [`createServer` 方法](https://nodejs.org/dist/latest-v14.x/docs/api/https.html#https_https_createserver_options_requestlistener)一致。
当值为 `null` 时，socket 连接将不会配置 TLS。

当 <a href="./Server.md#factory-http2">
<code><b>http2</b></code>
</a> 选项设置时，`https` 选项也会被应用。

+ 默认值：`null`

<a name="factory-connection-timeout"></a>
### `connectionTimeout`

定义服务器超时，单位为毫秒。作用请见 [`server.timeout` 属性](https://nodejs.org/api/http.html#http_server_timeout)的文档。当指定了 `serverFactory` 时，该选项被忽略。

+ 默认值：`0` (无超时)

<a name="factory-keep-alive-timeout"></a>
### `keepAliveTimeout`

定义服务器 keep-alive 超时，单位为毫秒。作用请见 [`server.keepAliveTimeout` 属性](https://nodejs.org/api/http.html#http_server_timeout)的文档。仅当使用 HTTP/1 时有效。当指定了 `serverFactory` 时，该选项被忽略。

+ 默认值：`5000` (5 秒)

<a name="factory-max-requests-per-socket"></a>
### `maxRequestsPerSocket`

定义套接字在关闭 keep-alive 的连接前，可处理的最大请求数。要了解该选项的效果，参阅 [`server.maxRequestsPerSocket` 属性](https://nodejs.org/dist/latest/docs/api/http.html#http_server_maxrequestspersocket)的文档。该选项仅用于 HTTP/1.1 的连接，且当指定了 `serverFactory` 选项时会被忽略。
> 此文撰写时，只有 16.0.0 及以上版本的 Node.js 支持该选项。兼容性请参阅 Node.js 的文档。

+ 默认值：`0` (无限制)

<a name="factory-request-timeout"></a>
### `requestTimeout`

定义接收客户端请求的超时，单位为毫秒。要了解该选项的效果，参阅 [`server.requestTimeout` 属性](https://nodejs.org/dist/latest/docs/api/http.html#http_server_requesttimeout)的文档。当指定了 `serverFactory` 选项时会被忽略。当服务器之前没有反向代理时，该选项的值必须设置为非零值 (例如 120 秒)，以防范潜在的拒绝服务型 (Denial-of-Service) 攻击。
> 此文撰写时，只有 14.11.0 及以上版本的 Node.js 支持该选项。兼容性请参阅 Node.js 的文档。

+ 默认值：`0` (无限制)

<a name="factory-ignore-slash"></a>
### `ignoreTrailingSlash`

Fastify 使用 [find-my-way](https://github.com/delvedor/find-my-way) 处理路由。该选项为 `true` 时，尾斜杠将被省略。
这一选项应用于 server 实例上注册的*所有*路由。

+ 默认值：`false`

```js
const fastify = require('fastify')({
  ignoreTrailingSlash: true
})

// 同时注册 "/foo" 与 "/foo/"
fastify.get('/foo/', function (req, reply) {
  reply.send('foo')
})

// 同时注册 "/bar" 与 "/bar/"
fastify.get('/bar', function (req, reply) {
  reply.send('bar')
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

+ 默认值：`1048576` (1MiB)

<a name="factory-on-proto-poisoning"></a>
### `onProtoPoisoning`

由 [secure-json-parse](https://github.com/fastify/secure-json-parse) 提供的功能，指定解析带有 `__proto__` 键的 JSON 对象时框架的行为。
更多关于原型污染 (prototype poisoning) 的内容请看 https://hueniverse.com/a-tale-of-prototype-poisoning-2610fa170061。

允许的值为 `'error'`、`'remove'` 与 `'ignore'`。

+ 默认值：`'error'`

<a name="factory-on-constructor-poisoning"></a>
### `onConstructorPoisoning`

由 [secure-json-parse](https://github.com/fastify/secure-json-parse) 提供的功能，指定解析带有 `constructor` 的 JSON 对象时框架的行为。
更多关于原型污染的内容请看 https://hueniverse.com/a-tale-of-prototype-poisoning-2610fa170061。

允许的值为 `'error'`、`'remove'` 与 `'ignore'`。

+ 默认值：`'error'`

<a name="factory-logger"></a>
### `logger`

Fastify 依托 [Pino](https://getpino.io/) 内建了一个日志工具。该属性用于配置日志实例。

属性可用的值为：

+ 默认: `false`。禁用日志。所有记录日志的方法将会指向一个空日志工具 [abstract-logging](https://npm.im/abstract-logging) 的实例。

+ `pinoInstance`: 一个已被实例化的 Pino 实例。内建的日志工具将指向这个实例。

+ `object`: 标准的 Pino [选项对象](https://github.com/pinojs/pino/blob/c77d8ec5ce/docs/API.md#constructor)。
它会被直接传递进 Pino 的构造函数。如果下列属性未在该对象中定义，它们将被相应地添加：
    * `level`: 最低的日志级别。若未被设置，则默认为 `'info'`。
    * `serializers`: 序列化函数的哈希。默认情况下，序列化函数应用在 `req` (来访的请求对象)、`res` (发送的响应对象) 以及 `err` (标准的 `Error` 对象) 之上。当一个日志方法接收到含有上述任意属性的对象时，对应的序列化器将会作用于该属性。举例如下：
        ```js
        fastify.get('/foo', function (req, res) {
          req.log.info({req}) // 日志输出经过序列化的请求对象
          res.send('foo')
        })
        ```
      用户提供的序列化函数将会覆盖对应属性默认的序列化函数。
+ `loggerInstance`：自定义日志工具实例。日志工具必须实现 Pino 的接口，即拥有如下方法：`info`, `error`, `debug`, `fatal`, `warn`, `trace`, `child`。例如：
```js
const pino = require('pino')();

const customLogger = {
  info: function (o, ...n) {},
  warn: function (o, ...n) {},
  error: function (o, ...n) {},
  fatal: function (o, ...n) {},
  trace: function (o, ...n) {},
  debug: function (o, ...n) {},
  child: function() {
    const child = Object.create(this);
    child.pino = pino.child(...arguments);
    return child;
  },
};

const fastify = require('fastify')({logger: customLogger});
```

<a name="factory-disable-request-logging"></a>
### `disableRequestLogging`
默认情况下当开启日志时，Fastify 会在收到请求与发送该请求的响应时记录 `info` 级别的日志。你可以设置该选项为 `true` 来禁用该功能。这时，通过自定义 `onRequest` 和 `onResponse` 钩子，你能更灵活地记录一个请求的开始与结束。

+ 默认值：`false`

```js
// 例子：通过钩子再造被禁用的请求日志功能。
fastify.addHook('onRequest', (req, reply, done) => {
  req.log.info({ url: req.raw.url, id: req.id }, 'received request')
  done()
})

fastify.addHook('onResponse', (req, reply, done) => {
  req.log.info({ url: req.raw.originalUrl, statusCode: reply.raw.statusCode }, 'request completed')
  done()
})
```

请注意，该选项同时也会禁止默认的 `onResponse` 钩子在响应的回调函数出错时记录错误日志。

<a name="custom-http-server"></a>
### `serverFactory`
通过 `serverFactory` 选项，你可以向 Fastify 传递一个自定义的 HTTP server。<br/>
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
Fastify 内在地使用 Node 原生 HTTP server 的 API。因此，如果你使用一个自定义的 server，你必须保证暴露了相同的 API。不这么做的话，你可以在 `serverFactory` 函数内部 `return` 语句之前，向 server 实例添加新的属性。<br/>

<a name="schema-json-shorthand"></a>
### `jsonShorthand`

+ 默认值：`true`

当未发现 JSON Schema 规范中合法的根属性时，Fastify 会默认地自动推断该根属性。如果你想实现自定义的 schema 校验编译器，例如使用 JTD (JSON Type Definition) 代替 JSON schema，你应当设置该选项为 `false` 来确保 schema 不被修改且不会被当成 JSON Schema 处理。

```js
const AjvJTD = require('ajv/dist/jtd'/* 只在 AJV v7 以上版本生效 */)
const ajv = new AjvJTD({
  // 当遇到非法 JTD schema 对象时抛出错误。
  allErrors: process.env.NODE_ENV === 'development'
})
const fastify = Fastify({ jsonShorthand: false })
fastify.setValidatorCompiler(({ schema }) => {
  return ajv.compile(schema)
})
fastify.post('/', {
  schema: {
    body: {
      properties: {
        foo: { type: 'uint8' }
      }
    }
  },
  handler (req, reply) { reply.send({ ok: 1 }) }
})
```

**注：目前 Fastify 并不会在发现非法 schema 时抛错。因此，假如你在已有项目中关闭了该选项，请确保现存所有的 schema 不会因此变得非法，因为它们会被当作笼统的一类进行处理。**

<a name="factory-case-sensitive"></a>
### `caseSensitive`

默认值为 `true`，此时路由对大小写敏感。这就意味着 `/foo` 与 `/Foo` 是两个不同的路由。当该选项为 `false` 时，路由大小写不敏感，`/foo`、`/Foo` 以及 `/FOO` 都是一样的。

将 `caseSensitive` 设置为 `false`，会导致所有路径变为小写，除了路由参数与通配符。

```js
fastify.get('/user/:username', (request, reply) => {
  // 原 URL: /USER/NodeJS
  console.log(request.params.username) // -> 'NodeJS'
})
```

要注意的是，将该选项设为 `false` 与 [RFC3986](https://tools.ietf.org/html/rfc3986#section-6.2.2.1) 相悖。

此外，该选项不影响 query string 的解析。要让 query string 忽略大小写，请看 [`querystringParser`](./Server.md#querystringparser)。

<a name="factory-request-id-header"></a>
### `requestIdHeader`

用来获知请求 ID 的 header 名。请看[请求 ID](Logging.md#logging-request-id) 一节。

+ 默认值：`'request-id'`

<a name="factory-request-id-log-label"></a>
### `requestIdLogLabel`

定义日志中请求 ID 的标签。

+ 默认值：`'reqId'`

<a name="factory-gen-request-id"></a>
### `genReqId`
用于生成请求 ID 的函数。参数为来访的请求对象。

+ 默认值：`'request-id' header 的值 (当存在时) 或单调递增的整数`
在分布式系统中，你可能会特别想覆盖如下默认的 ID 生成行为。要生成 `UUID`，请看[hyperid](https://github.com/mcollina/hyperid)。
 ```js
let i = 0
const fastify = require('fastify')({
  genReqId: function (req) { return i++ }
})
```

**注意：当设置了 <code>[requestIdHeader](#requestidheader)</code> 中定义的 header (默认为 'request-id') 时，genReqId _不会_ 被调用。**

<a name="factory-trust-proxy"></a>
### `trustProxy`

通过开启 `trustProxy` 选项，Fastify 会认为使用了代理服务，且 `X-Forwarded-*` header 是可信的，否则该值被认为是极具欺骗性的。

```js
const fastify = Fastify({ trustProxy: true })
```

+ 默认值：`false`
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

更多示例详见 [`proxy-addr`](https://www.npmjs.com/package/proxy-addr)。

你还可以通过 [`request`](Request.md) 对象获取 `ip`、`ips`、`hostname` 与 `protocol` 的值。

```js
fastify.get('/', (request, reply) => {
  console.log(request.ip)
  console.log(request.ips)
  console.log(request.hostname)
  console.log(request.protocol)
})
```

**注：如果请求存在多个 <code>x-forwarded-host</code> 或 <code>x-forwarded-proto</code> header，只会根据最后一个产生 <code>request.hostname</code> 和 <code>request.protocol</code>**

<a name="plugin-timeout"></a>
### `pluginTimeout`

单个插件允许加载的最长时间，以毫秒计。如果某个插件加载超时，则 [`ready`](Server.md#ready) 会抛出一个含有 `'ERR_AVVIO_PLUGIN_TIMEOUT'` 代码的 `Error` 对象。

+ 默认值：`10000`

 <a name="factory-querystring-parser"></a>
 ### `querystringParser`
 
Fastify 默认使用 Node.js 核心的 `querystring` 模块作为 query string 解析器。<br/>
你可以通过 `querystringParser` 选项来使用自定义的解析器，例如 [`qs`](https://www.npmjs.com/package/qs)。

```js
const qs = require('qs')
const fastify = require('fastify')({
  querystringParser: str => qs.parse(str)
})
```

你也可以改变默认解析器的行为，例如忽略 query string 的大小写：

```js
const querystring = require('querystring')
const fastify = require('fastify')({
  querystringParser: str => querystring.parse(str.toLowerCase())
})
```

若你只想忽略键的大小写，我们推荐你使用自定义解析器。

<a name="exposeHeadRoutes"></a>
### `exposeHeadRoutes`

自动为每个 `GET` 路由添加对应的 `HEAD` 路由。如果你不想禁用该选项，又希望自定义 `HEAD` 处理函数，请在 `GET` 路由前定义该处理函数。

+ 默认值：`false`

<a name="constraints"></a>
### `constraints`

Fastify 内建的路由约束由 `find-my-way` 支持，允许使用 `version` 或 `host` 来约束路由。通过为 `find-my-way` 提供 `constraints` 对象，你可以添加新的约束策略，或覆盖原有策略。更多内容可见于 [find-my-way](https://github.com/delvedor/find-my-way) 的文档。

```js
const customVersionStrategy = {
  storage: function () {
    let versions = {}
    return {
      get: (version) => { return versions[version] || null },
      set: (version, store) => { versions[version] = store },
      del: (version) => { delete versions[version] },
      empty: () => { versions = {} }
    }
  },
  deriveVersion: (req, ctx) => {
    return req.headers['accept']
  }
}

const fastify = require('fastify')({
  constraints: {
    version: customVersionStrategy
  }
})
```

<a name="factory-return-503-on-closing"></a>
### `return503OnClosing`

调用 `close` 方法后返回 503 状态码。
如果为 `false`，服务器会正常处理请求。

+ 默认值：`true`

<a name="factory-ajv"></a>
### `ajv`

配置 Fastify 使用的 Ajv 6 实例。这使得你无需提供一个自定义的实例。

+ 默认值：

```js
{
  customOptions: {
    removeAdditional: true,
    useDefaults: true,
    coerceTypes: true,
    allErrors: false,
    nullable: true
  },
  plugins: []
}
```

```js
const fastify = require('fastify')({
  ajv: {
    customOptions: {
      nullable: false // 参见 [ajv 的配置选项](https://github.com/ajv-validator/ajv/tree/v6#options)
    },
    plugins: [
      require('ajv-merge-patch'),
      [require('ajv-keywords'), 'instanceof']
      // 用法： [plugin, pluginOptions] - 插件与选项
      // 用法： plugin - 仅插件
    ]
  }
})
```

<a name="serializer-opts"></a>
### `serializerOpts`

自定义用于序列化响应 payload 的 [`fast-json-stringify`](https://github.com/fastify/fast-json-stringify#options) 实例的配置：

```js
const fastify = require('fastify')({
  serializerOpts: {
    rounding: 'ceil'
  }
})
```

<a name="http2-session-timeout"></a>
### `http2SessionTimeout`

为每个 HTTP/2 会话设置默认[超时时间](https://nodejs.org/api/http2.html#http2_http2session_settimeout_msecs_callback)。超时后，会话将关闭。默认值：`5000` 毫秒。

要注意的是，使用 HTTP/2 时需要提供一个优雅的“close”体验。一个低的默认值有助于减轻拒绝服务型攻击 (Denial-of-Service Attacks) 的影响。但当你的服务器使用负载均衡策略，或能自动扩容时，则可以延长超时时间。Node 的默认值为 `0`，即无超时。

<a name="framework-errors"></a>
### `frameworkErrors`

+ 默认值：`null`

对于最常见的场景，Fastify 已经提供了默认的错误处理方法。这个选项允许你重写这些处理方法。

*注：目前只实现了 `FST_ERR_BAD_URL` 这个错误。*

```js
const fastify = require('fastify')({
  frameworkErrors: function (error, req, res) {
    if (error instanceof FST_ERR_BAD_URL) {
      res.code(400)
      return res.send("Provided url is not valid")
    } else {
      res.send(err)
    }
  }
})
```

<a name="client-error-handler"></a>
### `clientErrorHandler`

设置 [clientErrorHandler](https://nodejs.org/api/http.html#http_event_clienterror) 来监听客户端连接造成的 `error` 事件，并响应 `400` 状态码。

设置了该选项，会覆盖默认的 `clientErrorHandler`。

+ 默认值：
```js
function defaultClientErrorHandler (err, socket) {
  if (err.code === 'ECONNRESET') {
    return
  }

  const body = JSON.stringify({
    error: http.STATUS_CODES['400'],
    message: 'Client Error',
    statusCode: 400
  })
  this.log.trace({ err }, 'client error')

  if (socket.writable) {
    socket.end(`HTTP/1.1 400 Bad Request\r\nContent-Length: ${body.length}\r\nContent-Type: application/json\r\n\r\n${body}`)
  }
}
```

*注：`clientErrorHandler` 使用底层的 socket，故处理函数需要返回格式正确的 HTTP 响应信息，包括状态行、HTTP header 以及 body。在写入之前，为了避免 socket 已被销毁，你还应该检查 socket 是否依然可写。*

```js
const fastify = require('fastify')({
  clientErrorHandler: function (err, socket) {
    const body = JSON.stringify({
      error: {
        message: 'Client error',
        code: '400'
      }
    })
    // `this` 为 fastify 实例
    this.log.trace({ err }, 'client error')
    // 处理函数应当发送正确的 HTTP 响应信息。
    socket.end(`HTTP/1.1 400 Bad Request\r\nContent-Length: ${body.length}\r\nContent-Type: application/json\r\n\r\n${body}`)
  }
})
```

<a name="rewrite-url"></a>
### `rewriteUrl`

设置一个异步函数，返回一个字符串，用于重写 URL。

> 重写 URL 会修改 `req` 对象的 `url` 属性

```js
function rewriteUrl (req) { // req 是 Node.js 的 HTTP 请求对象
  return req.url === '/hi' ? '/hello' : req.url;
}
```

要注意的是，`rewriteUrl` 在处理路由 _前_ 被调用，它不是封装的，而是整个应用级别的。

## 实例

### 服务器方法

<a name="server"></a>
#### 服务器
`fastify.server`：由 [**`Fastify 的工厂函数`**](Server.md) 生成的 Node 原生 [server](https://nodejs.org/api/http.html#http_class_http_server) 对象。

<a name="after"></a>
#### after
当前插件及在其中注册的所有插件加载完毕后调用。总在 `fastify.ready` 之前执行。

```js
fastify
  .register((instance, opts, done) => {
    console.log('当前插件')
    done()
  })
  .after(err => {
    console.log('当前插件之后')
  })
  .register((instance, opts, done) => {
    console.log('下一个插件')
    done()
  })
  .ready(err => {
    console.log('万事俱备')
  })
```

当 `after()` 没有回调参数时，它返回一个 `Promise`：

```js
fastify.register(async (instance, opts) => {
  console.log('Current plugin')
})

await fastify.after()
console.log('After current plugin')

fastify.register(async (instance, opts) => {
  console.log('Next plugin')
})

await fastify.ready()

console.log('Everything has been loaded')
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

<a name="getDefaultRoute"></a>
#### getDefaultRoute
获取服务器 `defaultRoute` 属性的方法：

```js
const defaultRoute = fastify.getDefaultRoute()
```

<a name="setDefaultRoute"></a>
#### setDefaultRoute
设置服务器 `defaultRoute` 属性的方法：

```js
const defaultRoute = function (req, res) {
  res.end('hello world')
}

fastify.setDefaultRoute(defaultRoute)
```

<a name="routing"></a>
#### routing
访问内部路由库的 `lookup` 方法，该方法将请求匹配到合适的处理函数：

```js
fastify.routing(req, res)
```

<a name="route"></a>
#### route
将路由添加到服务器的方法，支持简写。请看[这里](Routes.md)。

<a name="close"></a>
#### close
`fastify.close(callback)`：调用这个函数来关闭服务器实例，并触发 [`'onClose'`](Hooks.md#on-close) 钩子。<br>
服务器会向所有新的请求发送 `503` 错误，并销毁它们。
要改变这一行为，请见 [`return503OnClosing`](Server.md#factory-return-503-on-closing)。

如果无参调用，它会返回一个 Promise：

 ```js
fastify.close().then(() => {
  console.log('successfully closed!')
}, (err) => {
  console.log('an error happened', err)
})
```

<a name="decorate"></a>
#### decorate*
向 Fastify 实例、响应或请求添加装饰器函数。参阅[这里](Decorators.md)了解更多。

<a name="register"></a>
#### register
Fastify 允许用户通过插件扩展功能。插件可以是一组路由、装饰器或其他。请看[这里](Plugins.md)。

<a name="addHook"></a>
#### addHook
向 Fastify 添加特定的生命周期钩子函数，请看[这里](Hooks.md)。

<a name="prefix"></a>
#### prefix
添加在路由前的完整路径。

示例：

```js
fastify.register(function (instance, opts, done) {
  instance.get('/foo', function (request, reply) {
    // 输出："prefix: /v1"
    request.log.info('prefix: %s', instance.prefix)
    reply.send({prefix: instance.prefix})
  })

  instance.register(function (instance, opts, done) {
    instance.get('/bar', function (request, reply) {
      // 输出："prefix: /v1/v2"
      request.log.info('prefix: %s', instance.prefix)
      reply.send({prefix: instance.prefix})
    })

    done()
  }, { prefix: '/v2' })

  done()
}, { prefix: '/v1' })
```

<a name="pluginName"></a>
#### pluginName
当前插件的名称。有三种定义插件名称的方式（按顺序）。

1. 如果插件使用 [fastify-plugin](https://github.com/fastify/fastify-plugin)，那么名称为元数据 (metadata) 中的 `name`。
2. 如果插件通过 `module.exports` 导出，使用文件名。
3. 如果插件通过常规的 [函数定义](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#Defining_functions)，则使用函数名。

*回退方案*：插件函数的头两行将作为插件名，并使用 `--` 替代换行符。这有助于在处理涉及许多插件的问题时，找到根源。

重点：如果你要处理一些通过 [fastify-plugin](https://github.com/fastify/fastify-plugin) 包装的嵌套的异名插件，由于没有生成新的定义域，因此不会去覆盖上下文数据，而是将各插件名加入一个数组。在这种情况下，会按涉及到的插件的启动顺序，以 `plugin-A -> plugin-B` 的格式来展示插件名称。

<a name="log"></a>
#### log
日志的实例，详见[这里](Logging.md)。

<a name="version"></a>
#### version
Fastify 实例的版本。可在插件中使用。详见[插件](Plugins.md#handle-the-scope)一文。

<a name="inject"></a>
#### inject
伪造 HTTP 注入 (作为测试之用) 。请看[更多内容](Testing.md#inject)。

<a name="add-schema"></a>
#### addSchema
`fastify.addSchema(schemaObj)`，向 Fastify 实例添加 JSON schema。你可以通过 `$ref` 关键字在应用的任意位置使用它。<br/>
更多内容，请看[验证和序列化](Validation-and-Serialization.md)。

<a name="get-schemas"></a>
#### getSchemas
`fastify.getSchemas()`，返回一个对象，包含所有通过 `addSchema` 添加的 schema，对象的键是 JSON schema 的 `$id`。

<a name="get-schema"></a>
#### getSchema
`fastify.getSchema(id)`，返回通过 `addSchema` 添加的拥有匹配 `id` 的 schema，未找到则返回 `undefined`。

<a name="set-reply-serializer"></a>
#### setReplySerializer
作用于未设置 [Reply.serializer(func)](Reply.md#serializerfunc) 的所有路由的默认序列化方法。这个处理函数是完全封装的，因此，不同的插件允许有不同的错误处理函数。
注：仅当状态码为 `2xx` 时才被调用。关于错误处理，请看 [`setErrorHandler`](Server.md#seterrorhandler)。

 ```js
fastify.setReplySerializer(function (payload, statusCode){
  // 使用同步函数序列化 payload
  return `my serialized ${statusCode} content: ${payload}`
})
```

<a name="set-validator-compiler"></a>
#### setValidatorCompiler
为所有的路由设置 schema 校验编译器 (validator compiler)。详见 [#schema-validator](Validation-and-Serialization.md#schema-validator)。

<a name="set-schema-error-formatter"></a>
#### setSchemaErrorFormatter
为所有的路由设置 schema 错误格式化器 (schema error formatter)。详见 [#error-handling](Validation-and-Serialization.md#schemaerrorformatter)。

<a name="set-serializer-resolver"></a>
#### setSerializerCompiler
为所有的路由设置 schema 序列化编译器 (serializer compiler)。详见 [#schema-serializer](Validation-and-Serialization.md#schema-serializer)。
**注：** [`setReplySerializer`](#set-reply-serializer) 有更高的优先级！

<a name="validator-compiler"></a>
#### validatorCompiler
该属性用于获取 schema 校验器。未设置校验器时，在服务器启动前，该值是 `null`，之后是一个签名为 `function ({ schema, method, url, httpPart })` 的函数。该函数将 `schema` 参数编译为一个校验数据的函数，并返回生成的函数。
`schema` 参数能访问到所有通过 [`.addSchema`](#add-schema) 添加的共用 schema。

<a name="serializer-compiler"></a>
#### serializerCompiler
该属性用于获取 schema 序列化器。未设置序列化器时，在服务器启动前，该值是 `null`，之后是一个签名为 `function ({ schema, method, url, httpPart })` 的函数。该函数将 `schema` 参数编译为一个校验数据的函数，并返回生成的函数。
`schema` 参数能访问到所有通过 [`.addSchema`](#add-schema) 添加的共用 schema。

<a name="schema-error-formatter"></a>
#### schemaErrorFormatter
该属性设置一个函数用于格式化 `validationCompiler` 在校验 schema 时发生的错误。详见 [#error-handling](Validation-and-Serialization.md#schemaerrorformatter)。

<a name="schema-controller"></a>
#### schemaController
该属性用于管理：
- `bucket`：应用的 schema 的存放位置
- `compilersFactory`：必须编译 JSON schema 的模块

可用于 Fastify 无法分辨保存于某些数据结构中的 schema 之时。在 [issue #2446](https://github.com/fastify/fastify/issues/2446) 里有一个通过该属性解决问题的例子。

另一个用例是微调所有 schema 的处理过程。这么做可以用 Ajv 8 替代默认的 Ajv 6！例子见下文。

```js
const fastify = Fastify({
  schemaController: {
    /**
     * 以下的 factory 函数会在每次调用 `fastify.register()` 时被执行。
     * 若父级上下文添加了 schema，它会作为参数传入 factory 函数。
     * @param {object} parentSchemas 会由 `bucket` 对象的 `getSchemas()` 方法返回。

     */
    bucket: function factory (parentSchemas) {
      return {
        addSchema (inputSchema) {
          // 该函数保存用户添加的 schema。
          // 调用 `fastify.addSchema()` 时被执行。
        },
        getSchema (schema$id) {
          // 该函数返回通过 `schema$id` 检索得到的原始 schema。
          // 调用 `fastify.getSchema()` 时被执行。
          return aSchema
        },
        getSchemas () {
          // 返回路由 schema 中通过 $ref 引用的所有 schema。
          // 返回对象以 schema 的 `$id` 为键，以原始内容为值。
          const allTheSchemaStored = {
            'schema$id1': schema1,
            'schema$id2': schema2
          }
          return allTheSchemaStored
        }
      }
    },

    /**
     * 编译器的 factory 函数让你能充分地控制 Fastify 生命周期里的验证器与序列化器，并给你的编译器提供封装。
     */
    compilersFactory: {
      /**
       * 以下的 factory 函数会在每次需要一个新的验证器实例时被执行。
       * 当新的 schema 被添加到封装上下文时，调用 `fastify.register()` 也会执行该函数。
       * 若父级上下文添加了 schema，它会作为参数传入 factory 函数。
       * @param {object} externalSchemas 这些 schema 将被 `bucket.getSchemas()` 返回。需要处理外部引用 $ref。
       * @param {object} ajvServerOption 服务器的 `ajv` 选项。
       */
      buildValidator: function factory (externalSchemas, ajvServerOption) {
        // 该 factory 函数必须返回一个 schema 校验编译器。
        // 详见 [#schema-validator](Validation-and-Serialization.md#schema-validator)。
        const yourAjvInstance = new Ajv(ajvServerOption.customOptions)
        return function validatorCompiler ({ schema, method, url, httpPart }) {
          return yourAjvInstance.compile(schema)
        }
      },

      /**
       * 以下的 factory 函数会在每次需要一个新的序列化器实例时被执行。
       * 当新的 schema 被添加到封装上下文时，调用 `fastify.register()` 也会执行该函数。
       * 若父级上下文添加了 schema，它会作为参数传入 factory 函数。
       * @param {object} externalSchemas 这些 schema 将被 `bucket.getSchemas()` 返回。需要处理外部引用 $ref。
       * @param {object} serializerOptsServerOption 服务器的 `serializerOpts` 选项。
       */
      buildSerializer: function factory (externalSchemas, serializerOptsServerOption) {
        // 该 factory 函数必须返回一个 schema 序列化编译器。
        // 详见 [#schema-serializer](Validation-and-Serialization.md#schema-serializer)。
        return function serializerCompiler ({ schema, method, url, httpStatus }) {
          return data => JSON.stringify(data)
        }
      }
    }
  }
});
```

##### 将 Ajv 8 作为默认的 schema 验证器

Ajv 8 是 Ajv 6 的后继版本，拥有许多改进与新特性。Ajv 8 新特性 (如 JTD 与 Standalone 模式) 的用法详见 [`@fastify/ajv-compiler` 的文档](https://github.com/fastify/ajv-compiler#usage)。

下列代码可以将 Ajv 8 作为默认的 schema 验证器使用：

```js
const AjvCompiler = require('@fastify/ajv-compiler') // 必须是 v2.x.x 版本

// 请注意，默认情况下 Ajv 8 不支持 schema 的关键词 `format`，
// 因此需要手动添加
const ajvFormats = require('ajv-formats')

const app = fastify({
  ajv: {
    customOptions: {
      validateFormats: true
    },
    plugins: [ajvFormats]
  },
  schemaController: {
    compilersFactory: {
      buildValidator: AjvCompiler()
    }
  }
})

// 大功告成！现在你可以在 schema 中使用 Ajv 8 的选项与关键词了！
```

<a name="set-not-found-handler"></a>
#### setNotFoundHandler

`fastify.setNotFoundHandler(handler(request, reply))`：为 404 状态 (not found) 设置处理函数 (handler)。向 `fastify.register()` 传递不同的 [`prefix` 选项](Plugins.md#route-prefixing-option)，就可以为不同的插件设置不同的处理函数。这些处理函数被视为常规的路由处理函数，因此它们的请求会经历一个完整的 [Fastify 生命周期](Lifecycle.md#lifecycle)。

你也可以为 404 处理函数注册 [`preValidation`](Hooks.md/#route-hooks) 或 [`preHandler`](Hooks.md/#route-hooks) 钩子。

_注：通过此方法注册的 `preValidation` 钩子会在遇到未知路由时触发，但手动调用 [`reply.callNotFound`](Reply.md#call-not-found) 方法时则**不会**_。此时只有 preHandler 会执行。

```js
fastify.setNotFoundHandler({
  preValidation: (req, reply, done) => {
    // 你的代码
    done()
  } ,
  preHandler: (req, reply, done) => {
    // 你的代码
    done()
  }  
}, function (request, reply) {
    // 设置了 preValidation 与 preHandler 钩子的默认 not found 处理函数
})

fastify.register(function (instance, options, done) {
  instance.setNotFoundHandler(function (request, reply) {
    // '/v1' 开头的 URL 的 not found 处理函数，
    // 未设置 preValidation 与 preHandler 钩子
  })
  done()
}, { prefix: '/v1' })
```

Fastify 启动时，会在插件注册之前就调用 `setNotFoundHandler` 方法添加默认的 404 处理函数。假如你想拓展默认 404 处理函数的行为，例如与插件一同使用，你可以在插件上下文内，不传参数地调用 `fastify.setNotFoundHandler()`。

<a name="set-error-handler"></a>
#### setErrorHandler

`fastify.setErrorHandler(handler(error, request, reply))`：设置任意时刻的错误处理函数。错误处理函数绑定在 Fastify 实例之上，是完全封装 (fully encapsulated) 的，因此不同插件的处理函数可以不同。支持 *async-await* 语法。<br>
*注：假如错误的 `statusCode` 小于 400，在处理错误前 Fastify 将会自动将其设为 500。*

```js
fastify.setErrorHandler(function (error, request, reply) {
  // 记录错误
  this.log.error(error)
  // 发送错误响应
  reply.status(409).send({ ok: false })
})
```

当没有设置错误处理函数时，Fastify 会调用一个默认函数。你能通过 `fastify.errorHandler` 访问该函数。它根据 `statusCode` 相应地记录日志。

```js
var statusCode = error.statusCode
if (statusCode >= 500) {
  log.error(error)
} else if (statusCode >= 400) {
  log.info(error)
} else {
  log.error(error)
}
```

<a name="print-routes"></a>
#### printRoutes

`fastify.printRoutes()`：打印路由的基数树 (radix tree)，可作调试之用。可以用 `fastify.printRoutes({ commonPrefix: false })` 来打印扁平化后的路由<br/>
*记得在 `ready` 函数的内部或之后调用它。*

```js
fastify.get('/test', () => {})
fastify.get('/test/hello', () => {})
fastify.get('/hello/world', () => {})

fastify.ready(() => {
  console.log(fastify.printRoutes())
  // └── /
  //     ├── test (GET)
  //     │   └── /hello (GET)
  //     └── hel
  //         ├── lo/world (GET)
  //         └── licopter (GET)

  console.log(fastify.printRoutes({ commonPrefix: false }))
  // └── / (-)
  //     ├── test (GET)
  //     │   └── /hello (GET)
  //     ├── hello/world (GET)
  //     └── helicopter (GET)
})
```

`fastify.printRoutes({ includeMeta: (true | []) })` 会打印出路由的 `route.store` 对象上的属性。`includeMeta` 的值可以是属性名的数组 (例如：`['onRequest', Symbol('key')]`)，也可以只是一个 `true`，表示显示所有属性。简写 `fastify.printRoutes({ includeHooks: true })` 将包含所有的[钩子](Hooks.md)。

```js
  console.log(fastify.printRoutes({ includeHooks: true, includeMeta: ['metaProperty'] }))
  // └── /
  //     ├── test (GET)
  //     │   • (onRequest) ["anonymous()","namedFunction()"]
  //     │   • (metaProperty) "value"
  //     │   └── /hello (GET)
  //     └── hel
  //         ├── lo/world (GET)
  //         │   • (onTimeout) ["anonymous()"]
  //         └── licopter (GET)
  
  console.log(fastify.printRoutes({ includeHooks: true }))
  // └── /
  //     ├── test (GET)
  //     │   • (onRequest) ["anonymous()","namedFunction()"]  
  //     │   └── /hello (GET)
  //     └── hel
  //         ├── lo/world (GET)
  //         │   • (onTimeout) ["anonymous()"]
  //         └── licopter (GET)
```

<a name="print-plugins"></a>
#### printPlugins

`fastify.printPlugins()`：打印 avvio 内部的插件树，可用于调试插件注册顺序相关的问题。<br/>
*请在 `ready` 事件的回调中或事件触发之后调用该方法。*

```js
fastify.register(async function foo (instance) {
  instance.register(async function bar () {})
})
fastify.register(async function baz () {})

fastify.ready(() => {
  console.error(fastify.printPlugins())
  // 输出：
  // └── root
  //     ├── foo
  //     │   └── bar
  //     └── baz
})
```

<a name="addContentTypeParser"></a>
#### addContentTypeParser

`fastify.addContentTypeParser(content-type, options, parser)` 用于给指定 content type 自定义解析器，当你使用自定义的 content types 时会很有帮助。例如 `text/json, application/vnd.oasis.opendocument.text`。`content-type` 是一个字符串、字符串数组或正则表达式。

```js
// 传递给 getDefaultJsonParser 的两个参数用于配置原型污染以及构造函数污染，允许的值为 'ignore'、'remove' 和 'error'。设置为 ignore 会跳过校验，和直接调用 JSON.parse() 效果相同。详见 <a href="https://github.com/fastify/secure-json-parse#api">`secure-json-parse` 的文档</a>。

fastify.addContentTypeParser('text/json', { asString: true }, fastify.getDefaultJsonParser('ignore', 'ignore'))
```

<a name="getDefaultJsonParser"></a>
#### getDefaultJsonParser

`fastify.getDefaultJsonParser(onProtoPoisoning, onConstructorPoisoning)` 接受两个参数。第一个参数是原型污染的配置，第二个则是构造函数污染的配置。详见 <a href="https://github.com/fastify/secure-json-parse#api">`secure-json-parse` 的文档</a>。

<a name="defaultTextParser"></a>
#### defaultTextParser

`fastify.defaultTextParser()` 可用于将 content 解析为纯文本。

```js
fastify.addContentTypeParser('text/json', { asString: true }, fastify.defaultTextParser())
```

<a name="errorHandler"></a>
#### errorHandler

`fastify.errorHandler` 使用 Fastify 默认的错误处理函数来处理错误。

```js
fastify.get('/', {
  errorHandler: (error, request, reply) => {
    if (error.code === 'SOMETHING_SPECIFIC') {
      reply.send({ custom: 'response' })
      return
    }

    fastify.errorHandler(error, request, response)
  }
}, handler)
```

<a name="initial-config"></a>
#### initialConfig

`fastify.initialConfig`：暴露一个记录了 Fastify 初始选项的只读对象。

当前暴露的属性有：
- connectionTimeout
- keepAliveTimeout
- bodyLimit
- caseSensitive
- http2
- https (返回 `false`/`true`。当特别指明时，返回 `{ allowHTTP1: true/false }`)
- ignoreTrailingSlash
- disableRequestLogging
- maxParamLength
- onProtoPoisoning
- onConstructorPoisoning
- pluginTimeout
- requestIdHeader
- requestIdLogLabel
- http2SessionTimeout

```js
const { readFileSync } = require('fs')
const Fastify = require('fastify')

const fastify = Fastify({
  https: {
    allowHTTP1: true,
    key: readFileSync('./fastify.key'),
    cert: readFileSync('./fastify.cert')
  },
  logger: { level: 'trace'},
  ignoreTrailingSlash: true,
  maxParamLength: 200,
  caseSensitive: true,
  trustProxy: '127.0.0.1,192.168.1.1/24',
})

console.log(fastify.initialConfig)
/*
输出：
{
  caseSensitive: true,
  https: { allowHTTP1: true },
  ignoreTrailingSlash: true,
  maxParamLength: 200
}
*/

fastify.register(async (instance, opts) => {
  instance.get('/', async (request, reply) => {
    return instance.initialConfig
    /*
    返回：
    {
      caseSensitive: true,
      https: { allowHTTP1: true },
      ignoreTrailingSlash: true,
      maxParamLength: 200
    }
    */
  })

  instance.get('/error', async (request, reply) => {
    // 会抛出错误
    // 因为 initialConfig 是只读的，不可修改
    instance.initialConfig.https.allowHTTP1 = false

    return instance.initialConfig
  })
})

// 开始监听
fastify.listen(3000, (err) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```
