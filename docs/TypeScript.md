<h1 align="center">Fastify</h1>

## TypeScript

Fastify 是用普通的 JavaScript 编写的，因此，类型定义的维护并不容易。可喜的是，自版本 2 以来，维护者和贡献者们已经在类型维护上投入了巨大的努力。

版本 3 的类型系统发生了改变。新的系统带来了泛型约束 (generic constraining) 与默认值，以及定义请求 body，querystring 等 schema 的新方式！在团队改善框架和类型定义的协作中，难免有所纰漏。我们鼓励你**参与贡献**。请记得在开始前阅读 [`CONTRIBUTING.md`](https://github.com/fastify/fastify/blob/HEAD/CONTRIBUTING.md) 一文！

> 本文档介绍的是 Fastify 3.x 版本的类型

> 插件不一定包含类型定义。更多内容请看[插件](#plugins)。我们鼓励用户提交 PR 来改善插件的类型支持。

别忘了安装 `@types/node`。

## 从例子中学习

通过例子来学习 Fastify 的类型系统是最好的途径！以下四个例子涵盖了最常见的开发场景。例子之后是更详尽深入的文档。

### 起步

这个例子展示了如何使用 Fastify 和 TypeScript 构建一个最简单的 http 服务器。

1. 创建一个 npm 项目，安装 Fastify、typescript 和 node.js 的类型文件：
  ```bash
  npm init -y
  npm i fastify
  npm i -D typescript @types/node
  ```
2. 在 `package.json` 的 `"scripts"` 里添加以下内容：
  ```json
  {
    "scripts": {
      "build": "tsc -p tsconfig.json",
      "start": "node index.js"
    }
  }
  ```

*注：在 `tsconfig.json` 中将 `target` 设置为 `es2017` 或更高，以避免 [FastifyDeprecation](https://github.com/fastify/fastify/issues/3284) 的警告。*
  
3. 初始化 TypeScript 配置文件：
  ```bash
  npx tsc --init
  ```
  或使用一个[推荐的配置文件](https://github.com/tsconfig/bases#node-10-tsconfigjson)。
4. 创建 `index.ts` ，在此编写服务器的代码。
5. 将下列代码添加到该文件中：
   ```typescript
   import fastify from 'fastify'

   const server = fastify()

   server.get('/ping', async (request, reply) => {
     return 'pong\n'
   })

   server.listen(8080, (err, address) => {
     if (err) {
       console.error(err)
       process.exit(1)
     }
     console.log(`Server listening at ${address}`)
   })
   ```
6. 执行 `npm run build`。这么做会将 `index.ts` 编译为能被 Node.js 运行的 `index.js`。如果遇到了错误，请在 [fastify/help](https://github.com/fastify/help/) 发布 issue。
7. 执行 `npm run start` 来启动 Fastify 服务器。
8. 你将看到控制台输出： `Server listening at http://127.0.0.1:8080`。
9. 通过 `curl localhost:8080/ping` 访问服务，你将收到 `pong`。

🎉 现在，你有了一个能用的 TypeScript 写的 Fastify 服务器！这个例子演示了在 3.x 版本中，类型系统有多么简单。默认情况下，类型系统会假定你使用的是 `http` 服务器。后续的例子将为你展现更多内容，例如，创建较为复杂的服务器 (`https` 与 `http2`)，以及指定路由的 schema！

> 更多使用 TypeScript 初始化 Fastify 的示例 (如启用 HTTP2)，请在[这里][Fastify]查阅详细的 API。

### 使用泛型

类型系统重度依赖于泛型属性来提供最精确的开发时体验。有人可能会认为这么做有些麻烦，但这是值得的！这个例子将展示如何在路由 schema 中实现泛型，以及路由层 `request` 对象上的动态属性。

1. 照着上面例子的 1-4 步来初始化项目。
2. 在 `index.ts` 中定义两个接口 (interface)，`IQuerystring` 和 `IHeaders`：
   ```typescript
   interface IQuerystring {
     username: string;
     password: string;
   }

   interface IHeaders {
     'h-Custom': string;
   }
   ```
3. 使用这两个接口，定义一个新的 API 路由，并将它们用作泛型。路由方法的简写形式 (如 `.get`) 接受一个泛型对象 `RouteGenericInterface`，它包含了五个具名属性：`Body`、`Querystring`、`Params`、`Headers` 以及 `Reply`。`Body`、`Querystring`、`Params` 和 `Headers` 四个接口会随着路由方法向下传递，到达路由处理函数中的 `request` 实例，`Reply` 接口则会到达 `reply` 实例。
   ```typescript
   server.get<{
     Querystring: IQuerystring,
     Headers: IHeaders
   }>('/auth', async (request, reply) => {
     const { username, password } = request.query
     const customerHeader = request.headers['h-Custom']
     // 处理请求数据

     return `logged in!`
   })
   ```

4. 执行 `npm run build` 和 `npm run start` 来构建并运行项目。
5. 访问 api：
   ```bash
   curl localhost:8080/auth?username=admin&password=Password123!
   ```
   将会返回 `logged in!`。
6. 此外，泛型接口还可以用在路由层钩子方法中。在上面的路由内加上一个 `preValidation` 钩子：
   ```typescript
   server.get<{
     Querystring: IQuerystring,
     Headers: IHeaders
   }>('/auth', {
     preValidation: (request, reply, done) => {
       const { username, password } = request.query
       done(username !== 'admin' ? new Error('Must be admin') : undefined)
     }
     //  或使用 async
     //  preValidation: async (request, reply) => {
     //    const { username, password } = request.query
     //    return username !== "admin" ? new Error("Must be admin") : undefined;
     //  }
   }, async (request, reply) => {
     const customerHeader = request.headers['h-Custom']
     // 处理请求数据
     return `logged in!`
   })
   ```
7. 构建运行之后，使用任何值不为 `admin` 的 `username` 查询字符串访问服务。你将收到一个 500 错误：`{"statusCode":500,"error":"Internal Server Error","message":"Must be admin"}`

   干得漂亮。现在你能够为每个路由定义接口，并拥有严格类型的请求与响应实例了。Fastify 类型系统的其他部分依赖于泛型属性。关于如何使用它们，请参照后文详细的类型系统文档。

### JSON Schema

你可以通过 JSON Schema 来验证请求与响应。给 Fastify 路由定义 schema 还能提高吞吐量！更多信息请见[验证和序列化](Validation-and-Serialization.md)。

此外，在路由处理函数 (包括 pre-validation 等钩子) 中使用定义好的类型也是有好处的。

以下列出了几种实现方案。

#### typebox

[typebox](https://www.npmjs.com/package/@sinclair/typebox) 能帮助你同时构建类型与 schema。通过 typebox 在代码里定义好 schema 之后，你便能将其当作类型或 schema 来使用。

在 Fastify 路由中验证 payload，你可以这么做：

1. 安装 `typebox`。

    ```bash
    npm i @sinclair/typebox
    ```

2. 使用 `Type` 定义 schema，并通过 `Static` 创建相应的类型。

    ```typescript
    import { Static, Type } from '@sinclair/typebox'

    const User = Type.Object({
      name: Type.String(),
      mail: Type.Optional(Type.String({ format: "email" })),
    });
    type UserType = Static<typeof User>;
    ```

3. 在路由中使用定义好的类型与 schema。

    ```typescript
    const app = fastify();

    app.post<{ Body: UserType; Reply: UserType }>(
      "/",
      {
        schema: {
          body: User,
          response: {
            200: User,
          },
        },
      },
      (req, rep) => {
        const { body: user } = req;
        /* user 的类型如下：
        * const user: StaticProperties<{
        *  name: TString;
        *  mail: TOptional<TString>;
        * }>
        */
        //...
        rep.status(200).send(user);
      }
    );
    ```

#### Schemas in JSON Files

在上一个例子里，我们使用接口定义了请求 querystring 和 header 的类型。许多用户使用 JSON Schema 来处理这些工作，幸运的是，有一套方法能将现有的 JSON Schema 转换为 TypeScript 接口！

1. 完成 '起步' 中例子的 1-4 步。
2. 安装 `json-schema-to-typescript` 模块：

   ```bash
   npm i -D json-schema-to-typescript
   ```

3. 新建一个名为 `schemas` 的文件夹。在其中添加 `headers.json` 与 `querystring.json` 两个文件，将下面的 schema 定义粘贴到对应文件中。

   ```json
   {
     "title": "Headers Schema",
     "type": "object",
     "properties": {
       "h-Custom": { "type": "string" }
     },
     "additionalProperties": false,
     "required": ["h-Custom"]
   }
   ```

   ```json
   {
     "title": "Querystring Schema",
     "type": "object",
     "properties": {
       "username": { "type": "string" },
       "password": { "type": "string" }
     },
     "additionalProperties": false,
     "required": ["username", "password"]
   }
   ```

4. 在 package.json 里加上一行 `compile-schemas` 脚本：

   ```json
   {
     "scripts": {
       "compile-schemas": "json2ts -i schemas -o types"
     }
   }
   ```

   `json2ts` 是囊括在 `json-schema-to-typescript` 中的命令行工具。`schemas` 是输入路径，`types` 则是输出路径。
5. 执行 `npm run compile-schemas`，在 `types` 文件夹下生成两个新文件。
6. 更新 `index.ts`：

   ```typescript
   import fastify from 'fastify'

   // 导入 json schema
   import QuerystringSchema from './schemas/querystring.json'
   import HeadersSchema from './schemas/headers.json'

   // 导入生成的接口
   import { QuerystringSchema as QuerystringSchemaInterface } from './types/querystring'
   import { HeadersSchema as HeadersSchemaInterface } from './types/headers'

   const server = fastify()

   server.get<{
     Querystring: QuerystringSchemaInterface,
     Headers: HeadersSchemaInterface
   }>('/auth', {
     schema: {
       querystring: QuerystringSchema,
       headers: HeadersSchema
     },
     preValidation: (request, reply, done) => {
       const { username, password } = request.query
       done(username !== 'admin' ? new Error('Must be admin') : undefined)
     }
   }, async (request, reply) => {
     const customerHeader = request.headers['h-Custom']
     // 处理请求数据
     return `logged in!`
   })

   server.route<{
     Querystring: QuerystringSchemaInterface,
     Headers: HeadersSchemaInterface
   }>({
     method: 'GET',
     url: '/auth2',
     schema: {
       querystring: QuerystringSchema,
       headers: HeadersSchema
     },
     preHandler: (request, reply, done) => {
       const { username, password } = request.query
       const customerHeader = request.headers['h-Custom']
       done()
     },
     handler: (request, reply) => {
       const { username, password } = request.query
       const customerHeader = request.headers['h-Custom']
       reply.status(200).send({username});
     }
   })

   server.listen(8080, (err, address) => {
     if (err) {
       console.error(err)
       process.exit(0)
     }
     console.log(`Server listening at ${address}`)
   })
   ```
   要特别关注文件顶部的导入。虽然看上去有些多余，但你必须同时导入 schema 与生成的接口。

真棒！现在你就能同时运用 JSON Schema 与 TypeScript 的定义了。

#### json-schema-to-ts

不想基于 schema 生成类型，而是直接使用它们的话，你可以考虑 [json-schema-to-ts](https://www.npmjs.com/package/json-schema-to-ts) 模块。

安装该模块为 dev-dependency：

```bash
npm install -D json-schema-to-ts
```

你可以像定义正常的对象一样定义 schema。但得注意要用 *const* 来定义，原因见该模块的文档。

```typescript
const todo = {
  type: 'object',
  properties: {
    name: { type: 'string' },
    description: { type: 'string' },
    done: { type: 'boolean' },
  },
  required: ['name'],
} as const;
```

通过类型 `FromSchema` 你可以基于 schema 构建一个类型，并在函数中使用它。

```typescript
import { FromSchema } from "json-schema-to-ts";
fastify.post<{ Body: FromSchema<typeof todo> }>(
  '/todo',
  {
    schema: {
      body: todo,
      response: {
        201: {
          type: 'string',
        },
      },
    }
  },
  async (request, reply): Promise<void> => {

    /*
    request.body 的类型如下：
    {
      [x: string]: unknown;
      description?: string;
      done?: boolean;
      name: string;
    }
    */

    request.body.name // 不会抛出类型错误
    request.body.notthere // 会抛出类型错误
    
    reply.status(201).send();
  },
);
```

### 插件

拓展性强的插件生态系统是 Fastify 最突出的特性之一。插件完全支持类型系统，并利用了[声明合并]() (declaration merging) 模式的优势。下面的例子将分为三个部分：用 TypeScript 编写 Fastify 插件，为插件编写类型定义，以及在 TypeScript 项目中使用插件。

#### 用 TypeScript 编写 Fastify 插件

1. 初始化新的 npm 项目，并安装必需的依赖。
   ```bash
   npm init -y
   npm i fastify fastify-plugin
   npm i -D typescript @types/node
   ```
2. 在 `package.json` 的 `"scripts"` 中加上一行 `build`，`"types"` 中写入 `'index.d.ts'`：
   ```json
   {
     "types": "index.d.ts",
     "scripts": {
       "build": "tsc -p tsconfig.json"
     }
   }
   ```
3. 初始化 TypeScript 配置文件：
   ```bash
   npx typescript --init
   ```
   文件生成后，启用 `"compilerOptions"` 对象中的 `"declaration"` 选项。
   ```json
   {
     "compileOptions": {
       "declaration": true
     }
   }
   ```
4. 新建 `index.ts` 文件，在这里编写插件代码。
5. 在 `index.ts` 中写入以下代码。
   ```typescript
   import { FastifyPluginCallback, FastifyPluginAsync } from 'fastify'
   import fp from 'fastify-plugin'

   // 利用声明合并，将插件的属性加入合适的 fastify 接口。
   declare module 'fastify' {
     interface FastifyRequest {
       myPluginProp: string
     }
     interface FastifyReply {
       myPluginProp: number
     }
   }

   // 定义选项
   export interface MyPluginOptions {
     myPluginOption: string
   }

   // 使用回调函数定义插件
   const myPluginCallback: FastifyPluginCallback<MyPluginOptions> = (fastify, options, done) => {
     fastify.decorateRequest('myPluginProp', 'super_secret_value')
     fastify.decorateReply('myPluginProp', options.myPluginOption)

     done()
   }

   // 使用 promise 定义插件
   const myPluginAsync: FastifyPluginAsync<MyPluginOptions> = async (fastify, options) => {
     fastify.decorateRequest('myPluginProp', 'super_secret_value')
     fastify.decorateReply('myPluginProp', options.myPluginOption)
   }

   // 使用 fastify-plugin 导出插件
   export default fp(myPluginCallback, '3.x')
   // 或者
   // export default fp(myPluginAsync, '3.x')
   ```
6. 运行 `npm run build` 编译，生成 JavaScript 源文件以及类型定义文件。
7. 如此一来，插件便完工了。你可以[发布到 npm] 或直接本地使用。
   > 并非将插件发布到 npm _才能_ 使用。你可以将其放在 Fastify 项目内，并像引用任意代码一样引用它！请确保声明文件在项目编译的范围内，以便能被 TypeScript 处理器使用。

#### 为插件编写类型定义

以下例子是为 JavaScript 编写的 Fastify 插件所作，展示了如何在插件中加入 TypeScript 支持，以方便用户使用。

1. 初始化新的 npm 项目，并安装必需的依赖。
   ```bash
   npm init -y
   npm i fastify-plugin
   ```
2. 新建 `index.js` 和 `index.d.ts`。
3. 将这两个文件写入 package.json 的 `main` 和 `types` 中 (文件名不一定为 `index`，但推荐都使用这个名字)：
   ```json
   {
     "main": "index.js",
     "types": "index.d.ts"
   }
   ```
4. 在 `index.js` 中加入以下代码：
   ```javascript
   // 极力推荐使用 fastify-plugin 包装插件
   const fp = require('fastify-plugin')

   function myPlugin (instance, options, done) {

     // 用自定义函数 myPluginFunc 装饰 fastify 实例
     instance.decorate('myPluginFunc', (input) => {
       return input.toUpperCase()
     })

     done()
   }
  
   module.exports = fp(myPlugin, {
     fastify: '3.x',
     name: 'my-plugin' // 被 fastify-plugin 用来获取属性名
   })
   ```
5. 在 `index.d.ts` 中加入以下代码：
   ```typescript
   import { FastifyPlugin } from 'fastify'

   interface PluginOptions {
     //...
   }
   // 你可以导出任意内容
   // 在此，我们导出之前添加的装饰器
   export interface myPluginFunc {
     (input: string): string
   }

   // 利用声明合并将自定义属性加入 Fastify 的类型系统
   declare module 'fastify' {
     interface FastifyInstance {
       myPluginFunc: myPluginFunc
     }
   }

   // fastify-plugin 会自动添加具名导出，因此请确保加上该类型。
   // 如果缺少 `module.exports.myPlugin`，变量名会通过 `options.name` 属性获取。
   export const myPlugin: FastifyPlugin<PluginOptions>
   // fastify-plugin 会自动在导出的插件上添加 `.default` 属性。详见下文。
   export default myPlugin
   ```

__注意__：v2.3.0 及以上版本的 [fastify-plugin](https://github.com/fastify/fastify-plugin) 会自动给导出的插件添加 `default` 属性以及具名导出。为了更好的开发体验，请确保在类型文件中加上了 `export default` 与 `export const myPlugin`。完整的例子可以查看 [fastify-swagger](https://github.com/fastify/fastify-swagger/blob/main/index.d.ts)。

这样一来，该插件便能被任意 TypeScript 项目使用了！

Fastify 的插件系统允许开发者装饰 Fastify 以及 request/reply 的实例。更多信息请见[声明合并与泛型继承](https://dev.to/ethanarrowood/is-declaration-merging-and-generic-inheritance-at-the-same-time-impossible-53cp)一文。

#### 使用插件

在 TypeScript 中使用插件和在 JavaScript 中使用一样简单，只需要用到 `import/from` 而已，除了一个特殊情况。

Fastify 插件使用声明合并来修改已有的 Fastify 类型接口 (详见上一个例子)。声明合并没有那么 _聪明_，只要插件的类型定义在 TypeScript 解释器的范围内，那么**不管**插件本身是否被使用，这些定义都会被包括。这是 TypeScript 的限制，目前无法规避。

尽管如此，还是有一些建议能帮助改善这种状况：
- 确保 [ESLint](https://eslint.org/docs/rules/no-unused-vars) 开启了 `no-unused-vars`，并且所有导入的插件都得到了加载。
- 通过诸如 [depcheck](https://www.npmjs.com/package/depcheck) 或 [npm-check](https://www.npmjs.com/package/npm-check) 的工具来验证所有插件都在项目中得到了使用。

## 原生 JavaScript 的代码补全

原生 JavaScript 能根据发布好的类型，依照 [TypeScript 的 JSDoc 参考](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html) 来完成代码补全 (例如 [Intellisense](https://code.visualstudio.com/docs/editor/intellisense))。

举个例子：

```js
/**  @type {import('fastify').FastifyPluginAsync<{ optionA: boolean, optionB: string }>} */
module.exports = async function (fastify, { optionA, optionB }) {
  fastify.get('/look', () => 'at me');
}
```

## API 类型系统文档

本节详述了所有在 Fastify 3.x 版本中可用的类型。

所有 `http`、`https` 以及 `http2` 的类型来自 `@types/node`。

[泛型](#generics)的文档包括了其默认值以及约束值。更多关于 TypeScript 泛型的信息请阅读以下文章。
- [泛型参数默认值](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-3.html#generic-parameter-defaults)
- [泛型约束](https://www.typescriptlang.org/docs/handbook/generics.html#generic-constraints)

#### 如何导入

Fastify 的 API 都首先来自于 `fastify()` 方法。在 JavaScript 中，通过 `const fastify = require('fastify')` 来导入。在 TypeScript 中，建议的做法是使用 `import/from` 语法，这样类型能得到处理。有如下几种导入的方法。

1. `import fastify from 'fastify'`
   - 类型得到了处理，但无法通过点标记 (dot notation) 访问
   - 例子：
     ```typescript
     import fastify from 'fastify'

     const f = fastify()
     f.listen(8080, () => { console.log('running') })
     ```
   - 通过解构赋值访问类型
     ```typescript
     import fastify, { FastifyInstance } from 'fastify'

     const f: FastifyInstance = fastify()
     f.listen(8080, () => { console.log('running') })
     ```
   - 主 API 方法也可以使用解构赋值
     ```typescript
     import { fastify, FastifyInstance } from 'fastify'

     const f: FastifyInstance = fastify()
     f.listen(8080, () => { console.log('running') })
     ```
2. `import * as Fastify from 'fastify'`
   - 类型得到了处理，并可通过点标记访问
   - 主 API 方法要用稍微不同的语法调用 (见例子)
   - 例子：
     ```typescript
     import * as Fastify from 'fastify'

     const f: Fastify.FastifyInstance = Fastify.fastify()
     f.listen(8080, () => { console.log('running') })
     ```
3. `const fastify = require('fastify')`
   - 语法有效，也能正确地导入。然而并**不**支持类型
   - 例子：
     ```typescript
     const fastify = require('fastify')

     const f = fastify()
     f.listen(8080, () => { console.log('running') })
     ```
   - 支持解构，且能处理类型
     ```typescript
     const { fastify } = require('fastify')

     const f = fastify()
     f.listen(8080, () => { console.log('running') })
     ```

#### 泛型

许多类型定义共用了某些泛型参数。它们都在本节有详尽的描述。

多数定义依赖于 `@node/types` 中的 `http`、`https` 与 `http2` 模块。

##### RawServer
底层 Node.js server 的类型。

默认值：`http.Server`

约束值：`http.Server`、`https.Server`、`http2.Http2Server`、`http2.Http2SecureServer`

必要的泛型参数 (Enforces generic parameters)：[`RawRequest`][RawRequestGeneric]、[`RawReply`][RawReplyGeneric]

##### RawRequest
底层 Node.js request 的类型。

默认值：[`RawRequestDefaultExpression`][RawRequestDefaultExpression]

约束值：`http.IncomingMessage`、`http2.Http2ServerRequest`

被 [`RawServer`][RawServerGeneric] 约束。

##### RawReply
底层 Node.js response 的类型。

默认值：[`RawReplyDefaultExpression`][RawReplyDefaultExpression]

约束值：`http.ServerResponse`、`http2.Http2ServerResponse`

被 [`RawServer`][RawServerGeneric]  约束。

##### Logger
Fastify 日志工具。

默认值：[`FastifyLoggerOptions`][FastifyLoggerOptions]

被 [`RawServer`][RawServerGeneric]  约束。

##### RawBody
为 content-type-parser 方法提供的泛型参数。

约束值：`string | Buffer`

---

#### Fastify

##### fastify<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [Logger][LoggerGeneric]>(opts?: [FastifyServerOptions][FastifyServerOptions]): [FastifyInstance][FastifyInstance]
[源码](https://github.com/fastify/fastify/blob/main/fastify.d.ts#L19)

Fastify 首要的 API 方法。默认情况下创建一个 HTTP 服务器。通过可辨识联合 (discriminant unions) 及重载的方法 (overload methods)，类型系统能自动地根据传递给该方法的选项 (详见下文例子)，推断出服务器的类型 (http、https 或 http2)。同时，可拓展的泛型类型系统允许用户拓展底层的 Node.js Server、Request 和 Reply 对象。此外，自定义日志类型则可以运用 `Logger` 泛型。详见下文的例子和泛型分类说明。

###### 例子 1：标准的 HTTP 服务器

无需指明 `Server` 的具体类型，因为默认值就是 HTTP 服务器。
```typescript
import fastify from 'fastify'

const server = fastify()
```
回顾“从例子中学习”的[起步](#getting-started)一节的示例来获取更详细的内容。

###### 例子 2：HTTPS 服务器

1. 从 `@types/node` 与 `fastify` 导入模块。
   ```typescript
   import fs from 'fs'
   import path from 'path'
   import fastify from 'fastify'
   ```
2. 按照官方 [Node.js https 服务器指南](https://nodejs.org/en/knowledge/HTTP/servers/how-to-create-a-HTTPS-server/)的步骤，创建 `key.pem` 与 `cert.pem` 文件。
3. 实例化一个 Fastify https 服务器，并添加一个路由：
   ```typescript
   const server = fastify({
     https: {
       key: fs.readFileSync(path.join(__dirname, 'key.pem')),
       cert: fs.readFileSync(path.join(__dirname, 'cert.pem'))
     }
   })

   server.get('/', async function (request, reply) {
     return { hello: 'world' }
   })

   server.listen(8080, (err, address) => {
     if (err) {
       console.error(err)
       process.exit(0)
     }
     console.log(`Server listening at ${address}`)
   })
   ```
4. 构建并运行！执行 `curl -k https://localhost:8080` 来测试服务。

###### 例子 3：HTTP2 服务器

HTTP2 服务器有两种类型，非安全与安全。两种类型都需要在 `options` 对象中设置 `http2` 属性的值为 `true`。设置 `https` 属性会创建一个安全的 http2 服务器；忽略该属性则创建非安全的服务器。

```typescript
const insecureServer = fastify({ http2: true })
const secureServer = fastify({
  http2: true,
  https: {} // 使用 https 服务的 `key.pem` 和 `cert.pem` 文件
})
```

更多细节详见 Fastify 的 [HTTP2](HTTP2.md) 文档。

###### 例子 4：拓展 HTTP 服务器

你不仅可以指定服务器的类型，还可以指定请求与响应的类型，即指定特殊的属性、方法等！在服务器实例化之时指定类型，则之后的实例都可应用自定义的类型。
```typescript
import fastify from 'fastify'
import http from 'http'

interface customRequest extends http.IncomingMessage {
  mySpecialProp: string
}

const server = fastify<http.Server, customRequest>()

server.get('/', async (request, reply) => {
  const someValue = request.raw.mySpecialProp // 由于 `customRequest` 接口的存在，TypeScript 能知道这是一个字符串
  return someValue.toUpperCase()
})
```

###### 例子 5：指定日志类型

Fastify 使用 [Pino](https://getpino.io/#/) 作为日志工具。其中一些属性可以在构建 Fastify 实例时，在 `logger` 字段中配置。如果需要的属性未被暴露出来，你也能通过将一个外部配置好的 Pino 实例 (或其他兼容的日志工具) 传给这个字段，来配置这些属性。这么做也允许你自定义序列化工具，详见[日志](Logging.md)的文档。

要使用 Pino 的外部实例，请将 `@types/pino` 添加到 devDependencies 中，并把实例传给 `logger` 字段：

```typescript
import fastify from 'fastify'
import pino from 'pino'

const server = fastify({
  logger: pino({
    level: 'info',
    redact: ['x-userinfo'],
    messageKey: 'message'
  })
})

server.get('/', async (request, reply) => {
  server.log.info('log message')
  return 'another message'
})
```

---

##### fastify.HTTPMethods
[源码](https://github.com/fastify/fastify/blob/main/types/utils.d.ts#L8)

`'DELETE' | 'GET' | 'HEAD' | 'PATCH' | 'POST' | 'PUT' | 'OPTIONS'` 的联合类型 (Union type)

##### fastify.RawServerBase
[源码](https://github.com/fastify/fastify/blob/main/types/utils.d.ts#L13)

依赖于 `@types/node` 的模块 `http`、`https`、`http2`

`http.Server | https.Server | http2.Http2Server | http2.Http2SecureServer` 的联合类型

##### fastify.RawServerDefault
[源码](https://github.com/fastify/fastify/blob/main/types/utils.d.ts#L18)

依赖于 `@types/node` 的模块 `http`

`http.Server` 的类型别名

---

##### fastify.FastifyServerOptions<[RawServer][RawServerGeneric], [Logger][LoggerGeneric]>

[源码](../fastify.d.ts#L29)

Fastify 服务器实例化时，调用 [`fastify()`][Fastify] 方法使用到的接口。泛型参数 `RawServer` 和 `Logger` 会随此方法向下传递。

关于用 TypeScript 实例化一个 Fastify 服务器的例子，请见 [fastify][Fastify] 主方法的类型定义。

##### fastify.FastifyInstance<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RequestGeneric][FastifyRequestGenericInterface], [Logger][LoggerGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/instance.d.ts#L16)

表示 Fastify 服务器对象的接口，[`fastify()`][Fastify] 方法的返回值。假如你使用了 `decorate` 方法，借由[声明合并](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)可以拓展该接口。

通过泛型级联 (generic cascading)，实例上所有的方法都能继承实例化时的泛型属性。这意味着只要指定了服务器、请求或响应的类型，所有方法都能随之确定这些对象的类型。

具体说明请看“[从例子中学习](#learn-by-example)”一节，或 [fastify][Fastify] 方法中更简洁的例子。

---

#### Request

##### fastify.FastifyRequest<[RequestGeneric][FastifyRequestGenericInterface], [RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric]>
[源码](https://github.com/fastify/fastify/blob/main/types/request.d.ts#L15)

该接口包含了 Fastify 请求对象的属性。这些属性无视请求类型 (http 或 http2)，也无关路由层级。因此在 GET 请求中访问 `request.body` 并不会抛错 (假如 GET 有 body 😉)。

假如你需要为 `FastifyRequest` 对象添加自定义属性 (例如使用 [`decorateRequest`][DecorateRequest] 方法时)，你应该针对该接口应用声明合并。

在 [`FastifyRequest`][FastifyRequest] 里有基本的范例。更详细的例子请见“从例子中学习”的[插件](#plugins)一节。

###### 例子
```typescript
import fastify from 'fastify'

const server = fastify()

server.decorateRequest('someProp', 'hello!')

server.get('/', async (request, reply) => {
  const { someProp } = request // 需要通过声明合并将该属性添加到 request 接口上
  return someProp
})

// 以下声明必须在 typescript 解释器的作用域内
declare module 'fastify' {
  interface FastifyRequest { // 引用的是接口而非类型
    someProp: string
  }
}

// 你也可以如此定义 request 的类型
type CustomRequest = FastifyRequest<{
  Body: { test: boolean };
}>

server.get('/typedRequest', async (request: CustomRequest, reply: FastifyReply) => {
  return request.body.test
})
```

##### fastify.RequestGenericInterface
[源码](https://github.com/fastify/fastify/blob/main/types/request.d.ts#L4)

Fastify 的请求对象有四个动态属性：`body`、`params`、`query` 以及 `headers`，它们对应的类型可以通过该接口设定。这是具名属性接口，允许开发者忽略他们不想指定的类型。所有忽略的属性默认为 `unknown`。四个属性名为：`Body`、`Querystring`、`Params` 和 `Headers`。

```typescript
import fastify, { RequestGenericInterface } from 'fastify'

const server = fastify()

interface requestGeneric extends RequestGenericInterface {
  Querystring: {
    name: string
  }
}

server.get<requestGeneric>('/', async (request, reply) => {
  const { name } = request.query // 此时 query 属性上有了 name
  return name.toUpperCase()
})
```

在“从例子中学习”的 [JSON Schema](#jsonschema) 一节中，你能找到更具体的范例。

##### fastify.RawRequestDefaultExpression\<[RawServer][RawServerGeneric]\>
[源码](https://github.com/fastify/fastify/blob/main/types/utils.d.ts#L23)

依赖于 `@types/node` 的模块 `http`、`https`、`http2`

泛型参数 `RawServer` 的默认值为 [`RawServerDefault`][RawServerDefault]

如果 `RawServer` 的类型为 `http.Server` 或 `https.Server`，那么该表达式返回 `http.IncomingMessage`，否则返回 `http2.Http2ServerRequest`。

```typescript
import http from 'http'
import http2 from 'http2'
import { RawRequestDefaultExpression } from 'fastify'

RawRequestDefaultExpression<http.Server> // -> http.IncomingMessage
RawRequestDefaultExpression<http2.Http2Server> // -> http2.Http2ServerRequest
```

---

#### Reply

##### fastify.FastifyReply<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>
[源码](https://github.com/fastify/fastify/blob/main/types/reply.d.ts#L32)

该接口包含了 Fastify 添加到 Node.js 标准的 reply 对象上的属性。这些属性和 reply 对象的类型 (http 或 http2) 无关。

假如你需要为 FastifyReply 对象添加自定义属性 (例如使用 `decorateReply` 方法时)，你应该针对该接口应用声明合并。

在 [`FastifyReply`][FastifyReply] 里有基本的范例。更详细的例子请见“从例子中学习”的[插件](#plugins)一节。

###### 例子
```typescript
import fastify from 'fastify'

const server = fastify()

server.decorateReply('someProp', 'world')

server.get('/', async (request, reply) => {
  const { someProp } = reply //需要通过声明合并将该属性添加到 reply 接口上
  return someProp
})

// 以下声明必须在 typescript 解释器的作用域内
declare module 'fastify' {
  interface FastifyReply { // 引用的是接口而非类型
    someProp: string
  }
}
```

##### fastify.RawReplyDefaultExpression<[RawServer][RawServerGeneric]>
[源码](https://github.com/fastify/fastify/blob/main/types/utils.d.ts#L27)

依赖于 `@types/node` 的模块 `http`、`https`、`http2`

泛型参数 `RawServer` 的默认值为 [`RawServerDefault`][RawServerDefault]

如果 `RawServer` 的类型为 `http.Server` 或 `https.Server`，那么该表达式返回 `http.ServerResponse`，否则返回 `http2.Http2ServerResponse`。

```typescript
import http from 'http'
import http2 from 'http2'
import { RawReplyDefaultExpression } from 'fastify'

RawReplyDefaultExpression<http.Server> // -> http.ServerResponse
RawReplyDefaultExpression<http2.Http2Server> // -> http2.Http2ServerResponse
```

---

#### 插件

通过插件，用户能拓展 Fastify 的功能。一个插件可以是一组路由，也可以是一个装饰器，或其它逻辑。要激活一个插件，需调用 [`fastify.register()`][FastifyRegister] 方法。

创建插件时，我们推荐使用 `fastify-plugin`。在“从例子中学习”的[插件](#plugins)一节中有使用 TypeScript 创建插件的指南。

##### fastify.FastifyPluginCallback<[Options][FastifyPluginOptions]>
[源码](https://github.com/fastify/fastify/blob/main/types/plugin.d.ts#L9)

[`fastify.register()`][FastifyRegister] 使用的接口方法定义。

##### fastify.FastifyPluginAsync<[Options][FastifyPluginOptions]>
[源码](https://github.com/fastify/fastify/blob/main/types/plugin.d.ts#L20)

[`fastify.register()`][FastifyRegister] 使用的接口方法定义。

##### fastify.FastifyPlugin<[Options][FastifyPluginOptions]>
[源码](https://github.com/fastify/fastify/blob/main/types/plugin.d.ts#L29)

[`fastify.register()`][FastifyRegister] 使用的接口方法定义。

通用的 `FastifyPlugin` 已不推荐使用，取而代之的是上述 `FastifyPluginCallback` 以及 `FastifyPluginAsync`。这是因为 `FastifyPlugin` 无法正确推断出异步函数的类型。

##### fastify.FastifyPluginOptions
[源码](https://github.com/fastify/fastify/blob/main/types/plugin.d.ts#L31)

一个用于约束 [`fastify.register()`][FastifyRegister] 的 `options` 参数为对象类型的宽松类型对象 (loosely typed object)。在创建插件时，将插件的选项定义为此接口 (`interface MyPluginOptions extends FastifyPluginOptions`)，传递给 register 方法。

---

#### Register

##### fastify.FastifyRegister(plugin: [FastifyPluginCallback][FastifyPluginCallback], opts: [FastifyRegisterOptions][FastifyRegisterOptions])
[源码](https://github.com/fastify/fastify/blob/main/types/register.d.ts#L9)
##### fastify.FastifyRegister(plugin: [FastifyPluginAsync][FastifyPluginAsync], opts: [FastifyRegisterOptions][FastifyRegisterOptions])
[源码](https://github.com/fastify/fastify/blob/main/types/register.d.ts#L9)
##### fastify.FastifyRegister(plugin: [FastifyPlugin][FastifyPlugin], opts: [FastifyRegisterOptions][FastifyRegisterOptions])
[源码](https://github.com/fastify/fastify/blob/main/types/register.d.ts#L9)

指定 [`fastify.register()`](Server.md#register) 类型的类型接口，返回一个拥有默认值为 [FastifyPluginOptions][FastifyPluginOptions] 的 `Options` 泛型的函数签名。当调用此函数时，根据 FastifyPlugin 参数能推断出该泛型，因此不必特别指定。options 参数是插件选项以及 `prefix: string` 和 `logLevel` ([LogLevel][LogLevel]) 两个属性的交叉类型。

以下例子展示了 options 的推断：

```typescript
const server = fastify()

const plugin: FastifyPlugin<{
  option1: string;
  option2: boolean;
}> = function (instance, opts, done) { }

fastify().register(plugin, {}) // 错误 - options 对象缺失了必要的属性
fastify().register(plugin, { option1: '', option2: true }) // OK - options 对象包括了必要的属性
```

在“从例子中学习”的[插件](#plugins)一节中有使用 TypeScript 创建插件的详细示例。

##### fastify.FastifyRegisterOptions<Options>
[源码](https://github.com/fastify/fastify/blob/main/types/register.d.ts#L16)

该类型是 `Options` 泛型以及包括 `prefix: string` 和 `logLevel` ([LogLevel][LogLevel]) 两个可选属性的未导出接口 `RegisterOptions` 的交叉类型。也可以被指定为返回前述交叉类型的函数。

---

#### 日志

请在[指定日志类型](#example-5-specifying-logger-types)的例子中，查阅自定义日志工具的细节。

##### fastify.FastifyLoggerOptions<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/logger.d.ts#L17)

Fastify 内建日志工具的接口定义，模仿了 [Pino.js](https://getpino.io/#/) 的接口定义。当通过服务器选项启用日志时，参照[日志](Logging.md)文档使用它。

##### fastify.FastifyLogFn

[源码](https://github.com/fastify/fastify/blob/main/types/logger.d.ts#L7)

一个重载函数接口，实现 Fastify 调用日志的方法，会传递到所有 FastifyLoggerOptions 中启用的日志级别属性。

##### fastify.LogLevel

[源码](https://github.com/fastify/fastify/blob/main/types/logger.d.ts#L12)

`'info' | 'error' | 'debug' | 'fatal' | 'warn' | 'trace'` 的联合类型

---

#### Context

context 类型定义和类型系统中其它高度动态化的部分类似。路由上下文 (context) 在路由函数内可用。

##### fastify.FastifyContext

[源码](https://github.com/fastify/fastify/blob/main/types/context.d.ts#L6)

有一个默认为 `unknown` 的必填属性 `config` 的接口。可用泛型或重载来指定。

此类型定义可能尚不完善。假如你有改进它的建议，欢迎在 [fastify/fastify](https://github.com/fastify/fastify) 仓库发布一个 issue。感谢！

---

#### 路由

Fastify 的其中一条核心原则便是强大的路由。本节中多数的类型被 Fastify 实例的 `.route` 及 `.get/.post` 等方法内在地使用。

##### fastify.RouteHandlerMethod<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/route.d.ts#L105)

路由控制函数的类型声明，有两个参数：类型为 `FastifyRequest` 的 `request`，以及类型为 `FastifyReply` 的 `reply`。泛型参数会传递给这些参数。当控制函数为同步函数时，返回 `void`，异步则返回 `Promise<any>`。

##### fastify.RouteOptions<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/route.d.ts#L78)

拓展了 RouteShorthandOptions 的接口，并添加以下三个必填属性：
1. `method` 单个或一组 [HTTP 方法][HTTPMethods]。
2. `url` 路由路径字符串。
3. `handler` 路由控制函数，详见 [RouteHandlerMethod][]。

##### fastify.RouteShorthandMethod<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/route.d.ts#12)

一个重载函数接口，用于定义 `.get/.post` 等简写方法的三种不同形式。

##### fastify.RouteShorthandOptions<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/route.d.ts#55)

包含所有路由基本选项的接口。所有属性都是可选的。该接口是 RouteOptions 和 RouteShorthandOptionsWithHandler 接口的基础。

##### fastify.RouteShorthandOptionsWithHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/route.d.ts#93)

向 RouteShorthandOptions 接口添加一个必填属性：`handler`，类型为 RouteHandlerMethod。

---

#### Parsers

##### RawBody

一个为 `string` 或 `Buffer` 的泛型类型。

##### fastify.FastifyBodyParser<[RawBody][RawBodyGeneric], [RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/content-type-parser.d.ts#L7)

定义 body 解析器 (body parser) 的函数类型。使用 `RawBody` 泛型指定被解析的 body。

##### fastify.FastifyContentTypeParser<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/content-type-parser.d.ts#L17)

定义 body 解析器的函数类型。使用 `RawRequest` 泛型定义 content。

##### fastify.AddContentTypeParser<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric]>

[源码](https://github.com/fastify/fastify/blob/main/types/content-type-parser.d.ts#L46)

`addContentTypeParser` 方法的重载函数接口。当 `parseAs` 出现在 `opts` 参数中时，`parser` 参数使用 [FastifyBodyParser][]，否则使用 [FastifyContentTypeParser][]。

##### fastify.hasContentTypeParser

[源码](https://github.com/fastify/fastify/blob/main/types/content-type-parser.d.ts#L63)

检查指定 content type 解析器是否存在的方法。

---

#### 错误

##### fastify.FastifyError

[源码](https://github.com/fastify/fastify/blob/main/types/error.d.ts#L17)

FastifyError 是自定义的错误对象，包括了状态码及校验结果。

拓展了 Node.js 的 `Error` 类型，并加入了两个可选属性：`statusCode: number` 和 `validation: ValiationResult[]`。

##### fastify.ValidationResult

[源码](https://github.com/fastify/fastify/blob/main/types/error.d.ts#L4)

路由校验内在地依赖于 Ajv，一个高性能 JSON schema 校验工具。

该接口会传递给 FastifyError 的实例。

---

#### 钩子

##### fastify.onRequestHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], done: (err?: [FastifyError][FastifyError]) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L17)

`onRequest` 是第一个被执行的钩子，其下一个钩子为 `preParsing`。

注意：在 `onRequest` 钩子中，request.body 永远为 null，因为此时 body 尚未解析 (解析发生在 `preHandler` 钩子之前)。

##### fastify.preParsingHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], done: (err?: [FastifyError][FastifyError]) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L35)

`preParsing` 是第二个钩子，前一个为 `onRequest`，下一个为 `preValidation`。

注意：在 `preParsing` 钩子中，request.body 永远为 null，因为此时 body 尚未解析 (解析发生在 `preValidation` 钩子之前)。

注意：你应当给返回的 stream 添加 `receivedEncodedLength` 属性。这是为了通过比对请求头的 `Content-Length`，来精确匹配请求的 payload。理想情况下，每收到一块数据都应该更新该属性。

##### fastify.preValidationHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], done: (err?: [FastifyError][FastifyError]) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L53)

`preValidation` 是第三个钩子，前一个为 `preParsing`，下一个为 `preHandler`。

注意：在 `preValidation` 钩子中，request.body 永远为 null，因为此时 body 尚未解析 (解析发生在 `preHandler` 钩子之前)。 

##### fastify.preHandlerHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], done: (err?: [FastifyError][FastifyError]) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L70)

`preHandler` 是第四个钩子，前一个为 `preValidation`，下一个为 `preSerialization`。

##### fastify.preSerializationHookHandler<PreSerializationPayload, [RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], payload: PreSerializationPayload, done: (err: [FastifyError][FastifyError] | null, res?: unknown) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L94)

`preSerialization` 是第五个钩子，前一个为 `preHandler`，下一个为 `onSend`。

注：当 payload 为 string、Buffer、stream 或 null 时，该钩子不会执行。 

##### fastify.onSendHookHandler<OnSendPayload, [RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], payload: OnSendPayload, done: (err: [FastifyError][FastifyError] | null, res?: unknown) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L114)

你可以在 `onSend` 钩子中变更 payload。这是第六个钩子，前一个为 `preSerialization`，下一个为 `onResponse`。

注：你只能将 payload 改为 string、Buffer、stream 或 null。

##### fastify.onResponseHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], done: (err?: [FastifyError][FastifyError]) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L134)

`onResponse` 是第七个，也是最后一个钩子，前一个为 `onSend`。

该钩子在响应发出后执行，因此无法再发送更多数据了。但是你可以在此向外部服务发送数据，执行收集数据之类的任务。

##### fastify.onErrorHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(request: [FastifyRequest][FastifyRequest], reply: [FastifyReply][FastifyReply], error: [FastifyError][FastifyError], done: () => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L154)

该钩子可用于自定义错误日志，或当发生错误时添加特定的 header。

该钩子并不是为了变更错误而设计的，且调用 reply.send 会抛出一个异常。

它只会在 customErrorHandler 向用户发送错误之后被执行 (要注意的是，默认的 customErrorHandler 总是会发送错误)。

注意：与其他钩子不同，该钩子不支持向 done 函数传递错误。

##### fastify.onRouteHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [RequestGeneric][FastifyRequestGenericInterface], [ContextConfig][ContextConfigGeneric]>(opts: [RouteOptions][RouteOptions] & { path: string; prefix: string }): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L174)

当注册一个新的路由时被触发。它的监听函数拥有一个唯一的参数：routeOptions 对象。该接口是同步的，因此，监听函数不接受回调作为参数。

##### fastify.onRegisterHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [Logger][LoggerGeneric]>(instance: [FastifyInstance][FastifyInstance], done: (err?: [FastifyError][FastifyError]) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L191)

当注册一个新的插件，或创建了新的封装好的上下文后被触发。该钩子在注册的代码之前被执行。

当你的插件需要知晓上下文何时创建完毕，并操作它们时，可以使用这一钩子。

注：被 fastify-plugin 所封装的插件不会触发该钩子。

##### fastify.onCloseHookHandler<[RawServer][RawServerGeneric], [RawRequest][RawRequestGeneric], [RawReply][RawReplyGeneric], [Logger][LoggerGeneric]>(instance: [FastifyInstance][FastifyInstance], done: (err?: [FastifyError][FastifyError]) => void): Promise\<unknown\> | void

[源码](https://github.com/fastify/fastify/blob/main/types/hooks.d.ts#L206)

使用 fastify.close() 停止服务器时被触发。当插件需要一个 "shutdown" 事件时有用，例如关闭一个数据库连接。

<!-- Links -->

[Fastify]: #fastifyrawserver-rawrequest-rawreply-loggeropts-fastifyserveroptions-fastifyinstance
[RawServerGeneric]: #rawserver
[RawRequestGeneric]: #rawrequest
[RawReplyGeneric]: #rawreply
[LoggerGeneric]: #logger
[RawBodyGeneric]: #rawbody
[HTTPMethods]: #fastifyhttpmethods
[RawServerBase]: #fastifyrawserverbase
[RawServerDefault]: #fastifyrawserverdefault
[FastifyRequest]: #fastifyfastifyrequestrawserver-rawrequest-requestgeneric
[FastifyRequestGenericInterface]: #fastifyrequestgenericinterface
[RawRequestDefaultExpression]: #fastifyrawrequestdefaultexpressionrawserver
[FastifyReply]: #fastifyfastifyreplyrawserver-rawreply-contextconfig
[RawReplyDefaultExpression]: #fastifyrawreplydefaultexpression
[FastifyServerOptions]: #fastifyfastifyserveroptions-rawserver-logger
[FastifyInstance]: #fastifyfastifyinstance
[FastifyLoggerOptions]: #fastifyfastifyloggeroptions
[ContextConfigGeneric]: #ContextConfigGeneric
[FastifyPlugin]: ##fastifyfastifypluginoptions-rawserver-rawrequest-requestgeneric
[FastifyPluginCallback]: #fastifyfastifyplugincallbackoptions
[FastifyPluginAsync]: #fastifyfastifypluginasyncoptions
[FastifyPluginOptions]: #fastifyfastifypluginoptions
[FastifyRegister]: #fastifyfastifyregisterrawserver-rawrequest-requestgenericplugin-fastifyplugin-opts-fastifyregisteroptions
[FastifyRegisterOptions]: #fastifyfastifytregisteroptions
[LogLevel]: #fastifyloglevel
[FastifyError]: #fastifyfastifyerror
[RouteOptions]: #fastifyrouteoptionsrawserver-rawrequest-rawreply-requestgeneric-contextconfig