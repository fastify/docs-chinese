<h1 align="center">Fastify</h1>

## Request
处理函数的第一个参数是 `Request`.<br>
Request 是 Fastify 的核心对象，包含了以下字段:
- `query` - 解析后的 querystring，其格式由 [`querystringParser`](Server.md#querystringparser) 指定。
- `body` - 请求主体。[Content Type 解析器](ContentTypeParser.md)一文中列举了 Fastify 原生支持的 content type 类型，并阐明了如何支持其他类型。
- `params` - URL 参数
- [`headers`](#headers) - header 的 getter 与 setter
- `raw` - Node 原生的 HTTP 请求
- `req` *(不推荐，请使用 `.raw`)* - Node 原生的 HTTP 请求
- `server` - Fastify 服务器的实例，以当前的[封装上下文](Encapsulation.md)为作用域。
- `id` - 请求 ID
- `log` - 请求的日志实例
- `ip` - 请求方的 ip 地址
- `ips` - x-forwarder-for header 中保存的请求源 ip 数组，按访问先后排序 (仅当 [`trustProxy`](Server.md#factory-trust-proxy) 开启时有效)
- `hostname` - 请求方的主机名 (当 [`trustProxy`](Server.md#factory-trust-proxy) 启用时，从 `X-Forwarded-Host` header 中获取)。为了兼容 HTTP/2，当没有相关 header 存在时，将返回 `:authority`。
- `protocol` - 请求协议 (`https` 或 `http`)
- `method` - 请求方法
- `url` - 请求路径
- `routerMethod` - 处理请求的路由函数
- `routerPath` - 处理请求的路由的匹配模式
- `is404` - 当请求被 404 处理时为 true，反之为 false
- `connection` - 不推荐，请使用 `socket`。请求的底层连接
- `socket` - 请求的底层连接
- `context` - Fastify 内建的对象。你不应该直接使用或修改它，但可以访问它的下列特殊属性：
  - `context.config` - 路由的 [`config`](Routes.md#routes-config) 对象。

### Headers

`request.headers` 返回来访请求的 header 对象。你也可以如下设置自定义的 header：

```js
request.headers = {
  'foo': 'bar',
  'baz': 'qux'
}
```

该操作能向请求 header 添加新的值，且该值能通过 `request.headers.bar` 读取。此外，`request.raw.headers` 能让你访问标准的请求 header。

```js
fastify.post('/:params', options, function (request, reply) {
  console.log(request.body)
  console.log(request.query)
  console.log(request.params)
  console.log(request.headers)
  console.log(request.raw)
  console.log(request.server)
  console.log(request.id)
  console.log(request.ip)
  console.log(request.ips)
  console.log(request.hostname)
  console.log(request.protocol)
  console.log(request.url)
  console.log(request.routerMethod)
  console.log(request.routerPath)
  request.log.info('some info')
})
```
