<h1 align="center">Fastify</h1>

## Fluent Schema

在[验证和序列化](https://github.com/fastify/docs-chinese/blob/master/docs/Validation-and-Serialization.md)一文中，我们列明了使用 JSON schema 验证输入、优化输出时所有可用的参数。

现在，你可以使用 [`fluent-schema`][fluent-schema-repo] 来更简单地设置 JSON schema，并且复用常量。

### 基本设置

```js
const S = require('fluent-schema')

// 你可以使用如下的一个对象，或查询数据库来获取数据
const MY_KEY = {
  KEY1: 'ONE',
  KEY2: 'TWO'
}

const bodyJsonSchema = S.object()
  .prop('someKey', S.string())
  .prop('someOtherKey', S.number())
  .prop('requiredKey', S.array().maxItems(3).items(S.integer()).required())
  .prop('nullableKey', S.mixed([S.TYPES.NUMBER, S.TYPES.NULL]))
  .prop('multipleTypesKey', S.mixed([S.TYPES.BOOLEAN, S.TYPES.NUMBER]))
  .prop('multipleRestrictedTypesKey', S.oneOf([S.string().maxLength(5), S.number().minimum(10)]))
  .prop('enumKey', S.enum(Object.values(MY_KEYS)))
  .prop('notTypeKey', S.not(S.array()))

const queryStringJsonSchema = S.object()
  .prop('name', S.string())
  .prop('excitement', S.integer())

const paramsJsonSchema = S.object()
  .prop('par1', S.string())
  .prop('par2', S.integer())

const headersJsonSchema = S.object()
  .prop('x-foo', S.string().required())

const schema = {
  body: bodyJsonSchema.valueOf(),
  querystring: queryStringJsonSchema.valueOf(),
  params: paramsJsonSchema.valueOf(),
  headers: headersJsonSchema.valueOf()
}

fastify.post('/the/url', { schema }, handler)
```

### 复用

使用 `fluent-schema`，你可以简单且程序化地处理 schema，并通过 `addSchema()` 来复用它们。
正如[验证和序列化](./Validation-and-Serialization.md#adding-a-shared-schema)一文所述，有两种方法来引用 schema。

以下是一些例子：

**`使用$ref`**：引用外部的 schema。

```js
const addressSchema = S.object()
  .id('#address')
  .prop('line1').required()
  .prop('line2')
  .prop('country').required()
  .prop('city').required()
  .prop('zipcode').required()
  .valueOf()

const commonSchemas = S.object()
  .id('https://fastify/demo')
  .definition('addressSchema', addressSchema)
  .definition('otherSchema', otherSchema) // 你可以任意添加需要的 schema
  .valueOf()

fastify.addSchema(commonSchemas)

const bodyJsonSchema = S.object()
  .prop('residence', S.ref('https://fastify/demo#address')).required()
  .prop('office', S.ref('https://fastify/demo#/definitions/addressSchema')).required()
  .valueOf()

const schema = { body: bodyJsonSchema }

fastify.post('/the/url', { schema }, handler)
```


**`替换方式`**：在验证阶段之前，使用共用 schema 替换某些字段。

```js
const sharedAddressSchema = {
  $id: 'sharedAddress',
  type: 'object',
  required: ['line1', 'country', 'city', 'zipcode'],
  properties: {
    line1: { type: 'string' },
    line2: { type: 'string' },
    country: { type: 'string' },
    city: { type: 'string' },
    zipcode: { type: 'string' }
  }
}
fastify.addSchema(sharedAddressSchema)

const bodyJsonSchema = {
  type: 'object',
  properties: {
    vacation: 'sharedAddress#'
  }
}

const schema = { body: bodyJsonSchema }

fastify.post('/the/url', { schema }, handler)
```

特别注意：你可以在 `fastify.addSchema` 方法里混用 `$ref` 和 `替换方式`。

[fluent-schema-repo]: https://github.com/fastify/fluent-schema