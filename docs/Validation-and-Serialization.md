<h1 align="center">Fastify</h1>

## 验证和序列化
Fastify 使用基于 schema 的途径，从本质上将 schema 编译成了高性能的函数，来实现路由的验证与输出的序列化。我们推荐使用 [JSON Schema](http://json-schema.org/)，虽然这并非必要。

> ## ⚠  安全须知
> 应当将 schema 的定义写入代码。
> 因为不管是验证还是序列化，都会使用 `new Function()` 来动态生成代码并执行。
> 所以，用户提供的 schema 是不安全的。
> 更多内容，请看 [Ajv](http://npm.im/ajv) 与 [fast-json-stringify](http://npm.im/fast-json-stringify)。

### 核心观念
验证与序列化的任务分别由两个可定制的工具完成：
- [Ajv](https://www.npmjs.com/package/ajv) 用于验证请求。
- [fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) 用于序列化响应的 body。

这些工具相互独立，但共享通过 `.addSchema(schema)` 方法添加到 Fastify 实例上的 JSON schema。

<a name="shared-schema"></a>
#### 添加共用 schema (shared schema)
得益于 `addSchema` API，你能向 Fastify 实例添加多个 schema，并在程序的不同部分复用它们。
像往常一样，该 API 是封装好的。

共用 schema 可以通过 JSON Schema 的 [**`$ref`**](https://tools.ietf.org/html/draft-handrews-json-schema-01#section-8) 关键字复用。
以下是引用方法的 _总结_：

+ `myField: { $ref: '#foo'}` 将在当前 schema 内搜索 `$id: '#foo'` 字段。
+ `myField: { $ref: '#/definitions/foo'}` 将在当前 schema 内搜索 `definitions.foo` 字段。
+ `myField: { $ref: 'http://url.com/sh.json#'}` 会搜索含 `$id: 'http://url.com/sh.json'` 的共用 schema。
+ `myField: { $ref: 'http://url.com/sh.json#/definitions/foo'}` 会搜索含 `$id: 'http://url.com/sh.json'` 的共用 schema，并使用其 `definitions.foo` 字段。
+ `myField: { $ref: 'http://url.com/sh.json#foo'}` 会搜索含 `$id: 'http://url.com/sh.json'` 的共用 schema，并使用其内部带 `$id: '#foo'` 的对象。


**简单用法：**

```js
fastify.addSchema({
  $id: 'http://example.com/',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.post('/', {
  handler () {},
  schema: {
    body: {
      type: 'array',
      items: { $ref: 'http://example.com#/properties/hello' }
    }
  }
})
```

**`$ref` 作为根引用 (root reference)：**

```js
fastify.addSchema({
  $id: 'commonSchema',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.post('/', {
  handler () {},
  schema: {
    body: { $ref: 'commonSchema#' },
    headers: { $ref: 'commonSchema#' }
  }
})
```

<a name="get-shared-schema"></a>
#### 获取共用 schema

当自定义验证器或序列化器的时候，Fastify 不再能控制它们，此时 `.addSchema` 方法失去了作用。
因此，要获取添加到 Fastify 实例上的 schema，你可以使用 `.getSchemas()`：

```js
fastify.addSchema({
  $id: 'schemaId',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

const mySchemas = fastify.getSchemas()
const mySchema = fastify.getSchema('schemaId')
```

`getSchemas` 方法也是封装好的，返回的是指定作用域中可用的共用 schema：

```js
fastify.addSchema({ $id: 'one', my: 'hello' })
// 只返回 schema `one`
fastify.get('/', (request, reply) => { reply.send(fastify.getSchemas()) }) 

fastify.register((instance, opts, done) => {
  instance.addSchema({ $id: 'two', my: 'ciao' })
  // 会返回 schema `one` 与 `two`
  instance.get('/sub', (request, reply) => { reply.send(instance.getSchemas()) })

  instance.register((subinstance, opts, done) => {
    subinstance.addSchema({ $id: 'three', my: 'hola' })
    // 会返回 schema `one`、`two` 和 `three`
    subinstance.get('/deep', (request, reply) => { reply.send(subinstance.getSchemas()) })
    done()
  })
  done()
})
```

### 验证
路由的验证是依赖 [Ajv](https://www.npmjs.com/package/ajv) 实现的。这是一个高性能的 JSON schema 校验工具。验证输入十分简单，只需将字段加入路由的 schema 中即可！

支持的验证类型如下：
- `body`：当请求方法为 POST 或 PUT 时，验证 body。
- `querystring` 或 `query`：验证 querystring。
- `params`：验证路由参数。
- `headers`：验证 header。

所有的验证都可以是一个完整的 JSON Schema 对象 (包括值为 `object` 的 `type` 属性以及包含参数的 `properties` 对象)，也可以是一个没有 `type` 与 `properties`，而仅仅在顶层列明参数的简单变种 (见下文示例)。

Example:
```js
const bodyJsonSchema = {
  type: 'object',
  required: ['requiredKey'],
  properties: {
    someKey: { type: 'string' },
    someOtherKey: { type: 'number' },
    requiredKey: {
      type: 'array',
      maxItems: 3,
      items: { type: 'integer' }
    },
    nullableKey: { type: ['number', 'null'] }, // 或 { type: 'number', nullable: true }
    multipleTypesKey: { type: ['boolean', 'number'] },
    multipleRestrictedTypesKey: {
      oneOf: [
        { type: 'string', maxLength: 5 },
        { type: 'number', minimum: 10 }
      ]
    },
    enumKey: {
      type: 'string',
      enum: ['John', 'Foo']
    },
    notTypeKey: {
      not: { type: 'array' }
    }
  }
}

const queryStringJsonSchema = {
  name: { type: 'string' },
  excitement: { type: 'integer' }
}

const paramsJsonSchema = {
  par1: { type: 'string' },
  par2: { type: 'number' }
}

const headersJsonSchema = {
  type: 'object',
  properties: {
    'x-foo': { type: 'string' }
  },
  required: ['x-foo']
}

const schema = {
  body: bodyJsonSchema,
  querystring: queryStringJsonSchema,
  params: paramsJsonSchema,
  headers: headersJsonSchema
}

fastify.post('/the/url', { schema }, handler)
```

*请注意，为了通过校验，并在后续过程中使用正确类型的数据，Ajv 会尝试将数据[隐式转换](https://github.com/epoberezkin/ajv#coercing-data-types)为 schema 中 `type` 属性指明的类型。*

<a name="ajv-plugins"></a>
#### Ajv 插件

你可以提供一组用于 Ajv 的插件：

> 插件格式参见 [`ajv 选项`](https://github.com/fastify/docs-chinese/blob/master/docs/Server.md#factory-ajv)

```js
const fastify = require('fastify')({
  ajv: {
    plugins: [
      require('ajv-merge-patch')
    ]
  }
})

fastify.post('/', {
  handler (req, reply) { reply.send({ ok: 1 }) },
  schema: {
    body: {
      $patch: {
        source: {
          type: 'object',
          properties: {
            q: {
              type: 'string'
            }
          }
        },
        with: [
          {
            op: 'add',
            path: '/properties/q',
            value: { type: 'number' }
          }
        ]
      }
    }
  }
})

fastify.post('/foo', {
  handler (req, reply) { reply.send({ ok: 1 }) },
  schema: {
    body: {
      $merge: {
        source: {
          type: 'object',
          properties: {
            q: {
              type: 'string'
            }
          }
        },
        with: {
          required: ['q']
        }
      }
    }
  }
})
```

<a name="schema-validator"></a>
#### 验证生成器

`validatorCompiler` 返回一个用于验证 body、url、路由参数、header 以及 querystring 的函数。默认返回一个实现了 [ajv](https://ajv.js.org/) 验证接口的函数。Fastify 内在地使用该函数以加速验证。

Fastify 使用的 [ajv 基本配置](https://github.com/epoberezkin/ajv#options-to-modify-validated-data)如下：

```js
{
  removeAdditional: true, // 移除额外属性
  useDefaults: true, // 当属性或项目缺失时，使用 schema 中预先定义好的 default 的值代替
  coerceTypes: true, // 根据定义的 type 的值改变数据类型
  allErrors: true,   // 检查出所有错误（译注：为 false 时出现首个错误后即返回）
  nullable: true     // 支持 OpenAPI Specification 3.0 版本的 "nullable" 关键字
}
```

上述配置可通过 [`ajv.customOptions`](https://github.com/fastify/docs-chinese/blob/master/docs/Server.md#factory-ajv) 修改。

假如你想改变或增加额外的选项，你需要创建一个自定义的实例，并覆盖已存在的实例：

```js
const fastify = require('fastify')()
const Ajv = require('ajv')
const ajv = new Ajv({
  // fastify 使用的默认参数（如果需要）
  removeAdditional: true,
  useDefaults: true,
  coerceTypes: true,
  allErrors: true,
  nullable: true,
  // 任意其他参数
  // ...
})
fastify.setValidatorCompiler((method, url, httpPart, schema) => {
  return ajv.compile(schema)
})
```

也许你想使用其他验证工具，例如 `Joi`。下面的例子展示了如何通过 `Joi` 来验证 url、参数、body 与 querystring！
<a name="using-other-validation-libraries"></a>
##### 使用其他验证工具

通过 `setValidatorCompiler` 函数，你可以轻松地将 `ajv` 替换为几乎任意的 Javascript 验证工具 (如 [joi](https://github.com/hapijs/joi/)、[yup](https://github.com/jquense/yup/) 等)，或自定义它们。	

```js
const Joi = require('@hapi/joi')

fastify.post('/the/url', {
  schema: {
    body: Joi.object().keys({
      hello: Joi.string().required()
    }).required()
  },
  validatorCompiler: (method, url, httpPart, schema) => {
    return (data) => Joi.validate(data, schema)
  }
}, handler)
```

```js
const yup = require('yup')
// 等同于前文 ajv 基本配置的 yup 的配置
const yupOptions = {
  strict: false,
  abortEarly: false, // 返回所有错误（译注：为 true 时出现首个错误后即返回）
  stripUnknown: true, // 移除额外属性
  recursive: true
}
fastify.post('/the/url', {
  schema: {
    body: yup.object({
      age: yup.number().integer().required(),
      sub: yup.object().shape({
        name: yup.string().required()
      }).required()
    })
  },
  validatorCompiler: (method, url, httpPart, schema) => {
    return function (data) {
      // 当设置 strict = false 时， yup 的 `validateSync` 函数在验证成功后会返回经过转换的值，而失败时则会抛错。
      try {
        const result = schema.validateSync(data, yupOptions)
        return { value: result }
      } catch (e) {
        return { error: e }
      }
    }
  }
}, handler)
```

##### 其他验证工具的验证信息

Fastify 的错误验证与其默认的验证引擎 `ajv` 紧密结合，错误最终会经由 `schemaErrorsText` 函数转化为便于阅读的信息。然而，也正是由于 `schemaErrorsText` 与 `ajv` 的强关联性，当你使用其他校验工具时，可能会出现奇怪或不完整的错误信息。

要规避以上问题，主要有两个途径：

1. 确保自定义的 `schemaCompiler` 返回的错误结构与 `ajv` 的一致 (当然，由于各引擎的差异，这是件困难的活儿)。	
2. 使用自定义的 `errorHandler` 拦截并格式化验证错误。

Fastify 给所有的验证错误添加了两个属性，来帮助你自定义 `errorHandler`：

* validation：来自 `schemaCompiler` 函数的验证函数所返回的对象上的 `error` 属性的内容。	
* validationContext：验证错误的上下文 (body、params、query、headers)。

以下是一个自定义 `errorHandler` 来处理验证错误的例子：

```js
const errorHandler = (error, request, reply) => {
  const statusCode = error.statusCode
  let response

  const { validation, validationContext } = error

  // 检验是否发生了验证错误
  if (validation) {
    response = {
      // validationContext 的值可能是 'body'、'params'、'headers' 或 'query'
      message: `A validation error occured when validating the ${validationContext}...`,
     // 验证工具返回的结果
      errors: validation
    }
  } else {
    response = {
      message: 'An error occurred...'
    }
  }

  // 其余代码。例如，记录错误日志。	
  // ...

  reply.status(statusCode).send(response)
}
```

<a name="serialization"></a>
### 序列化
通常，你会通过 JSON 格式将数据发送至客户端。鉴于此，Fastify 提供了一个强大的工具——[fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) 来帮助你。当你在路由选项中提供了输出的 schema 时，它能派上用场。
我们推荐你编写一个输出的 schema，因为这能让应用的吞吐量提升 100-400% (根据 payload 的不同而有所变化)，也能防止敏感信息的意外泄露。

示例：
```js
const schema = {
  response: {
    200: {
      type: 'object',
      properties: {
        value: { type: 'string' },
        otherValue: { type: 'boolean' }
      }
    }
  }
}

fastify.post('/the/url', { schema }, handler)
```

如你所见，响应的 schema 是建立在状态码的基础之上的。当你想对多个状态码使用同一个 schema 时，你可以使用类似 `'2xx'` 的表达方法，例如：
```js
const schema = {
  response: {
    '2xx': {
      type: 'object',
      properties: {
        value: { type: 'string' },
        otherValue: { type: 'boolean' }
      }
    },
    201: {
      // 对比写法
      value: { type: 'string' }
    }
  }
}

fastify.post('/the/url', { schema }, handler)
```

<a name="schema-serializer"></a>
#### 序列化函数生成器

`serializerCompiler` 返回一个根据输入参数返回字符串的函数。你应该提供一个函数，用于序列化所有定义了 `response` JSON Schema 的路由。

```js
fastify.setSerializerCompiler((method, url, httpPart, schema) => {
  return data => JSON.stringify(data)
})

fastify.get('/user', {
  handler (req, reply) {
    reply.send({ id: 1, name: 'Foo', image: 'BIG IMAGE' })
  },
  schema: {
    response: {
      '2xx': {
        id: { type: 'number' },
        name: { type: 'string' }
      }
    }
  }
})
```

*假如你需要在特定位置使用自定义的序列化工具，你可以使用 [`reply.serializer(...)`](https://github.com/fastify/docs-chinese/blob/master/docs/Reply.md#serializerfunc)。*

### 错误控制
当某个请求 schema 校验失败时，Fastify 会自动返回一个包含校验结果的 400 响应。举例来说，假如你的路由有一个如下的 schema：
 ```js
const schema = {
  body: {
    type: 'object',
    properties: {
      name: { type: 'string' }
    },
    required: ['name']
  }
}
```
当校验失败时，路由会立即返回一个包含以下内容的响应：
 ```js
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "body should have required property 'name'"
}
```

如果你想在路由内部控制错误，可以设置 `attachValidation` 选项。当出现 _验证错误_ 时，请求的 `validationError` 属性将会包含一个 `Error` 对象，在这对象内部有原始的验证结果 `validation`，如下所示：
 ```js
const fastify = Fastify()
 fastify.post('/', { schema, attachValidation: true }, function (req, reply) {
  if (req.validationError) {
    // `req.validationError.validation` 包含了原始的验证错误信息
    reply.code(400).send(req.validationError)
  }
})
```

你还可以使用 [setErrorHandler](https://www.fastify.io/docs/latest/Server/#seterrorhandler) 方法来自定义一个校验错误响应，如下：
 ```js
fastify.setErrorHandler(function (error, request, reply) {
  if (error.validation) {
     // error.validationContext 是 [body, params, querystring, headers] 之中的值
     reply.status(422).send(new Error(`validation failed of the ${error.validationContext}`))
  }
})
```

假如你想轻松愉快地自定义错误响应，可以看[这里](https://github.com/epoberezkin/ajv-errors)。

### JSON Schema 支持

为了能更简单地重用 schema，JSON Schema 提供了一些功能，来结合 Fastify 的共用 schema。

| 用例                          | 验证器 | 序列化器 |
|-----------------------------------|-----------|------------|
| 引用 (`$ref`) `$id`                   | ✔ | ✔️ |
| 引用 (`$ref`) `/definitions`          | ✔️ | ✔️ |
| 引用 (`$ref`) 共用 schema `$id`          | ✔ | ✔️ |
| 引用 (`$ref`) 共用 schema `/definitions` | ✔ | ✔️ |

#### 示例

##### 同一个 JSON Schema 中对 `$id` 的引用 ($ref)

```js
const refToId = {
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { type: 'string' }
      }
    }
  },
  properties: {
    home: { $ref: '#address' },
    work: { $ref: '#address' }
  }
}
```

##### 同一个 JSON Schema 中对 `/definitions` 的引用 ($ref)
```js
const refToDefinitions = {
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { type: 'string' }
      }
    }
  },
  properties: {
    home: { $ref: '#/definitions/foo' },
    work: { $ref: '#/definitions/foo' }
  }
}
```

##### 对外部共用 schema 的 `$id` 的引用 ($ref)
```js
fastify.addSchema({
  $id: 'http://foo/common.json',
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { type: 'string' }
      }
    }
  }
})

const refToSharedSchemaId = {
  type: 'object',
  properties: {
    home: { $ref: 'http://foo/common.json#address' },
    work: { $ref: 'http://foo/common.json#address' }
  }
}
```

##### 对外部共用 schema 的 `/definitions` 的引用 ($ref)
```js
fastify.addSchema({
  $id: 'http://foo/shared.json',
  type: 'object',
  definitions: {
    foo: {
      type: 'object',
      properties: {
        city: { type: 'string' }
      }
    }
  }
})

const refToSharedSchemaDefinitions = {
  type: 'object',
  properties: {
    home: { $ref: 'http://foo/shared.json#/definitions/foo' },
    work: { $ref: 'http://foo/shared.json#/definitions/foo' }
  }
}
```

<a name="resources"></a>
### 资源
- [JSON Schema](http://json-schema.org/)
- [理解 JSON Schema](https://spacetelescope.github.io/understanding-json-schema/)
- [fast-json-stringify 文档](https://github.com/fastify/fast-json-stringify)
- [Ajv 文档](https://github.com/epoberezkin/ajv/blob/master/README.md)
- [Ajv i18n](https://github.com/epoberezkin/ajv-i18n)
- [Ajv 自定义错误](https://github.com/epoberezkin/ajv-errors)
- 使用核心方法自定义错误处理，并实现错误文件转储的[例子](https://github.com/fastify/example/tree/master/validation-messages)