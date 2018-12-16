<h1 align="center">Fastify</h1>

## 中间件

Fastify 提供了一个开箱即用、兼容 [Express](https://expressjs.com/) 与 [Restify](http://restify.com/) 的中间件的异步[中间件引擎](https://github.com/fastify/middie)。

*假如你需要可视化反馈来明白中间件何时执行，请看[生命周期](https://github.com/fastify/docs-chinese/blob/master/docs/Lifecycle.md)一文。*

Fastify 的中间件不支持 `middleware(err, req, res, next)` 这一完整语法，因为错误在 Fastify 内部就被解决了。
此外，Express 和 Restify 添加在 `req` 和 `res` 对象之上的方法，Fastify 也不支持。

为了性能考虑，我们不推荐你使用一个打包了多个更小的中间件的中间件，例如 [*helmet*](https://helmetjs.github.io/)。我们建议你使用单个的模块：

```js
fastify.use(require('cors')())
fastify.use(require('dns-prefetch-control')())
fastify.use(require('frameguard')())
fastify.use(require('hide-powered-by')())
fastify.use(require('hsts')())
fastify.use(require('ienoopen')())
fastify.use(require('x-xss-protection')())
```

或者，在这个 *helmet* 的例子中，你可以使用针对 Fastify 和 helmet 的整合做了优化的 [*fastify-helmet*](https://github.com/fastify/fastify-helmet) [插件](Plugins.md)：

```js
const fastify = require('fastify')()
const helmet = require('fastify-helmet')

fastify.register(helmet)
```

请记住，中间件能被封装。这就意味着你可以通过使用 `register` 来决定中间件该在何处运行，正如[插件指南](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins-Guide.md)一文所述。

Fastify 中间件不会暴露 `send` 等 Fastify 的 [Reply]('./Reply.md' "Reply") 实例上专属的方法。这是因为，虽然 Fastify 使用 [Request](./Request.md "Request") 和 [Reply](./Reply.md "Reply") 对象包裹了 Node 原生的 `req` 和 `res` 实例，但是它们的处理要在中间件阶段之后。因此，在一个中间件里，你必须使用 Node 原生的 `req` 和 `res` 对象。要使用 Fastify 的 [Request](./Request.md "Request") 与 [Reply](./Reply.md "Reply") 实例，你可以通过 `preHandler` 钩子。更多信息，请看[钩子](./Hooks.md "Hooks")。

<a name="restrict-usage"></a>
#### 将中间件限定在特定的路径执行
如果你只想在某些路径下运行一个中间件，只需将路径作为 `use` 的第一个参数传递即可！

*注意，该做法不支持参数路由 (例如：`/user/:id/comments`)，且在多个路径中不能使用通配符。*

```js
const path = require('path')
const serveStatic = require('serve-static')

// 单个路径
fastify.use('/css', serveStatic(path.join(__dirname, '/assets')))

// 通配符路径
fastify.use('/css/*', serveStatic(path.join(__dirname, '/assets')))

// 多个路径
fastify.use(['/css', '/js'], serveStatic(path.join(__dirname, '/assets')))
```

<a name="express-middleware"></a>
#### Express middleware compatibility
#### Express 中间件兼容性
Express 很大程度上修改了 Node 原生的 `Request` 和 `Response` 对象，所以 Fastify 无法确保中间件一定完全兼容。一些 Express 特殊的功能，例如 `res.sendFile()`、`res.send()` 或者 `express.Router()` 的实例将无法在 Fastify 中正常工作。举个例子，[cors](https://github.com/expressjs/cors) 可以正常兼容但是 [passport](https://github.com/jaredhanson/passport) 就不可以。