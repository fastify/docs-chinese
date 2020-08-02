<h1 align="center">Fastify</h1>

<a id="errors"></a>
## 错误

<a name="error-handling"></a>
### Node.js 错误处理

#### 未捕获的错误
在 Node.js 里，未捕获的错误容易引起内存泄漏、文件描述符泄漏等生产环境主要的问题。[Domain 模块](https://nodejs.org/en/docs/guides/domain-postmortem/)被设计用来解决这一问题，然而效果不佳。

由于不可能合理地处理所有未捕获的错误，目前最好的处理方案就是[使程序崩溃](https://nodejs.org/api/process.html#process_warning_using_uncaughtexception_correctly)。

#### 在 Promise 里捕获错误
未处理的 promise rejection (即未被 `.catch()` 处理) 在 Node.js 中也可能引起内存或文件描述符泄漏。`unhandledRejection` 不被推荐，未处理的 rejection 也不会抛出，因此还是可能会泄露。你应当使用如 [`make-promises-safe`](https://github.com/mcollina/make-promises-safe) 的模块来确保未处理的 rejection _总能_ 被抛出。

假如你使用 promise，你应当同时给它们加上 `.catch()`。

### Fastify 的错误
Fastify 遵循不全则无的原则，旨在精而优。因此，确保正确处理错误是开发者需要考虑的问题。

#### 输入数据的错误
由于大部分的错误源于预期外的输入，我们建议为输入的数据指明 [JSON.schema 验证](Validation-and-Serialization.md)。

#### 在 Fastify 中捕捉未捕获的错误
在不影响性能的前提下，Fastify 尽可能多地捕捉未捕获的错误。这些错误包括：

1. 同步路由中的错误。如 `app.get('/', () => { throw new Error('kaboom') })`
2. `async` 路由中的错误。如 `app.get('/', async () => { throw new Error('kaboom') })`

上述错误都会被安全地捕捉，并移交给 Fastify 默认的错误处理函数，发送一个通用的 `500 Internal Server Error` 响应。

要自定义该行为，请见 [`setErrorHandler`](Server.md#seterrorhandler)。

### Fastify 生命周期钩子的错误，及自定义错误控制函数

在[钩子](Hooks/#manage-errors-from-a-hook)的文档中提到：
> 假如在钩子执行过程中发生错误，只需把它传递给 `done()`，Fastify 便会自动地关闭请求，并向用户发送合适的错误代码。

如果通过 `setErrorHandler` 自定义了一个错误函数，那么错误会被引导到那里，否则被引导到 Fastify 默认的错误函数中去。

自定义错误函数应该考虑以下几点：

- 你可以调用 `reply.send(data)`，正如在[常规路由](Reply/#senddata)中那样
  - object 会被序列化，并触发 `preSerialization` 钩子 (假如有定义的话)
  - string、buffer 及 stream 会被直接发送至客户端 (不会序列化)，并附带上合适的 header。

- 在错误函数里你可以抛出新的错误
  - 错误 (新的错误，或被重新抛出的错误参数) 会触发 `onError` 钩子，并被发送给用户
  - 在同一个钩子内，一个错误不会被触发两次。Fastify 会内在地监控错误的触发，以此避免在回复阶段无限循环地抛错 (在路由函数执行后)。

<a name="fastify-error-codes"></a>
### Fastify 错误代码

<a name="FST_ERR_BAD_URL"></a>
#### FST_ERR_BAD_URL

无效的 url。

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
#### FST_ERR_REP_ALREADY_SENT

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

<a name="FST_ERR_SCH_VALIDATION_BUILD"></a>
#### FST_ERR_SCH_VALIDATION_BUILD

用于校验路由的 JSON schema 不合法。

<a name="FST_ERR_SCH_SERIALIZATION_BUILD"></a>
#### FST_ERR_SCH_SERIALIZATION_BUILD

用于序列化响应的 JSON schema 不合法。

<a name="FST_ERR_PROMISE_NOT_FULLFILLED"></a>
#### FST_ERR_PROMISE_NOT_FULLFILLED

状态码不为 204 时，Promise 的 payload 不能为 'undefined'。

<a name="FST_ERR_SEND_UNDEFINED_ERR"></a>
#### FST_ERR_SEND_UNDEFINED_ERR

发生了未定义的错误。