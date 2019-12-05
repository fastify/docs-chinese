<h1 align="center">Fastify</h1>

<a id="typescript"></a>
## TypeScript
尽管 Fastify 自带了 typings 声明文件，你可能仍然需要根据所使用的 Node.js 版本来安装 `@types/node`。

## Types 支持
我们关注 TypeScript 社区，当前也有一名团队的核心成员正在重做所有的 types。
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

<a id="generic-parameters"></a>
## 一般类型的参数
你不但可以校验 querystring、url 参数、body 以及 header，你还可以覆盖 request 接口中定义的默认类型：

```ts
import * as fastify from 'fastify'

const server = fastify({})

interface Query {
  foo?: number
}

interface Params {
  bar?: string
}

interface Body {
  baz?: string
}

interface Headers {
  a?: string
}

const opts: fastify.RouteShorthandOptions = {
  schema: {
    querystring: {
      type: 'object',
      properties: {
        foo: {
          type: 'number'
        }
      }
    },
    params: {
      type: 'object',
      properties: {
        bar: {
          type: 'string'
        }
      }
    },
    body: {
      type: 'object',
      properties: {
        baz: {
          type: 'string'
        }
      }
    },
    headers: {
      type: 'object',
      properties: {
        a: {
          type: 'string'
        }
      }
    }
  }
}

server.get<Query, Params, Headers, Body>('/ping/:bar', opts, (request, reply) => {
  console.log(request.query) // 这是 Query 类型
  console.log(request.params) // 这是 Params 类型
  console.log(request.body) // 这是 Body 类型
  console.log(request.headers) // 这是 Headers 类型
  reply.code(200).send({ pong: 'it worked!' })
})
```

所有的一般类型都是可选的，因此你可以只传递你使用 schema 校验的类型：

```ts
import * as fastify from 'fastify'

const server = fastify({})

interface Params {
  bar?: string
}

const opts: fastify.RouteShorthandOptions = {
  schema: {
    params: {
      type: 'object',
      properties: {
        bar: {
          type: 'string'
        }
      }
    },
  }
}

server.get<fastify.DefaultQuery, Params, unknown>('/ping/:bar', opts, (request, reply) => {
  console.log(request.query) // 这是 fastify.DefaultQuery 类型
  console.log(request.params) // 这是 Params 类型
  console.log(request.body) // 这是未知的类型
  console.log(request.headers) // 这是 fastify.DefaultHeader 类型，因为 typescript 会使用默认类型
  reply.code(200).send({ pong: 'it worked!' })
})

// 假设你不校验 querystring、body 或者 header，
// 最好将类型设为 `unknown`。但这个选择取决于你。
// 下面的例子展示了这一做法，它可以避免你搬起石头砸自己的脚。
// 换句话说，就是别使用不去校验的类型。
server.get<unknown, Params, unknown, unknown>('/ping/:bar', opts, (request, reply) => {
  console.log(request.query) // 这是未知的类型
  console.log(request.params) // 这是 Params 类型
  console.log(request.body) // 这是未知的类型
  console.log(request.headers) // 这是未知的类型
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

和 fastify 仓库一样，由 GitHub 上的 fastify 组织所维护的插件，应当自带 typings 文件。
目前一些插件还没有 typings 文件，我们很欢迎你参与其中。typings 的例子请看 [fastify-cors](https://github.com/fastify/fastify-cors) 仓库。

第三方插件可能自带 typings 文件，或存放于 DefinitelyTyped 之上。请记住，如果你写了一个插件，也请选择上述两种途径之一来存放 typings 文件！从 DefinitelyTyped 安装 typings 的方法可以在[这里](https://github.com/DefinitelyTyped/DefinitelyTyped#npm)找到。

一些类型可能还不可用，因此尽管去贡献吧。

<a id="authoring-plugin-types"></a>
### 编写插件类型
扩展了 `FastifyRequest`、`FastifyReply` 或 `FastifyInstance` 对象的许多插件，可以通过如下方式获取。

以下代码展示了 `fastify-static` 插件的 typings。

```ts
/// <reference types="node" />

// 导入 fastify typings
import * as fastify from 'fastify';

// 导入必需的 http, http2, https typings
import { Server, IncomingMessage, ServerResponse } from "http";
import { Http2SecureServer, Http2Server, Http2ServerRequest, Http2ServerResponse } from "http2";
import * as https from "https";
type HttpServer = Server | Http2Server | Http2SecureServer | https.Server;
type HttpRequest = IncomingMessage | Http2ServerRequest;
type HttpResponse = ServerResponse | Http2ServerResponse;

// 拓展 fastify typings
declare module "fastify" {
  interface FastifyReply<HttpResponse> {
    sendFile(filename: string): FastifyReply<HttpResponse>;
  }
}

// 使用 fastify.Plugin 声明插件的类型
declare function fastifyStatic(): fastify.Plugin<
  Server,
  IncomingMessage,
  ServerResponse,
  {
    root: string;
    prefix?: string;
    serve?: boolean;
    decorateReply?: boolean;
    schemaHide?: boolean;
    setHeaders?: (...args: any[]) => void;
    redirect?: boolean;
    wildcard?: boolean | string;

    // `send` 的选项
    acceptRanges?: boolean;
    cacheControl?: boolean;
    dotfiles?: boolean;
    etag?: boolean;
    extensions?: string[];
    immutable?: boolean;
    index?: string[];
    lastModified?: boolean;
    maxAge?: string | number;
  }
>;

declare namespace fastifyStatic {
  interface FastifyStaticOptions {}
}

// 导出插件类型
export = fastifyStatic;
```

现在你便可以如此使用插件了：

```ts
import * as Fastify from 'fastify'
import * as fastifyStatic from 'fastify-static'

const app = Fastify()

// 这里的配置项会类型检查
app.register(fastifyStatic, {
  acceptRanges: true,
  cacheControl: true,
  decorateReply: true,
  dotfiles: true,
  etag: true,
  extensions: ['.js'],
  immutable: true,
  index: ['1'],
  lastModified: true,
  maxAge: '',
  prefix: '',
  root: '',
  schemaHide: true,
  serve: true,
  setHeaders: (res, pathName) => {
    res.setHeader('some-header', pathName)
  }
})

app.get('/file', (request, reply) => {
  // 使用上文定义的 FastifyReply.sendFile 方法
  reply.sendFile('some-file-name')
})
```

给我们所有的插件加上 typings 需要社区的努力，因此尽管来贡献吧！