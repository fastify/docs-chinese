<h1 align="center">Fastify</h1>

## 中间件

从版本 3 开始，Fastify 便不再内建地支持中间件了，你需要通过插件 [`fastify-express`](https://github.com/fastify/fastify-express) 和 [`middie`](https://github.com/fastify/middie) 来使用它们。

如果你想使用 express 风格的中间件，可以借助上述两个工具之一的 `fastify-express`：

```js
await fastify.register(require('fastify-express'))
fastify.use(require('cors')())
fastify.use(require('dns-prefetch-control')())
fastify.use(require('frameguard')())
fastify.use(require('hsts')())
fastify.use(require('ienoopen')())
fastify.use(require('x-xss-protection')())
```

或者通过 [`middie`](https://github.com/fastify/middie)，它提供了对简单的 express 风格的中间件的支持，但性能更佳：

```js
await fastify.register(require('middie'))
fastify.use(require('cors')())
```

### 替代

Fastify 提供了最常用中间件的替代品，例如：[`fastify-helmet`](https://github.com/fastify/fastify-helmet) 之于 [`helmet`](https://github.com/helmetjs/helmet)，[`fastify-cors`](https://github.com/fastify/fastify-cors) 之于 [`cors`](https://github.com/expressjs/cors)，以及 [`fastify-static`](https://github.com/fastify/fastify-static) 之于 [`serve-static`](https://github.com/expressjs/serve-static)。