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
       404 ◀─┴─▶ onRequest Hook
                  │
        4**/5** ◀─┴─▶ preParsing Hook
                        │
              4**/5** ◀─┴─▶ Parsing
                             │
                   4**/5** ◀─┴─▶ preValidation Hook
                                  │
                            415 ◀─┴─▶ Validation
                                        │
                                  400 ◀─┴─▶ preHandler Hook
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