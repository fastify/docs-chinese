# 迁移到 V3

本文帮助你从 Fastify v2 迁移到 v3。

开始之前，请确保所有 v2 中不推荐用法的警示都已修复。v2 的这些特性在新版都已被移除，升级后你将不能使用它们。([#1750](https://github.com/fastify/fastify/pull/1750))

## 重大改动

### 中间件支持 ([#2014](https://github.com/fastify/fastify/pull/2014))

从 Fastify v3 开始，框架本身便不再支持中间件功能了。

要使用 Express 的中间件的话，请安装 [`fastify-express`](https://github.com/fastify/fastify-express) 或 [`middie`](https://github.com/fastify/middie)。

**v2:**

```js
// 在 Fastify v2 中使用 Express 的 `cors` 中间件。
fastify.use(require('cors')());
```

**v3:**

```js
// 在 Fastify v3 中使用 Express 的 `cors` 中间件。
await fastify.register(require('fastify-express'));
fastify.use(require('cors')());
```

### 日志序列化 ([#2017](https://github.com/fastify/fastify/pull/2017))

日志的[序列化器](Logging.md)得到了升级，现在它接受 Fastify 的 [`Request`](Request.md) 和 [`Reply`](Reply.md) 对象，而非原生的对象。

任何依赖于原生对象而非 Fastify 对象上的 `request` 或 `reply` 属性的自定义的序列化器，都应当升级。

**v2:**

```js
const fastify = require('fastify')({
  logger: {
    serializers: {
      res(res) {
        return {
          statusCode: res.statusCode,
          customProp: res.customProp
        };
      }
    }
  }
});
```

**v3:**

```js
const fastify = require('fastify')({
  logger: {
    serializers: {
      res(reply) {
        return {
          statusCode: reply.statusCode, // 无需更改
          customProp: reply.raw.customProp // 从 res 对象 (译注：即 Node.js 原生的响应对象，此处为 raw) 中记录属性
        };
      }
    }
  }
});
```

### schema 代入 (schema substitution) ([#2023](https://github.com/fastify/fastify/pull/2023))

非标准`替换方式`的支持被移除了，取而代之的是符合 JSON Schema 标准的 `$ref` 方案。要更好地理解这一改变，请阅读[《Fastify v3 的验证与序列化》](https://dev.to/eomm/validation-and-serialization-in-fastify-v3-2e8l)一文。

**v2:**

```js
const schema = {
  body: 'schemaId#'
};
fastify.route({ method, url, schema, handler });
```

**v3:**

```js
const schema = {
  body: {
    $ref: 'schemaId#'
  }
};
fastify.route({ method, url, schema, handler });
```

### schema 验证选项 ([#2023](https://github.com/fastify/fastify/pull/2023))

为了未来工具的改善，`setSchemaCompiler` 和 `setSchemaResolver` 被替换成了 `setValidatorCompiler`。要更好地理解这一改变，请阅读[《Fastify v3 的验证与序列化》](https://dev.to/eomm/validation-and-serialization-in-fastify-v3-2e8l)一文。

**v2:**

```js
const fastify = Fastify();
const ajv = new AJV();
ajv.addSchema(schemaA);
ajv.addSchema(schemaB);

fastify.setSchemaCompiler(schema => ajv.compile(schema));
fastify.setSchemaResolver(ref => ajv.getSchema(ref).schema);
```

**v3:**

```js
const fastify = Fastify();
const ajv = new AJV();
ajv.addSchema(schemaA);
ajv.addSchema(schemaB);

fastify.setValidatorCompiler(({ schema, method, url, httpPart }) =>
  ajv.compile(schema)
);
```

### preParsing 钩子的行为 ([#2286](https://github.com/fastify/fastify/pull/2286))

为了支持针对请求 payload 的操作，从 Fastify v3 开始，`preParsing` 钩子的行为发生了微小的变化。

该钩子现在有一个额外的参数，`payload`，因此，新的函数签名是 `fn(request, reply, payload, done)` 或 `async fn(request, reply, payload)`。

钩子可以通过 `done(null, stream)` 回调，或 async 函数返回一个 stream。

如果返回了新的 stream，那么在之后的钩子里，这个新的 stream 将代替原有的 stream。这种做法的一个用途是处理经过压缩的请求。

新的 stream 还应当添加 `receivedEncodedLength` 属性，来反映客户端数据的真实大小。举例而言，假如请求被压缩了，那么该属性便是压缩后的 payload 的大小。该属性可以 (且应当) 在 `data` 事件中动态更新。

原先 Fastify v2 的语法仍然受支持，但不推荐使用。

### 钩子的行为 ([#2004](https://github.com/fastify/fastify/pull/2004))

为了支持钩子的封装，从 Fastify v3 开始，`onRoute` 与 `onRegister` 钩子的行为发生了微小的变化。

- `onRoute` - 现在改为了异步调用，且在同一个封装的作用域内会继承。因此，你得在注册其他插件 _之前_ 注册该钩子。
- `onRegister` - 和 onRoute 一样。唯一区别在于现在第一次的调用者不是框架本身了，而是首个注册的插件。

### Content Type 解析器的语法 ([#2286](https://github.com/fastify/fastify/pull/2286))

在 Fastify v3 中，Content Type 解析器现在有了单一函数签名。

新的签名是 `fn(request, payload, done)` 或 `async fn(request, payload)`。注意现在 `request` 是 Fastify 的请求对象，而不是 `IncomingMessage` 了。默认情况下，payload 是一个 stream。如果在 `addContentTypeParser` 中使用了 `parseAs` 选项，那么 `payload` 会被当做该选项的值来对待 (string 或 buffer)。

原先的函数签名 `fn(req, [done])` 或 `fn(req, payload, [done])` (这里 `req` 是 `IncomingMessage`) 仍然受支持，但不推荐使用。

### TypeScript 支持

版本 3 的类型系统发生了改变。新的系统带来了泛型约束 (generic constraining) 与默认值，以及定义请求 body，querystring 等 schema 的新方式！

**v2:**

```ts
interface PingQuerystring {
  foo?: number;
}

interface PingParams {
  bar?: string;
}

interface PingHeaders {
  a?: string;
}

interface PingBody {
  baz?: string;
}

server.get<PingQuerystring, PingParams, PingHeaders, PingBody>(
  '/ping/:bar',
  opts,
  (request, reply) => {
    console.log(request.query); // 类型为 `PingQuerystring`
    console.log(request.params); // 类型为 `PingParams`
    console.log(request.headers); // 类型为 `PingHeaders`
    console.log(request.body); // 类型为 `PingBody`
  }
);
```

**v3:**

```ts
server.get<{
  Querystring: PingQuerystring;
  Params: PingParams;
  Headers: PingHeaders;
  Body: PingBody;
}>('/ping/:bar', opts, async (request, reply) => {
  console.log(request.query); // 类型为 `PingQuerystring`
  console.log(request.params); // 类型为 `PingParams`
  console.log(request.headers); // 类型为 `PingHeaders`
  console.log(request.body); // 类型为 `PingBody`
});
```

### 管理未捕获异常 ([#2073](https://github.com/fastify/fastify/pull/2073))

在同步的路由方法里，一旦一个错误被抛出，服务器将会如设定的那样崩溃，且不调用 `.setErrorHandler()` 方法。现在这一行为发生了改变，所有在同步或异步的路由中未捕获的异常，都将得到控制。

**v2:**

```js
fastify.setErrorHandler((error, request, reply) => {
  // 不会调用
  reply.send(error)
})
fastify.get('/', (request, reply) => {
  const maybeAnArray = request.body.something ? [] : 'I am a string'
  maybeAnArray.substr() // 抛错：[].substr is not a function，同时服务器崩溃
})
```

**v3:**

```js
fastify.setErrorHandler((error, request, reply) => {
  // 会调用
  reply.send(error)
})
fastify.get('/', (request, reply) => {
  const maybeAnArray = request.body.something ? [] : 'I am a string'
  maybeAnArray.substr() // 抛错：[].substr is not a function，但错误得到控制
})
```

## 更多特性与改善

- 不管如何注册钩子，它们总拥有一致的上下文 ([#2005](https://github.com/fastify/fastify/pull/2005))
- 推荐使用 [`request.raw`](Request.md) 及 [`reply.raw`](Reply.md)，而非 `request.req` 和 `reply.res` ([#2008](https://github.com/fastify/fastify/pull/2008))
- 移除 `modifyCoreObjects` 选项 ([#2015](https://github.com/fastify/fastify/pull/2015))
- 添加 [`connectionTimeout`](Server.md#factory-connection-timeout) 选项 ([#2086](https://github.com/fastify/fastify/pull/2086))
- 添加 [`keepAliveTimeout`](Server.md#factory-keep-alive-timeout) 选项 ([#2086](https://github.com/fastify/fastify/pull/2086))
- [插件](Plugins.md#async-await)支持 async-await ([#2093](https://github.com/fastify/fastify/pull/2093))
- 支持将对象作为错误抛出 ([#2134](https://github.com/fastify/fastify/pull/2134))