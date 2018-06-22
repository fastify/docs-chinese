<h1 align="center">Fastify</h1>

## HTTP2

_Fastify_ 提供了从 Node 8.8.0 开始的对 HTTP2 **实验性支持**. _Fastify_ 支持 HTTPS 和普通文本的 HTTP2 支持.

当前没有任何 HTTP2 相关的 APIs 是可用的, 但 Node `req` 和 `res` 可以通过 `Request` 和 `Reply` 接口访问. 欢迎相关的 PR.

### 安全 (HTTPS)

所有的现代浏览器都__只能通过安全的连接__ 支持 HTTP2:

```js
'use strict'

const fs = require('fs')
const path = require('path')
const fastify = require('fastify')({
  http2: true,
  https: {
    key: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.key')),
    cert: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.cert'))
  }
})

fastify.get('/', function (request, reply) {
  reply.code(200).send({ hello: 'world' })
})

fastify.listen(3000)
```

ALPN 协商允许在同一个 socket 上支持 HTTPS 和 HTTP/2.
Node 核心 `req` 和 `res` 对象可以是 [HTTP/1](https://nodejs.org/api/http.html)
或者 [HTTP/2](https://nodejs.org/api/http2.html).
_Fastify_ 自带支持开箱即用:

```js
'use strict'

const fs = require('fs')
const path = require('path')
const fastify = require('fastify')({
  http2: true,
  https: {
    allowHTTP1: true, // fallback support for HTTP1
    key: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.key')),
    cert: fs.readFileSync(path.join(__dirname, '..', 'https', 'fastify.cert'))
  }
})

// this route can be accessed through both protocols
fastify.get('/', function (request, reply) {
  reply.code(200).send({ hello: 'world' })
})

fastify.listen(3000)
```

你可以像这样测试你的新服务器:

```
$ npx h2url https://localhost:3000
```

### 纯文本或者不安全

如果你搭建微服务, 你可以纯文本连接 HTTP2, 但是浏览器不支持这样做.

```js
'use strict'

const fastify = require('fastify')({
  http2: true
})

fastify.get('/', function (request, reply) {
  reply.code(200).send({ hello: 'world' })
})

fastify.listen(3000)
```

你可以像这样测试你的新服务器:

```
$ npx h2url http://localhost:3000
```

