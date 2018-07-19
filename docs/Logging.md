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
你可以在 [Pino 的文档](https://github.com/pinojs/pino/blob/master/docs/API.md#pinooptions-stream)中找到全部选项。如果你想向 Pino 传送自定义流 (stream)，仅需在 `logger` 对象中添加 `stream` 一项即可。

```js
const split = require('split2')
const stream = split(JSON.parse)

const fastify = require('fastify')({
  logger: {
    level: 'info',
    stream: stream
  }
})

fastify.get('/', options, function (request, reply) {
  request.log.info('Some info about the current request')
  reply.send({ hello: 'world' })
})
```

<a name="logging-request-id" />
默认情况下，Fastify 给每个请求分配了一个 id 以便跟踪。如果头部存在 "request-id" 即使用该值，否则会生成一个新的增量 id。你可以通过 Fastify 工厂函数的 [`requestIdHeader`](./Factory.md#factory-request-id-header) 选项来自定义该头部的名称。
此外，你还可以通过 `genReqId` 选项生成自定义的请求 id。它的参数是来访的请求。
```js
let i = 0
const fastify = require('fastify')({
  logger: {
    genReqId: function (req) { return i++ }
  }
})
```

默认的日志工具使用标准的序列化工具，生成包括 `req`、`res` 与 `err` 属性在内的序列化对象。可以借由指定自定义的序列化工具来改变这一行为。
```js
const fastify = require('fastify')({
  logger: {
    serializers: {
      req: function (req) {
        return { url: req.url }
      }
    }
  }
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

*当前请求的日志实例在[生命周期](https://github.com/fastify/docs-chinese/blob/master/docs/Lifecycle.md)的各部分均可使用。*
