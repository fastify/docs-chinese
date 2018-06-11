<h1 align="center">Fastify</h1>

## 测试
测试是开发应用最重要的一部分. Fastify 处理测试非常灵活并且它兼容绝大多数框架(例如 [Tap](https://www.npmjs.com/package/tap), 下面的例子都会用这个演示).

<a name="inject"></a>
### 带有 http 注入的测试
感谢有 [`light-my-request`](https://github.com/fastify/light-my-request), Fastify 自带了伪造的 http 注入.

想要注入伪造的 http 请求, 使用 `inject` 方法:
```js
fastify.inject({
  method: String,
  url: String,
  payload: Object,
  headers: Object
}, (error, response) => {
  // your tests
})
```

或是用 promise 的版本

```js
fastify
  .inject({
    method: String,
    url: String,
    payload: Object,
    headers: Object
  })
  .then(response => {
    // your tests
  })
  .catch(err => {
    // handle error
  })
```

Async await 也是支持的!
```js
try {
  const res = await fastify.inject({ method: String, url: String, payload: Object, headers: Object })
  // your tests
} catch (err) {
  // handle error
}
```

#### 举例:

**app.js**
```js
const Fastify = require('fastify')

function buildFastify () {
  const fastify = Fastify()

  fastify.get('/', function (request, reply) {
    reply.send({ hello: 'world' })
  })
  
  return fastify
}

module.exports = buildFastify
```

**test.js**
```js
const tap = require('tap')
const buildFastify = require('./app')

tap.test('GET `/` route', t => {
  t.plan(4)
  
  const fastify = buildFastify()
  
  // At the end of your tests it is highly recommended to call `.close()`
  // to ensure that all connections to external services get closed.
  t.tearDown(() => fastify.close())

  fastify.inject({
    method: 'GET',
    url: '/'
  }, (err, response) => {
    t.error(err)
    t.strictEqual(response.statusCode, 200)
    t.strictEqual(response.headers['content-type'], 'application/json; charset=utf-8')
    t.deepEqual(JSON.parse(response.payload), { hello: 'world' })
  })
})
```

### 测试正在运行的服务器
你还可以在 fastify.listen() 启动服务器之后，或是 fastify.ready() 初始化路由与插件之后，进行 Fastify 的测试.

#### 举例:

使用之前例子的 **app.js**.

**test-listen.js** (用 [`Request`](https://www.npmjs.com/package/request) 测试)
```js
const tap = require('tap')
const request = require('request')
const buildFastify = require('./app')

tap.test('GET `/` route', t => {
  t.plan(5)
  
  const fastify = buildFastify()
  
  t.tearDown(() => fastify.close())
  
  fastify.listen(0, (err) => {
    t.error(err)
    
    request({
      method: 'GET',
      url: 'http://localhost:' + fastify.server.address().port
    }, (err, response, body) => {
      t.error(err)
      t.strictEqual(response.statusCode, 200)
      t.strictEqual(response.headers['content-type'], 'application/json; charset=utf-8')
      t.deepEqual(JSON.parse(body), { hello: 'world' })
    })
  })
})
```

**test-ready.js** (用 [`SuperTest`](https://www.npmjs.com/package/supertest) 测试)
```js
const tap = require('tap')
const supertest = require('supertest')
const buildFastify = require('./app')

tap.test('GET `/` route', async (t) => {
  const fastify = buildFastify()

  t.tearDown(() => fastify.close())
  
  await fastify.ready()
  
  const response = await supertest(fastify.server)
    .get('/')
    .expect(200)
    .expect('Content-Type', 'application/json; charset=utf-8')
  t.deepEqual(response.body, { hello: 'world' })
})
```
