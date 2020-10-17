<h1 align="center">Fastify</h1>

## `Content-Type` 解析
Fastify 原生只支持 `'application/json'` 和 `'text/plain'` content types。默认的字符集是 `utf-8`。如果你需要支持其他的 content types，你需要使用 `addContentTypeParser` API。*默认的 JSON 或者纯文本解析器也可以被更改.*

和其他的 API 一样，`addContentTypeParser` 被封装在定义它的作用域中了。这就意味着如果你定义在了根作用域中，那么就是全局可用，如果你定义在一个插件中，那么它只能在那个作用域和子作用域中可用。

Fastify 自动将解析好的 payload 添加到 [Fastify request](Request.md) 对象，你能通过 `request.body` 访问。

### 用法
```js
fastify.addContentTypeParser('application/jsoff', function (request, payload, done) {
  jsoffParser(payload, function (err, body) {
    done(err, body)
  })
})

// 以相同方式处理多种 content type
fastify.addContentTypeParser(['text/xml', 'application/xml'], function (request, payload, done) {
  xmlParser(payload, function (err, body) {
    done(err, body)
  })
})

// Node 版本 >= 8.0.0 时也支持 async
fastify.addContentTypeParser('application/jsoff', async function (request, payload) {
  var res = await jsoffParserAsync(payload)

  return res
})

// 可以为不同的 content type 使用默认的 JSON/Text 解析器
fastify.addContentTypeParser('text/json', { parseAs: 'string' }, fastify.getDefaultJsonParser('ignore', 'ignore'))
```

你也可以用 `hasContentTypeParser` API 来验证某个 content type 解析器是否存在。

```js
if (!fastify.hasContentTypeParser('application/jsoff')){
  fastify.addContentTypeParser('application/jsoff', function (request, payload, done) {
    jsoffParser(payload, function (err, body) {
      done(err, body)
    })
  })
}
```

**注意**：早先的写法 `function(req, done)` 与 `async function(req)` 仍被支持，但不推荐使用。

#### Body Parser

你可以用两种方式解析消息主体。第一种方法在上面演示过了: 你可以添加定制的 content type 解析器来处理请求。第二种方法你可以在 `addContentTypeParser`  API 传递 `parseAs` 参数。它可以是 `'string'` 或者 `'buffer'`。如果你使用 `parseAs` 选项 Fastify 会处理 stream 并且进行一些检查，比如消息主体的 [最大尺寸](Factory.md#factory-body-limit) 和消息主体的长度。如果达到了某些限制，自定义的解析器就不会被调用。

```js
fastify.addContentTypeParser('application/json', { parseAs: 'string' }, function (req, body, done) {
  try {
    var json = JSON.parse(body)
    done(null, json)
  } catch (err) {
    err.statusCode = 400
    done(err, undefined)
  }
})
```

查看例子 [`example/parser.js`](https://github.com/fastify/fastify/blob/master/examples/parser.js)。

##### 自定义解析器的选项
+ `parseAs` (string): `'string'` 或者 `'buffer'` 定义了如何收集进来的数据。默认是 `'buffer'`。
+ `bodyLimit` (number): 自定义解析器能够接收的最大的数据长度，比特为单位。默认是全局的消息主体的长度限制[`Fastify 工厂方法`](Factory.md#bodylimit)。

#### 捕获所有
有些情况下你需要捕获所有的 content type。通过 Fastify，你只需添加`'*'` content type。
```js
fastify.addContentTypeParser('*', function (request, payload, done) {
  var data = ''
  payload.on('data', chunk => { data += chunk })
  payload.on('end', () => {
    done(null, data)
  })
})
```
在这种情况下，所有的没有特定 content type 解析器的请求都会被这个方法处理。

对请求流 (stream) 执行管道输送 (pipe) 操作也是有用的。你可以如下定义一个 content 解析器：
```js
fastify.addContentTypeParser('*', function (request, payload, done) {
  done()
})
```
之后通过核心 HTTP request 对象将请求流直接输送到任意位置：
```js
app.post('/hello', (request, reply) => {
  reply.send(request.raw)
})
```
这里有一个将来访的 [json line](http://jsonlines.org/) 对象完整输出到日志的例子：
```js
const split2 = require('split2')
const pump = require('pump')
 fastify.addContentTypeParser('*', (request, payload, done) => {
  done(null, pump(payload, split2(JSON.parse)))
})
fastify.route({
  method: 'POST',
  url: '/api/log/jsons',
  handler: (req, res) => {
    req.body.on('data', d => console.log(d)) // 记录每个来访的对象
  }
})
```
关于输送上传的文件，请看[该插件](https://github.com/fastify/fastify-multipart)。