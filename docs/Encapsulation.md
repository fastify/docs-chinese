<h1 align="center">Fastify</h1>

<a id="encapsulation"></a>
## 封装

“封装上下文”是 Fastify 的一个基础特性，负责控制[路由](./Routes.md)能访问的[装饰器](./Decorators.md)、[钩子](./Hooks.md)以及[插件](./Plugins.md)。下图是封装上下文的抽象表现：

![Figure 1](./resources/encapsulation_context.svg)

上图可归纳为以下几块内容：

1. _顶层上下文 (root context)_
2. 三个 _顶层插件 (root plugin)_
3. 两个 _子上下文 (child context)_，每个 _子上下文_ 拥有
    * 两个 _子插件 (child plugin)_
    * 一个 _孙子上下文 (grandchild context)_，其又拥有
        - 三个 _子插件 (child plugin)_

任意 _子上下文_ 或 _孙子上下文_ 都有权访问 _顶层插件_。_孙子上下文_ 有权访问它上级的 _子上下文_ 中注册的 _子插件_，但 _子上下文_ *无权* 访问它下级的 _孙子上下文中_ 注册的 _子插件_。

在 Fastify 中，除了 _顶层上下文_，一切皆为[插件](./Plugins.md)。下文的例子也不例外，所有的“上下文”和“插件”都是包含装饰器、钩子、插件及路由的插件。该例子为有三个路由的 REST API 服务器，第一个路由 (`/one`) 需要鉴权 (使用 [fastify-bearer-auth][bearer])，第二个路由 (`/two`) 无需鉴权，第三个路由 (`/three`) 有权访问第二个路由的上下文：

```js
'use strict'

const fastify = require('fastify')()

fastify.decorateRequest('answer', 42)

fastify.register(async function authenticatedContext (childServer) {
  childServer.register(require('fastify-bearer-auth'), { keys: ['abc123'] })

  childServer.route({
    path: '/one',
    method: 'GET',
    handler (request, response) {
      response.send({
        answer: request.answer,
        foo: request.foo,
        bar: request.bar
      })
    }
  })
})

fastify.register(async function publicContext (childServer) {
  childServer.decorateRequest('foo', 'foo')

  childServer.route({
    path: '/two',
    method: 'GET',
    handler (request, response) {
      response.send({
        answer: request.answer,
        foo: request.foo,
        bar: request.bar
      })
    }
  })

  childServer.register(async function grandchildContext (grandchildServer) {
    grandchildServer.decorateRequest('bar', 'bar')

    grandchildServer.route({
      path: '/three',
      method: 'GET',
      handler (request, response) {
        response.send({
          answer: request.answer,
          foo: request.foo,
          bar: request.bar
        })
      }
    })
  })
})

fastify.listen(8000)
```

上面的例子展示了所有封装相关的概念：

1. 每个 _子上下文_ (`authenticatedContext`、`publicContext` 及 `grandchildContext`) 都有权访问在 _顶层上下文_ 中定义的 `answer` 请求装饰器。
2. 只有 `authenticatedContext` 能访问 `fastify-bearer-auth` 插件。
3. `publicContext` 和 `grandchildContext` 都能访问 `foo` 请求装饰器。
4. 只有 `grandchildContext` 能访问 `bar` 请求装饰器。

启动服务来验证这些概念吧：

```sh
# curl -H 'authorization: Bearer abc123' http://127.0.0.1:8000/one
{"answer":42}
# curl http://127.0.0.1:8000/two
{"answer":42,"foo":"foo"}
# curl http://127.0.0.1:8000/three
{"answer":42,"foo":"foo","bar":"bar"}
```

[bearer]: https://github.com/fastify/fastify-bearer-auth

<a id="shared-context"></a>
## 在上下文间共享

请注意，在上文例子中，每个上下文都 _仅_ 从父级上下文进行继承，而父级上下文无权访问后代上下文中定义的实体。在某些情况下，我们并不想要这一默认行为。使用 [fastify-plugin][fastify-plugin] ，便能允许父级上下文访问到后代上下文中定义的实体。

假设上例的 `publicContext` 需要获取 `grandchildContext` 中定义的 `bar` 装饰器，我们可以重写代码如下：

```js
'use strict'

const fastify = require('fastify')()
const fastifyPlugin = require('fastify-plugin')

fastify.decorateRequest('answer', 42)

// 为了代码清晰，这里省略了 `authenticatedContext`

fastify.register(async function publicContext (childServer) {
  childServer.decorateRequest('foo', 'foo')

  childServer.route({
    path: '/two',
    method: 'GET',
    handler (request, response) {
      response.send({
        answer: request.answer,
        foo: request.foo,
        bar: request.bar
      })
    }
  })

  childServer.register(fastifyPlugin(grandchildContext))

  async function grandchildContext (grandchildServer) {
    grandchildServer.decorateRequest('bar', 'bar')

    grandchildServer.route({
      path: '/three',
      method: 'GET',
      handler (request, response) {
        response.send({
          answer: request.answer,
          foo: request.foo,
          bar: request.bar
        })
      }
    })
  }
})

fastify.listen(8000)
```

重启服务，访问 `/two` 和 `/three` 路由：

```sh
# curl http://127.0.0.1:8000/two
{"answer":42,"foo":"foo","bar":"bar"}
# curl http://127.0.0.1:8000/three
{"answer":42,"foo":"foo","bar":"bar"}
```

[fastify-plugin]: https://github.com/fastify/fastify-plugin