<h1 align="center">Fastify</h1>

## 错误处理

未捕获的错误容易引起内存泄漏、文件描述符泄漏等生产环境主要的问题。Node 的 [Domain 模块](https://nodejs.org/en/docs/guides/domain-postmortem/)被设计用来解决这一问题，然而效果不佳。事实上，以合理的方式来处理所有未捕获的错误是不可能的，目前，最好的方法就是[使程序崩溃](https://nodejs.org/api/process.html#process_warning_using_uncaughtexception_correctly)。当使用 promise 时，请注意[正确地](https://github.com/mcollina/make-promises-safe)[处理了](https://nodejs.org/dist/latest-v8.x/docs/api/deprecations.html#deprecations_dep0018_unhandled_promise_rejections)错误。

Fastify 遵循不全则无的原则，旨在精而优。因此，确保正确处理错误成了开发者要考虑的问题。由于大部分的错误源于预期外的输入，我们建议为输入的数据指明 [JSON.schema 验证](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md)。

要注意的是，在基于回调的路由中，Fastify 不会帮你捕获错误。因此，任何未捕获的错误都可能造成崩溃。
但当路由被声明为 `async` 模式时，错误会被 promise 安全地捕获，并通过 Fastify 默认的错误处理器以一般的 `Internal Server Error` 响应发送给客户端。要自定义这一行为，请看 [setErrorHandler](https://github.com/fastify/docs-chinese/blob/master/docs/Server.md#seterrorhandler)。

<a name="fastify-error-codes"></a>
### Fastify 错误代码

<a name="FST_ERR_CTP_ALREADY_PRESENT"></a>
#### FST_ERR_CTP_ALREADY_PRESENT

该 content type 的解析器已经被注册。

<a name="FST_ERR_CTP_INVALID_TYPE"></a>
#### FST_ERR_CTP_INVALID_TYPE

`Content-Type` 应为一个字符串。

<a name="FST_ERR_CTP_EMPTY_TYPE"></a>
#### FST_ERR_CTP_EMPTY_TYPE

content type 不能是一个空字符串。

<a name="FST_ERR_CTP_INVALID_HANDLER"></a>
#### FST_ERR_CTP_INVALID_HANDLER

该 content type 接收的处理函数无效。

<a name="FST_ERR_CTP_INVALID_PARSE_TYPE"></a>
#### FST_ERR_CTP_INVALID_PARSE_TYPE

提供的待解析类型不支持。只支持 `string` 和 `buffer`。

<a name="FST_ERR_CTP_BODY_TOO_LARGE"></a>
#### FST_ERR_CTP_BODY_TOO_LARGE

请求 body 大小超过限制。

<a name="FST_ERR_CTP_INVALID_MEDIA_TYPE"></a>
#### FST_ERR_CTP_INVALID_MEDIA_TYPE

收到的 media type 不支持 (例如，不存在合适的 `Content-Type` 解析器)。

<a name="FST_ERR_CTP_INVALID_CONTENT_LENGTH"></a>
#### FST_ERR_CTP_INVALID_CONTENT_LENGTH

请求 body 大小与 Content-Length 不一致。

<a name="FST_ERR_DEC_ALREADY_PRESENT"></a>
#### FST_ERR_DEC_ALREADY_PRESENT

已存在同名的装饰器。

<a name="FST_ERR_DEC_MISSING_DEPENDENCY"></a>
#### FST_ERR_DEC_MISSING_DEPENDENCY

缺失依赖导致装饰器无法注册。

<a name="FST_ERR_HOOK_INVALID_TYPE"></a>
#### FST_ERR_HOOK_INVALID_TYPE

钩子名称必须为字符串。

<a name="FST_ERR_HOOK_INVALID_HANDLER"></a>
#### FST_ERR_HOOK_INVALID_HANDLER

钩子的回调必须为函数。

<a name="FST_ERR_LOG_INVALID_DESTINATION"></a>
#### FST_ERR_LOG_INVALID_DESTINATION

日志工具目标地址无效。仅接受 `'stream'` 或 `'file'` 作为目标地址。

<a id="FST_ERR_REP_ALREADY_SENT"></a>
### FST_ERR_REP_ALREADY_SENT

响应已发送。

<a id="FST_ERR_SEND_INSIDE_ONERR"></a>
#### FST_ERR_SEND_INSIDE_ONERR

不能在 `onError` 钩子中调用 `send`。

<a name="FST_ERR_REP_INVALID_PAYLOAD_TYPE"></a>
#### FST_ERR_REP_INVALID_PAYLOAD_TYPE

响应 payload 类型无效。只允许 `string` 或 `Buffer`。

<a name="FST_ERR_SCH_MISSING_ID"></a>
#### FST_ERR_SCH_MISSING_ID

提供的 schema 没有 `$id` 属性。

<a name="FST_ERR_SCH_ALREADY_PRESENT"></a>
#### FST_ERR_SCH_ALREADY_PRESENT

同 `$id` 的 schema 已经存在。

<a name="FST_ERR_SCH_NOT_PRESENT"></a>
#### FST_ERR_SCH_NOT_PRESENT

不存在 `$id` 为提供的值的 schema。

<a name="FST_ERR_SCH_BUILD"></a>
#### FST_ERR_SCH_BUILD

某个路由的 JSON schema 不合法。

<a name="FST_ERR_PROMISE_NOT_FULLFILLED"></a>
#### FST_ERR_PROMISE_NOT_FULLFILLED

状态码不为 204 时，Promise 的 payload 不能为 'undefined'。

<a name="FST_ERR_SEND_UNDEFINED_ERR"></a>
#### FST_ERR_SEND_UNDEFINED_ERR

发生了未定义的错误。