<h1 align="center">Fastify</h1>

## Request
处理函数的第一个参数是 `Request`.<br>
Request 是 Fastify 的核心对象，包含了以下字段:
- `query` - 解析后的 querystring
- `body` - 消息主体
- `params` - URL 参数
- `headers` - headers
- `raw` - Node 原生的 HTTP 请求
- `req` *(不推荐，请使用 `.raw`)* - Node 原生的 HTTP 请求
- `id` - 请求 id
- `log` - 请求的日志实例
- `ip` - 请求方的 ip 地址
- `ips` - x-forwarder-for header 中保存的请求源 ip 数组 (仅当 [`trustProxy`](Server.md#factory-trust-proxy) 开启时有效)
- `hostname` - 请求方的主机名 (当 [`trustProxy`](Server.md#factory-trust-proxy) 启用时，从 `X-Forwarded-Host` header 中获取)
- `protocol` - 请求协议 (`https` 或 `http`)
- `method` - 请求方法
- `url` - 请求路径
- `routerMethod` - 处理请求的路由函数
- `routerPath` - 处理请求的路由的匹配模式
- `is404` - 当请求被 404 处理时为 true，反之为 false
- `connection` - 不推荐，请使用 `socket`。请求的底层连接
- `socket` - 请求的底层连接

```js
fastify.post('/:params', options, function (request, reply) {
  console.log(request.body)
  console.log(request.query)
  console.log(request.params)
  console.log(request.headers)
  console.log(request.raw)
  console.log(request.id)
  console.log(request.ip)
  console.log(request.ips)
  console.log(request.hostname)
  console.log(request.protocol)
  request.log.info('some info')
})
```
