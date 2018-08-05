<h1 align="center">Fastify</h1>

<a id="typescript"></a>
## TypeScript
尽管 Fastify 自带了 typings 声明文件，你仍然需要根据所使用的 Node.js 的版本来安装 `@types/node`。

## Types 支持
我们关注 TypeScript 社区。然而现状是 Fastify 由纯 JavaScript 写就，当前也没有一名核心团队的成员是 TypeScript 的使用者，仅有一名贡献者是。
我们尽自己最大的努力来保证 typings 文件与最新的 API 同步，但并不能完全避免不同步的情况发生。<br/>
幸运的是这是个开源项目，你可以参与修复。我们十分欢迎你的贡献，并会尽快发布补丁。请看 [贡献](#contributing) 指南吧！

插件有可能包含 typings，也可能没有。具体信息请参阅 [插件类型](#plugin-types)。

## 示例
以下 TypeScript 的程序示例和 JavaScript 版本的示例紧密相似：

```ts
import * as fastify from 'fastify'
import { Server, IncomingMessage, ServerResponse } from 'http'

// 创建一个 http 服务器，将 http 对应版本所使用的 typings 传递过去。
// 这么做我们便能获知路由底层 http 对象的结构。
// 如果使用 http2，你应该传递 <http2.Http2Server, http2.Http2ServerRequest, http2.Http2ServerResponse>
const server: fastify.FastifyInstance<Server, IncomingMessage, ServerResponse> = fastify({})

const opts: fastify.RouteShorthandOptions = {
  schema: {
    response: {
      200: {
        type: 'object',
        properties: {
          pong: {
            type: 'string'
          }
        }
      }
    }
  }
}

server.get('/ping', opts, (request, reply) => {
  console.log(reply.res) // 带有正确 typings 的 http.ServerResponse！
  reply.code(200).send({ pong: 'it worked!' })
})
```

<a id="http-prototypes"></a>
## HTTP 原型
默认情况下，Fastify 会根据你给的配置决定使用 http 的哪个版本。出于某种原因需要覆盖的话，你可以这么做：

```ts
interface CustomIncomingMessage extends http.IncomingMessage {
  getClientDeviceType: () => string
}

// 将覆盖的 http 原型传给 Fastify
const server: fastify.FastifyInstance<http.Server, CustomIncomingMessage, http.ServerResponse> = fastify()

server.get('/ping', (request, reply) => {
  // 使用自定义的 http 原型方法
  const clientDeviceType = request.raw.getClientDeviceType()

  reply.send({ clientDeviceType: `you called this endpoint from a ${clientDeviceType}` })
})
```

在这个例子中，我们传递了一个经过修改的 `http.IncomingMessage` 接口，该接口在程序的其他地方得到了扩展。


<a id="contributing"></a>
## 贡献
和 TypeScript 相关的改动可以被归入下列类别：

* Core - Fastify 的 typings 文件
* Plugins - Fastify 插件

记得要先阅读 `CONTRIBUTING.md` 文件，确保行事顺利！

<a id="core-types"></a>
### 核心类型
当更新核心类型时，你应当向本仓库发一个 PR。请确保：

1. 将改动反映在 `examples/typescript-server.ts` 文件中 (当需要时)
2. 更新 `test/types/index.ts` 来验证改动是否成功

<a id="plugin-types"></a>
### 插件类型

插件的 typings 文件存放在 DefinitelyTyped 仓库中，这意味着使用插件时你还需要像下述一样安装类型文件：

```
npm install fastify-url-data @types/fastify-url-data
```

这之后便顺利了。一些类型可能还不可用，因此尽管去贡献吧。

<a id="authoring-plugin-types"></a>
### 编写插件类型
扩展了 `FastifyRequest` 与 `FastifyReply` 对象的许多插件，可以通过如下方式获取。

该代码展示了如何给应用添加 `fastify-url-data` 的类型。

```ts
// 文件名: custom-types.d.ts

// 核心的 typings 与它的值
import fastify = require('fastify');

// 用于插件 typings 的额外类型
import { UrlObject } from 'url';

// 通过 "fastify-url-data" 插件扩展 FastifyReply
declare module 'fastify' {
  interface FastifyRequest {
    urlData (): UrlObject
  }
}

declare function urlData (): void

declare namespace urlData {}

export = urlData;
```

现在你可以如下使用 `fastify-url-data`：

```ts
import * as fastify from 'fastify'
import * as urlData from 'fastify-url-data'

/// <reference types="./custom-types.d.ts"/>

const server = fastify();

server.register(urlData)

server.get('/data', (request, reply) => {
  console.log(request.urlData().auth)
  console.log(request.urlData().host)
  console.log(request.urlData().port)
  console.log(request.urlData().query)

  reply.send({msg: 'ok'})
})

server.listen(3030)
```

请记住，如果你为一个插件创建了 typings，你应该将其发布到 DefinitelyTyped 仓库中！