<h1 align="center">Fastify</h1>

## Request
第一个句柄方法的参数是 `Request`.<br>
Request 是 Fastify 的核心对象，包含了一下的字段:
- `query` - 解析后的 querystring
- `body` - 消息主体
- `params` - URL 参数
- `headers` - headers
- `raw` - Node 原生的 HTTP 请求 *(可以用别名 `req`)*
- `id` - 请求 id
- `log` - 请求的日志实例

```js
fastify.post('/:params', options, function (request, reply) {
  console.log(request.body)
  console.log(request.query)
  console.log(request.params)
  console.log(request.headers)
  console.log(request.raw)
  console.log(request.id)
  request.log.info('some info')
})
```
