<h1 align="center">Fastify</h1>

## 服务器方法

<a name="server"></a>
#### 服务器
`fastify.server`：由 [**`Fastify 的工厂函数`**](https://github.com/fastify/docs-chinese/blob/master/docs/Factory.md) 生成的 Node 原生 [server](https://nodejs.org/api/http.html#http_class_http_server) 对象。

<a name="after"></a>
#### after
当前插件及在其中注册的所有插件加载完毕后调用。总在 `fastify.ready` 之前执行。

```js
fastify
  .register((instance. opts, next) => {
    console.log('当前插件')
    next()
  })
  .after(err => {
    console.log('当前插件之后')
  })
  .register((instance. opts, next) => {
    console.log('下一个插件')
    next()
  })
  .ready(err => {
    console.log('万事俱备')
  })
```

<a name="ready"></a>
#### ready
当所有插件的加载都完成时调用。如有错误发生，它会传递一个 `error` 参数。
```js
fastify.ready(err => {
  if (err) throw err
})
```
调用时不加参数，它会返回一个 `Promise` 对象：

```js
fastify.ready().then(() => {
  console.log('successfully booted!')
}, (err) => {
  console.log('an error happened', err)
})
```

<a name="listen"></a>
#### listen
所有的插件加载完毕、`ready` 事件触发后，在指定的端口启动服务器。它的回调函数与 Node 原生方法的回调相同。默认情况下，服务器监听 `127.0.0.1`。将地址设置为 `0.0.0.0` 可监听所有的 IPV4 地址。设置为 `::` 则可监听所有的 IPV6 地址，在某些系统中，这么做亦可同时监听所有 IPV4 地址。监听所有的接口要格外谨慎，因为这种方式存在着固有的[安全风险](https://web.archive.org/web/20170831174611/https://snyk.io/blog/mongodb-hack-and-secure-defaults/)。

```js
fastify.listen(3000, err => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

指定监听的地址：

```js
fastify.listen(3000, '127.0.0.1', err => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

指定积压队列 (backlog queue size) 的大小：

```js
fastify.listen(3000, '127.0.0.1', 511, err => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

没有提供回调函数时，它会返回一个 Promise 对象：

```js
fastify.listen(3000)
  .then(() => console.log('Listening'))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

你还可以在使用 Promise 的同时指定地址：

```js
fastify.listen(3000, '127.0.0.1')
  .then(() => console.log('Listening'))
  .catch(err => {
    console.log('Error starting server:', err)
    process.exit(1)
  })
```

当部署在 Docker 或其它容器上时，明智的做法是监听 `0.0.0.0`。因为默认情况下，这些容器并未将映射的端口暴露在 `127.0.0.1`：

```js
fastify.listen(3000, '0.0.0.0', (err) => {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

<a name="route"></a>
#### route
将路由添加到服务器的方法，支持简写。请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Routes.md)。

<a name="close"></a>
#### close
`fastify.close(callback)`：调用这个函数来关闭服务器实例，并触发 [`'onClose'`](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md#on-close) 钩子。<br>
服务器会向所有新的请求发送 `503` 错误，并销毁它们。

<a name="decorate"></a>
#### decorate*
向 Fastify 实例、响应或请求添加装饰器函数。参阅[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Decorators.md)了解更多。

<a name="register"></a>
#### register
Fastify 允许用户通过插件扩展功能。插件可以是一组路由、装饰器或其他。请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md)。

<a name="use"></a>
#### use
向 Fastify 添加中间件，请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Middlewares.md)。

<a name="addHook"></a>
#### addHook
向 Fastify 添加特定的生命周期钩子函数，请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Hooks.md)。

<a name="base-path"></a>
#### basepath
添加在路由前的完整路径。

示例：

```js
fastify.register(function (instance, opts, next) {
  instance.get('/foo', function (request, reply) {
    // 输出："basePath: /v1"
    request.log.info('basePath: %s', instance.basePath)
    reply.send({basePath: instance.basePath})
  })

  instance.register(function (instance, opts, next) {
    instance.get('/bar', function (request, reply) {
      // 输出："basePath: /v1/v2"
      request.log.info('basePath: %s', instance.basePath)
      reply.send({basePath: instance.basePath})
    })

    next()
  }, { prefix: '/v2' })

  next()
}, { prefix: '/v1' })
```

<a name="log"></a>
#### log
日志的实例，详见[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Logging.md)。

<a name="inject"></a>
#### inject
伪造 http 注入 (作为测试之用) 。请看[更多内容](https://github.com/fastify/docs-chinese/blob/master/docs/Testing.md#inject)。

<a name="add-schema"></a>
#### addSchema
`fastify.addSchema(schemaObj)`，向 Fastify 实例添加可共用的 schema，用于验证数据。你可以通过该 schema 的 id 在应用的任意位置使用它。<br/>
请看[验证和序列化](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md)一文中的[范例](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md#shared-schema)。
<a name="set-schema-compiler"></a>
#### setSchemaCompiler
为所有的路由设置 schema 编译器 (schema compiler)，请看[这里](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md#schema-compiler)了解更多信息。

<a name="set-not-found-handler"></a>
#### setNotFoundHandler

`fastify.setNotFoundHandler(handler(request, reply))`：为 404 状态 (not found) 设置处理器 (handler) 函数。向 `fastify.register()` 传递不同的 [`prefix` 选项](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md#route-prefixing-option)，就可以为不同的插件设置不同的处理器。这些处理器被视为常规的路由处理器，因此它们的请求会经历一个完整的 [Fastify 生命周期](https://github.com/fastify/docs-chinese/blob/master/docs/Lifecycle.md#lifecycle)。

你也可以为 404 处理器注册一个 [beforeHandler](https://www.fastify.io/docs/latest/Hooks/#beforehandler) 钩子。

```js
fastify.setNotFoundHandler({
  beforeHandler: (req, reply, next) => {
    req.body.beforeHandler = true
    next()
  }  
}, function (request, reply) {
    // 设置了 beforeHandler 钩子的默认 not found 处理器
})

fastify.register(function (instance, options, next) {
  instance.setNotFoundHandler(function (request, reply) {
    // '/v1' 开头的 URL 的 not found 处理器，
    // 未设置 beforeHandler 钩子
  })
  next()
}, { prefix: '/v1' })
```

<a name="set-error-handler"></a>
#### setErrorHandler

`fastify.setErrorHandler(handler(error, request, reply))`：设置任意时刻的错误处理器函数。错误处理器是完全封装 (fully encapsulated) 的，因此不同插件的处理器可以不同。支持 *async-await* 语法。

```js
fastify.setErrorHandler(function (error, request, reply) {
  // 发送错误响应
})
```

<a name="print-routes"></a>
#### printRoutes

`fastify.printRoutes()`：打印路由的基数树 (radix tree)，可作调试之用。<br/>
*记得在 `ready` 函数的内部或之后调用它。*

```js
fastify.get('/test', () => {})
fastify.get('/test/hello', () => {})
fastify.get('/hello/world', () => {})

fastify.ready(() => {
  console.log(fastify.printRoutes())
  // └── /
  //   ├── test (GET)
  //   │   └── /hello (GET)
  //   └── hello/world (GET)
})
```
