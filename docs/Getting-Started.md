<h1 align="center">Fastify</h1>

## 起步
Hello！感谢你来到 Fastify 的世界！<br>
这篇文档将向你介绍 Fastify 框架及其特性，也包含了一些示例和指向其他文档的链接。<br>
那，这就开始吧！

<a name="install"></a>
### 安装
```
npm i fastify --save
```
<a name="first-server"></a>
### 第一个服务器
让我们开始编写第一个服务器吧：
```js
// 加载框架并新建实例
const fastify = require('fastify')()

// 声明路由
fastify.get('/'，function (request，reply) {
  reply.send({ hello: 'world' })
})

// 启动服务！
fastify.listen(3000，function (err) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

更喜欢使用 `async/await`？Fastify 对其提供了开箱即用的支持。<br>
*(我们还建议使用 [make-promises-safe](https://github.com/mcollina/make-promises-safe) 来避免文件描述符 (file descriptor) 及内存的泄露)*
```js
const fastify = require('fastify')()

fastify.get('/'，async (request，reply) => {
  return { hello: 'world' }
})

const start = async () => {
  try {
    await fastify.listen(3000)
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()
```

如此简单，棒极了！<br>
可是，一个复杂的应用需要比上例多得多的代码。当你从头开始构建一个应用时，会遇到一些典型的问题，如多个文件的操作、异步引导，以及代码结构的布置。<br>
幸运的是，Fastify 提供了一个易于使用的平台，它能帮助你解决不限于上述的诸多问题。

> ## 注意
> 本文档中的示例，默认情况下只监听本地 `127.0.0.1` 端口。要监听所有有效的 IPv4 端口，需要将代码修改为监听 `0.0.0.0`，如下所示：
>
> ```js
> fastify.listen(3000，'0.0.0.0'，function (err) {
>   if (err) {
>     fastify.log.error(err)
>     process.exit(1)
>   }
> })
> ```
>
> 类似地，`::1` 表示只允许本地的 IPv6 连接。而 `::` 表示允许所有 IPv6 地址的接入，当操作系统支持时，所有的 IPv4 地址也会被允许。
>
> 当使用 Docker 或其他容器部署时，这会是最简单的暴露应用的方式。

<a name="first-plugin"></a>
### 第一个插件
就如同在 JavaScript 中一切皆为对象，在 Fastify 中，一切都是插件 (plugin)。<br>
在深入之前，先来看看插件系统是如何工作的吧！<br>
让我们新建一个基本的服务器，但这回我们把路由 (route) 的声明从入口文件转移到一个外部文件。(参阅[路由声明](https://github.com/fastify/fastify/blob/master/docs/Routes.md)).
```js
const fastify = require('fastify')()

fastify.register(require('./our-first-route'))

fastify.listen(3000，function (err) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

```js
// our-first-route.js

async function routes (fastify，options) {
  fastify.get('/'，async (request，reply) => {
    return { hello: 'world' }
  })
}

module.exports = routes
```
这个例子调用了 `register` API。这一 API 是 Fastify 框架的核心，也是注册路由、插件等的唯一方法。

在本文的开头，我们提到 Fastify 提供了帮助应用异步引导的基础功能。为什么这一功能十分重要呢？
考虑一下，当存在数据库操作时，数据库连接显然要在服务器接受外部请求之前完成。该如何解决这一问题呢？<br>
典型的解决方案是使用复杂的回调函数或 Promise，但如此会造成框架的 API、其他库以及应用程序的代码混杂在一起。<br>
Fastify 则不走寻常路，它从本质上用最轻松的方式解决这一问题！

让我们重写上述示例，加入一个数据库连接。<br>
*(在这里我们用简单的例子来说明，对于健壮的方案请考虑使用 [`fastify-mongo`](https://github.com/fastify/fastify-mongodb) 或 Fastify [生态](https://github.com/fastify/fastify/blob/master/docs/Ecosystem.md)中的其他插件)*

**server.js**
```js
const fastify = require('fastify')()

fastify.register(require('./our-db-connector')，{
  url: 'mongodb://localhost:27017/'
})
fastify.register(require('./our-first-route'))

fastify.listen(3000，function (err) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
})
```

**our-db-connector.js**
```js
const fastifyPlugin = require('fastify-plugin')
const MongoClient = require('mongodb').MongoClient

async function dbConnector (fastify，options) {
  const url = options.url
  delete options.url

  const db = await MongoClient.connect(url，options)
  fastify.decorate('mongo'，db)
}
// 用 fastify-plugin 包装插件，以使插件中声明的装饰器、钩子函数及中间件暴露在根作用域里。
module.exports = fastifyPlugin(dbConnector)
```

**our-first-route.js**
```js
async function routes (fastify，options) {
  const database = fastify.mongo.db('db')
  const collection = database.collection('test')

  fastify.get('/'，async (request，reply) => {
    return { hello: 'world' }
  })

  fastify.get('/search/:id'，async (request，reply) => {
    const result = await collection.findOne({ id: request.params.id })
    if (result.value === null) {
      throw new Error('Invalid value')
    }
    return result.value
  })
}

module.exports = routes
```

哇，真是快啊！<br>
介绍了一些新概念后，让我们回顾一下迄今为止都做了些什么吧。<br>
如你所见，我们可以使用 `register` 来注册数据库连接器或者路由。
这是 Fastify 最棒的特性之一了。它使得插件按声明的顺序来加载，唯有当前插件加载完毕后，才会加载下一个插件。如此，我们便可以在第一个插件中注册数据库连接器，并在第二个插件中使用它。*(参见 [这里](https://github.com/fastify/fastify/blob/master/docs/Plugins.md#handle-the-scope) 了解如何处理插件的作用域)*。
当调用函数 `fastify.listen()`、`fastify.inject()` 或 `fastify.ready()` 时，插件便开始加载了。

我们还用到了 `decorate` API。现在来看看这一 API 是什么，以及它是如何运作的吧。
考虑下需要在应用的不同部分使用相同的代码或库的场景。一种解决方案便是按需引入这些代码或库。哈，这固然可行，但却因为重复的代码和麻烦的重构让人苦恼。<br>
为了解决上述问题，Fastify 提供了 `decorate` API。它允许你在 Fastify 的命名空间下添加自定义对象，如此一来，你就可以在所有地方直接使用这些对象了。

更深入的内容，例如插件如何运作、如何新建，以及使用 Fastify 全部的 API 去处理复杂的异步引导的细节，请看[插件指南](https://github.com/fastify/fastify/blob/master/docs/Plugins-Guide.md)。

<a name="plugin-loading-order"></a>
### 插件加载顺序
为了保证应用的行为一致且可预测，我们强烈建议你采用以下的顺序来组织代码：
```
└── 来自 Fastify 生态的插件
└── 你自己的插件
└── 装饰器
└── 钩子函数和中间件
└── 你的服务应用
```
这确保了你总能访问当前作用域下声明的所有属性。<br/>
如前文所述，Fastify 提供了一个可靠的封装模型，它能帮助你的应用成为单一且独立的服务。假如你要为某些路由单独地注册插件，只需复写上述的结构就足够了。
```
└── 来自 Fastify 生态的插件
└── 你自己的插件
└── 装饰器
└── 钩子函数和中间件
└── 你的服务应用
    │
    └──  服务 A
    │     └── 来自 Fastify 生态的插件
    │     └── 你自己的插件
    │     └── 装饰器
    │     └── 钩子函数和中间件
    │     └── 你的服务应用
    │
    └──  服务 B
    │     └── 来自 Fastify 生态的插件
    │     └── 你自己的插件
    │     └── 装饰器
    │     └── 钩子函数和中间件
    │     └── 你的服务应用
```

<a name="validate-data"></a>
### 验证数据
数据的验证在我们的框架中是极为重要的一环，也是核心的概念。<br>
Fastify 使用 [JSON Schema](http://json-schema.org/) 验证来访的请求。
让我们来看一个验证路由的例子：
```js
const opts = {
  schema: {
    body: {
      type: 'object'，
      properties: {
        someKey: { type: 'string' }，
        someOtherKey: { type: 'number' }
      }
    }
  }
}

fastify.post('/'，opts，async (request，reply) => {
  return { hello: 'world' }
})
```
这个例子展示了如何向路由传递配置选项。选项中包含了一个名为 `schema` 的对象，它便是我们验证路由所用的模式 (schema)。借由 schema，我们可以验证 `body`、`querystring`、`params` 以及 `header`。<br>
请参阅[验证与序列化](https://github.com/fastify/fastify/blob/master/docs/Validation-and-Serialization.md)获取更多信息。

<a name="serialize-data"></a>
### 序列化数据
Fastify 对 JSON 提供了优异的支持，极大地优化了解析 JSON 与序列化 JSON 输出的过程。<br>
在 schema 的选项中设置 `response` 的值，能够加快 JSON 的序列化 (没错，这很慢！)，就像这样：
```js
const opts = {
  schema: {
    response: {
      200: {
        type: 'object'，
        properties: {
          hello: { type: 'string' }
        }
      }
    }
  }
}

fastify.get('/'，opts，async (request，reply) => {
  return { hello: 'world' }
})
```
就这么简单，序列化的过程达到了原先 2 倍甚至 3 倍的速度。这么做同时也保护了敏感数据不被泄露，因为 Fastify 仅对 schema 里出现的数据进行序列化。
请参阅 [验证与序列化](https://github.com/fastify/fastify/blob/master/docs/Validation-and-Serialization.md)获取更多信息。

<a name="extend-server"></a>
### 扩展服务器
Fastify 生来十分精简，也具有高可扩展性。我们相信，一个小巧的框架足以实现一个优秀的应用。<br>
换句话说，Fastify 并非一个面面俱到的框架，它依赖于自己惊人的[生态系统](https://github.com/fastify/fastify/blob/master/docs/Ecosystem.md)！

<a name="test-server"></a>
### 测试服务器
Fastify 并没有提供测试框架，但是我们推荐你在测试中使用 Fastify 的特性及结构。<br>
更多内容请看[测试](https://github.com/fastify/fastify/blob/master/docs/Testing.md)！

<a name="cli"></a>
### 从命令行启动服务器
感谢 [fastify-cli](https://github.com/fastify/fastify-cli)，它让 Fastify 集成到了命令行之中。

首先，你得安装 `fastify-cli`:

```
npm i fastify-cli
```

你还可以加入 `-g` 选项来全局安装它。

接下来，在 `package.json` 中添加如下行：
```json
{
  "scripts": {
    "start": "fastify start server.js"
  }
}
```

然后，创建你的服务器文件：
```js
// server.js
'use strict'

module.exports = async function (fastify，opts) {
  fastify.get('/'，async (request，reply) => {
    return { hello: 'world' }
  })
}
```

最后，启动你的服务器：
```bash
npm start
```

<a name="slides"></a>
### 幻灯片与视频 (英文资源)
- 幻灯片
  - [为你的 HTTP 服务器提速](https://mcollina.github.io/take-your-http-server-to-ludicrous-speed) by [@mcollina](https://github.com/mcollina)
  - [如果我告诉你 HTTP 可以更快](https://delvedor.github.io/What-if-I-told-you-that-HTTP-can-be-fast) by [@delvedor](https://github.com/delvedor)

- 视频
  - [为你的 HTTP 服务器提速](https://www.youtube.com/watch?v=5z46jJZNe8k) by [@mcollina](https://github.com/mcollina)
  - [如果我告诉你 HTTP 可以更快](https://www.webexpo.net/prague2017/talk/what-if-i-told-you-that-http-can-be-fast/) by [@delvedor](https://github.com/delvedor)
