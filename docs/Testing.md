<h1 align="center">Fastify</h1>

## 测试

测试是开发应用最重要的一部分。Fastify 处理测试非常灵活并且它兼容绝大多数框架 (例如 [Tap](https://www.npmjs.com/package/tap)。下面的例子都会用这个演示)。

让我们 `cd` 进入一个全新的 'testing-example' 文件夹，并在终端里输入 `npm init -y`。

执行 `npm install fastify && npm install tap pino-pretty --save-dev`。

### 关注点分离让测试变得轻松

首先，我们将应用代码与服务器代码分离：

**app.js**:

```js
'use strict'

const fastify = require('fastify')

function build(opts={}) {
  const app = fastify(opts)
  app.get('/', async function (request, reply) {
    return { hello: 'world' }
  })

  return app
}

module.exports = build
```

**server.js**:

```js
'use strict'

const server = require('./app')({
  logger: {
    level: 'info',
    prettyPrint: true
  }
})

server.listen(3000, (err, address) => {
  if (err) {
    console.log(err)
    process.exit(1)
  }
})
```

### 使用 fastify.inject() 的好处

感谢有 [`light-my-request`](https://github.com/fastify/light-my-request)，Fastify 自带了伪造的 HTTP 注入。

在进行任何测试之前，我们通过 `.inject` 方法向路由发送假的请求：

**app.test.js**:

```js
'use strict'

const build = require('./app')

const test = async () => {
  const app = build()

  const response = await app.inject({
    method: 'GET',
    url: '/'
  })

  console.log('status code: ', response.statusCode)
  console.log('body: ', response.body)
}
test()
```

我们的代码运行在异步函数里，因此可以使用 async/await。

`.inject` 确保了所有注册的插件都已引导完毕，可以开始测试应用了。之后请求方法将被传递到路由函数中去。使用 await 可以存储响应，且避免了回调函数。

在终端执行 `node app.test.js` 来开始测试。

```sh
status code:  200
body:  {"hello":"world"}
```

### HTTP 注入测试

现在我们能用真实的测试语句代替 `console.log` 了！

在 `package.json` 里修改 "test" script 如下：

`"test": "tap --reporter=list --watch"`

**app.test.js**:

```js
'use strict'

const { test } = require('tap')
const build = require('./app')

test('requests the "/" route', async t => {
  const app = build()

  const response = await app.inject({
    method: 'GET',
    url: '/'
  })
  t.equal(response.statusCode, 200, 'returns a status code of 200')
})
```

执行 `npm test`，查看结果！

`inject` 方法能完成的不只有简单的 GET 请求：
```js
fastify.inject({
  method: String,
  url: String,
  query: Object,
  payload: Object,
  headers: Object,
  cookies: Object
}, (error, response) => {
  // 你的测试
})
```

忽略回调函数，可以链式调用 `.inject` 提供的方法：

```js
fastify
  .inject()
  .get('/')
  .headers({ foo: 'bar' })
  .query({ foo: 'bar' })
  .end((err, res) => { //  调用 .end 触发请求
    console.log(res.payload)
  })
```

或是用 promise 的版本

```js
fastify
  .inject({
    method: String,
    url: String,
    query: Object,
    payload: Object,
    headers: Object,
    cookies: Object
  })
  .then(response => {
    // 你的测试
  })
  .catch(err => {
    // 处理错误
  })
```

Async await 也是支持的！
```js
try {
  const res = await fastify.inject({ method: String, url: String, payload: Object, headers: Object })
  // 你的测试
} catch (err) {
  // 处理错误
}
```

#### 另一个例子：

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

  // 在测试的最后，我们强烈建议你调用 `.close()`
  // 方法来确保所有与外部服务的连接被关闭。
  t.teardown(() => fastify.close())

  fastify.inject({
    method: 'GET',
    url: '/'
  }, (err, response) => {
    t.error(err)
    t.equal(response.statusCode, 200)
    t.equal(response.headers['content-type'], 'application/json; charset=utf-8')
    t.same(response.json(), { hello: 'world' })
  })
})
```

### 测试正在运行的服务器
你还可以在 fastify.listen() 启动服务器之后，或是 fastify.ready() 初始化路由与插件之后，进行 Fastify 的测试。

#### 举例:

使用之前例子的 **app.js**。

**test-listen.js** (用 [`Request`](https://www.npmjs.com/package/request) 测试)
```js
const tap = require('tap')
const request = require('request')
const buildFastify = require('./app')

tap.test('GET `/` route', t => {
  t.plan(5)

  const fastify = buildFastify()

  t.teardown(() => fastify.close())

  fastify.listen(0, (err) => {
    t.error(err)
    
    request({
      method: 'GET',
      url: 'http://localhost:' + fastify.server.address().port
    }, (err, response, body) => {
      t.error(err)
      t.equal(response.statusCode, 200)
      t.equal(response.headers['content-type'], 'application/json; charset=utf-8')
      t.same(JSON.parse(body), { hello: 'world' })
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

  t.teardown(() => fastify.close())

  await fastify.ready()

  const response = await supertest(fastify.server)
    .get('/')
    .expect(200)
    .expect('Content-Type', 'application/json; charset=utf-8')
  t.same(response.body, { hello: 'world' })
})
```

### 如何检测 tap 的测试
1. 设置 `{only: true}` 选项，将需要检测的测试与其他测试分离
```javascript
test('should ...', {only: true}, t => ...)
```
2. 通过 `npx` 运行 `tap`
```bash
> npx tap -O -T --node-arg=--inspect-brk test/<test-file.test.js>
```
- `-O` 表示开启 `only` 选项，只运行设置了 `{only: true}` 的测试
- `-T` 表示不设置超时
- `--node-arg=--inspect-brk` 会启动 node 调试工具
3. 在 VS Code 中创建并运行一个 `Node.js: Attach` 调试配置，不需要额外修改。

现在你便可以在编辑器中检测你的测试文件 (以及 `Fastify` 的其他部分) 了。