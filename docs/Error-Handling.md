<h1 align="center">Fastify</h1>

## 错误处理

未捕获的错误容易引起内存泄漏、文件描述符泄漏等生产环境主要的问题。Node 的 [Domain 模块](https://nodejs.org/en/docs/guides/domain-postmortem/)被设计用来解决这一问题，然而效果不佳。事实上，以合理的方式来处理所有未捕获的错误是不可能的，目前，最好的方法就是[使程序崩溃](https://nodejs.org/api/process.html#process_warning_using_uncaughtexception_correctly)。当使用 promise 时，请注意[正确地](https://github.com/mcollina/make-promises-safe)[处理了](https://nodejs.org/dist/latest-v8.x/docs/api/deprecations.html#deprecations_dep0018_unhandled_promise_rejections)错误。

Fastify 遵循不全则无的原则，旨在精而优。因此，确保正确处理错误成了开发者要考虑的问题。由于大部分的错误源于预期外的输入，我们建议为输入的数据指明 [JSON.schema 验证](https://github.com/fastify/fastify/blob/master/docs/Validation-and-Serialization.md)。

要注意的是，虽然 Fastify 没帮你捕获错误，但是当路由被声明为 `async` 模式时，错误会被 promise 安全地捕获，并通过 Fastify 默认的错误处理器以一般的 `Internal Server Error` 响应发送给客户端。要自定义这一行为，请看 [setErrorHandler](https://github.com/fastify/fastify/blob/master/docs/Server.md#seterrorhandler)。