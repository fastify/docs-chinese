<h1 align="center">Fastify</h1>

## 生命周期
下图展示了 Fastify 的内部生命周期。<br>
每个节点右边的分支为生命周期的下一阶段，左边的则是上一个生命周期抛出错误时产生的错误码 *(请注意 Fastify 会自动处理所有的错误)*。

```
Incoming Request
  │
  └─▶ Routing
        │
        └─▶ Instance Logger
             │
   4**/5** ◀─┴─▶ onRequest Hook
                  │
        4**/5** ◀─┴─▶ preParsing Hook
                        │
              4**/5** ◀─┴─▶ Parsing
                             │
                   4**/5** ◀─┴─▶ preValidation Hook
                                  │
                            415 ◀─┴─▶ Validation
                                        │
                              4**/5** ◀─┴─▶ preHandler Hook
                                              │
                                    4**/5** ◀─┴─▶ User Handler
                                                    │
                                                    └─▶ Reply
                                                          │
                                                4**/5** ◀─┴─▶ preSerialization Hook
                                                                │
                                                                └─▶ onSend Hook
                                                                      │
                                                            4**/5** ◀─┴─▶ Outgoing Response
                                                                            │
                                                                            └─▶ onResponse Hook
```

在`用户编写的处理函数`执行前或执行时，你可以调用 `reply.hijack()` 以使得 Fastify：
- 终止运行所有钩子及用户的处理函数
- 不再自动发送响应

特别注意 (*)：假如使用了 `reply.raw` 来发送响应，则 `onResponse` 依旧会执行。

## 响应生命周期

不管用户如何处理请求，结果无非以下几种：

- 异步函数中返回 payload
- 异步函数中抛出 `Error`
- 同步函数中发送 payload
- 同步函数中发送 `Error` 实例

当响应被劫持时 (即调用了 `reply.hijack()`) 会跳过之后的步骤，否则，响应被提交后的数据流向如下：

```
                        ★ schema validation Error
                                    │
                                    └─▶ schemaErrorFormatter
                                               │
                          reply sent ◀── JSON ─┴─ Error instance
                                                      │
                                                      │         ★ throw an Error
                     ★ send or return                 │                 │
                            │                         │                 │
                            │                         ▼                 │
       reply sent ◀── JSON ─┴─ Error instance ──▶ setErrorHandler ◀─────┘
                                                      │
                                 reply sent ◀── JSON ─┴─ Error instance ──▶ onError Hook
                                                                                │
                                                                                └─▶ reply sent
```

注：`reply sent` 意味着 JSON payload 将会如下被序列化：

- 通过[响应序列化方法](Server.md#setreplyserializer) (如有设置)
- 或当为返回的 HTTP 状态码设置了 JSON schema 时，通过[序列化函数生成器](Server.md#setserializercompiler)
- 或通过默认的 `JSON.stringify` 函数