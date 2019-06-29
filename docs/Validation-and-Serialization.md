<h1 align="center">Fastify</h1>

## 验证和序列化
Fastify 使用基于 schema 的途径，从本质上将 schema 编译成了高性能的函数，来实现路由的验证与输出的序列化。我们推荐使用 [JSON Schema](http://json-schema.org/)，虽然这并非必要。

> ## ⚠  安全须知
> 应当将 schema 的定义写入代码。
> 因为不管是验证还是序列化，都会使用 `new Function()` 来动态生成代码并执行。
> 所以，用户提供的 schema 是不安全的。
> 更多内容，请看 [Ajv](http://npm.im/ajv) 与 [fast-json-stringify](http://npm.im/fast-json-stringify)。

<a name="validation"></a>
### 验证
路由的验证是依赖 [Ajv](https://www.npmjs.com/package/ajv) 实现的。这是一个高性能的 JSON schema 校验工具。验证输入十分简单，只需将字段加入路由的 schema 中即可！支持的验证类型如下：
- `body`：当请求方法为 POST 或 PUT 时，验证请求主体。
- `querystring`：验证查询字符串。可以是一个完整的 JSON Schema 对象 (包括值为 `object` 的 `type` 属性以及包含参数的 `properties` 对象)，也可以是一个只带有查询参数 (无 `type` 与 `properties` 对象) 的简单对象 (见下文示例)。
- `params`：验证路由参数。
- `headers`：验证请求头部 (request headers)。

示例：
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
  type: 'object',
  properties: {
    par1: { type: 'string' },
    par2: { type: 'number' }
  }
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
*请注意，Ajv 会尝试将数据[隐式转换](https://github.com/epoberezkin/ajv#coercing-data-types)为 schema 中 `type` 属性指明的类型。这么做的目的是通过校验，并在后续过程中使用正确类型的数据。*

<a name="shared-schema"></a>
#### 添加共用 schema
感谢 `addSchema` API，它让你可以向 Fastify 实例添加多个 schema，并在你程序的不同部分使用它们。该 API 也是封装好的。

有两种方式可以复用你的共用 shema：
+ **`使用$ref`**：正如 [standard](https://tools.ietf.org/html/draft-handrews-json-schema-01#section-8) 中所述，你可以引用一份外部的 schema。做法是在 `addSchema` 的 `$id` 参数中指明外部 schema 的绝对 URI。
+ **`替换方式`**：Fastify 允许你使用共用 schema 替换某些字段。
你只需指明 `addSchema` 中的 `$id` 为相对 URI 的 fragment (译注：URI fragment是 URI 中 `#` 号后的部分) 即可，fragment 只接受字母与数字的组合`[A-Za-z0-9]`。

以下展示了你可以 _如何_ 设置 `$id` 以及 _如何_ 引用它：

+ `替换方式`
  + `myField: 'foobar#'` 会搜寻带 `$id: 'foobar'` 的共用 schema
+ `使用$ref`
  + `myField: { $ref: '#foo'}` 会在当前 schema 内搜寻带 `$id: '#foo'` 的字段
  + `myField: { $ref: '#/definitions/foo'}` 会在当前 schema 内搜寻 `definitions.foo` 字段
  + `myField: { $ref: 'http://url.com/sh.json#'}` 会搜寻带 `$id: 'http://url.com/sh.json'` 的共用 schema
  + `myField: { $ref: 'http://url.com/sh.json#/definitions/foo'}` 会搜寻带 `$id: 'http://url.com/sh.json'` 的共用 schema，并使用其 `definitions.foo` 字段
  + `myField: { $ref: 'http://url.com/sh.json#foo'}` 会搜寻带 `$id: 'http://url.com/sh.json'` 的共用 schema，并使用其内部带 `$id: '#foo'` 的对象


更多例子：

**`使用$ref`** 的例子：

```js
fastify.addSchema({
  $id: 'http://example.com/common.json',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: {
      type: 'array',
      items: { $ref: 'http://example.com/common.json#/properties/hello' }
    }
  },
  handler: () => {}
})
```

**`替换方式`** 的例子：

```js
const fastify = require('fastify')()

fastify.addSchema({
  $id: 'greetings',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: 'greetings#'
  },
  handler: () => {}
})

fastify.register((instance, opts, next) => {
  /**
  * 你可以在子作用域中使用在上层作用域里定义的 scheme，比如 'greetings'。
  * 父级作用域则无法使用子作用域定义的 schema。
  */
  instance.addSchema({
    $id: 'framework',
    type: 'object',
    properties: {
      fastest: { type: 'string' },
      hi: 'greetings#'
    }
  })
  instance.route({
    method: 'POST',
    url: '/sub',
    schema: {
      body: 'framework#'
    },
    handler: () => {}
  })
  next()
})
```

在任意位置你都能使用共用 schema，无论是在应用顶层，还是在其他 schema 的内部：
```js
const fastify = require('fastify')()

fastify.addSchema({
  $id: 'greetings',
  type: 'object',
  properties: {
    hello: { type: 'string' }
  }
})

fastify.route({
  method: 'POST',
  url: '/',
  schema: {
    body: {
      type: 'object',
      properties: {
        greeting: 'greetings#',
        timestamp: { type: 'number' }
      }
    }
  },
  handler: () => {}
})
```

<a name="get-shared-schema"></a>
#### 获取共用 schema 的拷贝

`getSchemas` 函数返回指定作用域中的共用 schema：
```js
fastify.addSchema({ $id: 'one', my: 'hello' })
fastify.get('/', (request, reply) => { reply.send(fastify.getSchemas()) })

fastify.register((instance, opts, next) => {
  instance.addSchema({ $id: 'two', my: 'ciao' })
  instance.get('/sub', (request, reply) => { reply.send(instance.getSchemas()) })

  instance.register((subinstance, opts, next) => {
    subinstance.addSchema({ $id: 'three', my: 'hola' })
    subinstance.get('/deep', (request, reply) => { reply.send(subinstance.getSchemas()) })
    next()
  })
  next()
})
```
这个例子的输出如下：

|  URL  | Schemas |
|-------|---------|
| /     | one             |
| /sub  | one, two        |
| /deep | one, two, three |

<a name="schema-compiler"></a>
#### Schema 编译器

`schemaCompiler` 返回一个用于验证请求主体、url 参数、header 以及查询字符串的函数。默认情况下，它返回一个实现了 [ajv](https://ajv.js.org/) 验证接口的函数。Fastify 使用它对验证进行加速。

Fastify 使用的 `ajv` 基本配置如下：

```js
{
  removeAdditional: true, // 移除额外属性
  useDefaults: true, // 当属性或项目缺失时，使用 schema 中预先定义好的 default 的值代替
  coerceTypes: true, // 根据定义的 type 的值改变数据类型
  allErrors: true,   // 检查出所有错误（译注：为 false 时出现首个错误后即返回）
  nullable: true     // 支持 OpenAPI Specification 3.0 版本的 "nullable" 关键字
}
```

上述配置无法被修改。假如你想改变或新增配置选项，你需要创建一个自定义的实例，并覆盖已存在的实例：

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
fastify.setSchemaCompiler(function (schema) {
  return ajv.compile(schema)
})

// -------
// 此外，你还可以通过 setter 方法来设置 schema 编译器：
fastify.schemaCompiler = function (schema) { return ajv.compile(schema) })
```

你可能还想使用其他的验证库，例如 `Joi`。这时，你只要参照下面的示例，便能轻松地使用它们来校验 url 参数、请求主体与查询字符串了！

```js
const Joi = require('joi')

fastify.post('/the/url', {
  schema: {
    body: Joi.object().keys({
      hello: Joi.string().required()
    }).required()
  },
  schemaCompiler: schema => data => Joi.validate(data, schema)
}, handler)
```

在上面的例子中，`schemaCompiler` 函数返回一个包含以下属性的对象：
* `error`：一个 `Error` 实例，或一个描述验证错误信息的字符串
* `value`：通过了验证的数据

<a name="serialization"></a>
### 序列化
通常，你会通过 JSON 格式将数据发送至客户端。鉴于此，Fastify 提供了一个强大的工具——[fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) 来帮助你。当你提供了输出的 schema 时，它能派上用场。我们推荐你编写一个输出的 schema，因为这能让应用的吞吐量提升 100-400% (根据 payload 的不同而有所变化)，也能防止敏感信息的意外泄露。

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
      type: 'object',
      properties: {
        value: { type: 'string' }
      }
    }
  }
}

fastify.post('/the/url', { schema }, handler)
```

*假如你需要在特定位置使用自定义的序列化工具，你可以使用 `reply.serializer(...)`。*

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

如果你想在路由内部控制错误，可以设置 `attachValidation` 选项。当出现验证错误时，请求的 `validationError` 属性将会包含一个 `Error` 对象，在这对象内部有原始的验证结果 `validation`，如下所示：
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
     reply.status(422).send(new Error('validation failed'))
  }
})
```

### JSON Schema 及共用 Schema (Shared Schema) 支持

为了能更简单地重用 schema，JSON Schema 提供了一些功能，来结合 Fastify 的共用 schema。

| 用例                          | 验证器 | 序列化器 |
|-----------------------------------|-----------|------------|
| 共用 schema                     | ✔️ | ✔️ |
| 引用 (`$ref`) `$id`                   | ✔ | ✔️ |
| 引用 (`$ref`) `/definitions`          | ✔️ | ✔️ |
| 引用 (`$ref`) 共用 schema `$id`          | ✔ | ✔️ |
| 引用 (`$ref`) 共用 schema `/definitions` | ✔ | ✔️ |

#### 示例

```js
// 共用 Schema 的用例
fastify.addSchema({
  $id: 'sharedAddress',
  type: 'object',
  properties: {
    city: { 'type': 'string' }
  }
})

const sharedSchema = {
  type: 'object',
  properties: {
    home: 'sharedAddress#',
    work: 'sharedAddress#'
  }
}
```

```js
// 同一 JSON Schema 内部对 $id 的引用 ($ref)
const refToId = {
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { 'type': 'string' }
      }
    }
  },
  properties: {
    home: { $ref: '#address' },
    work: { $ref: '#address' }
  }
}
```


```js
// 同一 JSON Schema 内部对 /definitions 的引用 ($ref)
const refToDefinitions = {
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { 'type': 'string' }
      }
    }
  },
  properties: {
    home: { $ref: '#/definitions/foo' },
    work: { $ref: '#/definitions/foo' }
  }
}
```

```js
// 对外部共用 schema 的 $id 的引用 ($ref)
fastify.addSchema({
  $id: 'http://foo/common.json',
  type: 'object',
  definitions: {
    foo: {
      $id: '#address',
      type: 'object',
      properties: {
        city: { 'type': 'string' }
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


```js
// 对外部共用 schema 的 /definitions 的引用 ($ref)
fastify.addSchema({
  $id: 'http://foo/common.json',
  type: 'object',
  definitions: {
    foo: {
      type: 'object',
      properties: {
        city: { 'type': 'string' }
      }
    }
  }
})

const refToSharedSchemaDefinitions = {
  type: 'object',
  properties: {
    home: { $ref: 'http://foo/common.json#/definitions/foo' },
    work: { $ref: 'http://foo/common.json#/definitions/foo' }
  }
}
```

<a name="resources"></a>
### 资源
- [JSON Schema](http://json-schema.org/)
- [理解 JSON schema](https://spacetelescope.github.io/understanding-json-schema/)
- [fast-json-stringify 文档](https://github.com/fastify/fast-json-stringify)
- [Ajv 文档](https://github.com/epoberezkin/ajv/blob/master/README.md)
