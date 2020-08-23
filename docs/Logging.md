<h1 align="center">Fastify</h1>

## 日志

日志默认关闭，你可以在创建 Fastify 实例时传入 `{ logger: true }` 或者 `{ logger: { level: 'info' } }` 选项来开启它。要注意的是，日志无法在运行时启用。为此，我们使用了
[abstract-logging](https://www.npmjs.com/package/abstract-logging)。

Fastify 专注于性能，因此使用了 [pino](https://github.com/pinojs/pino) 作为日志工具。默认的日志级别为 `'info'`。

开启日志相当简单：

```js
const fastify = require('fastify')({
  logger: true
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

如果你想为日志配置选项，直接将选项传递给 Fastify 实例就可以了。
你可以在 [Pino 的文档](https://github.com/pinojs/pino/blob/master/docs/api.md#pinooptions-stream)中找到全部选项。如果你想指定文件地址，可以：

```js
const fastify = require('fastify')({
  logger: {
    level: 'info',
    file: '/path/to/file' // 将调用 pino.destination()
  }
})
fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

如果需要向 Pino 传送自定义流 (stream)，仅需在 `logger` 对象中添加 `stream` 一项即可。

```js
const split = require('split2')
const stream = split(JSON.parse)

const fastify = require('fastify')({
  logger: {
    level: 'info',
    stream: stream
  }
})
```

<a name="logging-request-id"></a>
默认情况下，Fastify 给每个请求分配了一个 id 以便跟踪。如果头部存在 "request-id" 即使用该值，否则会生成一个新的增量 id。你可以通过 Fastify 工厂函数的 [`requestIdHeader`](Server.md#factory-request-id-header) 与 [`genReqId`](Server.md#gen-request-id) 来进行自定义。

默认的日志工具使用标准的序列化工具，生成包括 `req`、`res` 与 `err` 属性在内的序列化对象。`req` 对象是 Fastify [`Request`](https://github.com/fastify/fastify/blob/master/docs/Request.md) 对象，而 `res` 则是 Fastify [`Reply`](https://github.com/fastify/fastify/blob/master/docs/Reply.md) 对象。可以借由指定自定义的序列化工具来改变这一行为。
```js
const fastify = require('fastify')({
  logger: {
    serializers: {
      req (request) {
        return { url: request.url }
      }
    }
  }
})
```
响应的 payload 与 header 可以按如下方式记录日志 (即便这是*不推荐*的做法)：

```js
const fastify = require('fastify')({
  logger: {
    prettyPrint: true,
    serializers: {
      res (reply) {
        // 默认
        return {
          statusCode: reply.statusCode
        }
      },
      req (request) {
        return {
          method: request.method,
          url: request.url,
          path: request.path,
          parameters: request.parameters,
          // 记录 header 可能会触犯隐私法律，例如 GDPR (译注：General Data Protection Regulation)。你应该用 "redact" 选项来移除敏感的字段。此外，验证数据也可能在日志中泄露。
          headers: request.headers
        };
      }
    }
  }
});
```
**注**：在 `req` 方法中，body 无法被序列化。因为请求是在创建子日志时就序列化了，而此时 body 尚未被解析。

以下是记录 `req.body` 的一个方法

```js
app.addHook('preHandler', function (req, reply, done) {
  if (req.body) {
    req.log.info({ body: req.body }, 'parsed body')
  }
  done()
})
```

*Pino 之外的日志工具会忽略该选项。*

你还可以提供自定义的日志实例。直接将实例传入，取代配置选项就能实现该功能。提供的示例必须实现 Pino 的接口，换句话说，便是拥有下列方法：
`info`、`error`、`debug`、`fatal`、`warn`、`trace`、`child`。

示例:

```js
const log = require('pino')({ level: 'info' })
const fastify = require('fastify')({ logger: log })

log.info('does not have request information')

fastify.get('/', function (request, reply) {
  request.log.info('includes request information, but is the same logger instance as `log`')
  reply.send({ hello: 'world' })
})
```

*当前请求的日志实例在[生命周期](Lifecycle.md)的各部分均可使用。*

## 日志修订
 
[Pino](https://getpino.io) 支持低开销的日志修订，以隐藏特定内容。
举例来说，出于安全方面的考虑，我们也许想在 HTTP header 的日志中隐藏 `Authorization` 这一个 header：

```js
const fastify = Fastify({
  logger: {
    stream: stream,
    redact: ['req.headers.authorization'],
    level: 'info',
    serializers: {
      req (request) {
        return {
          method: request.method,
          url: request.url,
          headers: request.headers,
          hostname: request.hostname,
          remoteAddress: request.ip,
          remotePort: request.connection.remotePort
        }
      }
    }
  }
})
```

更多信息请看 https://getpino.io/#/docs/redaction。
