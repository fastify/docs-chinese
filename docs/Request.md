<h1 align="center">Fastify</h1>

## Request
处理函数的第一个参数是 `Request`.<br>
Request 是 Fastify 的核心对象，包含了一下的字段:
- `query` - 解析后的 querystring
- `body` - 消息主体
- `params` - URL 参数
- `headers` - headers
- `raw` - Node 原生的 HTTP 请求 *(可以用别名 `req`)*
- `id` - 请求 id
- `log` - 请求的日志实例
- `ip` - 请求方的 ip 地址
- `ips` - x-forwarder-for header 中保存的请求源 ip 数组 (仅当 [`trustProxy`](https://github.com/fastify/fastify/blob/master/docs/Server.md#factory-trust-proxy) 开启时有效)
- `hostname` - 请求方的主机名

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
  request.log.info('some info')
})
```
