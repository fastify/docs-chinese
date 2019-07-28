<h1 align="center">Serverless</h1>

使用现有的 Fastify 应用运行无服务器 (serverless) 应用与 REST API。

### 读者须知：
> Fastify 并不是为无服务器环境准备的。
Fastify 框架的设计初衷是轻松地实现一个传统的 HTTP/S 服务器。
无服务器环境则与此不同。
因此，我们不保证在无服务器环境下，Fastify 也能如预期般运转。
尽管如此，参照本文中的示例，你仍然可以在无服务器环境中运行 Fastify。
再次提醒，无服务器环境不是 Fastify 的目标所在，我们不会在这样的集成情景下进行测试。


## AWS Lambda

以下是使用 Fastify 在 AWS Lambda 和 Amazon API Gateway 架构上构建无服务器 web 应用/服务的示例。

*注：使用 [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify) 仅是一种可行方案。*

### app.js

```js
const fastify = require('fastify');

function init() {
  const app = fastify();
  app.get('/', (request, reply) => reply.send({ hello: 'world' }));
  return app;
}

if (require.main !== module) {
  // 直接调用，即执行 "node app"
  init().listen(3000, (err) => {
    if (err) console.error(err);
    console.log('server listening on 3000');
  });
} else {
  // 作为模块引入 => 用于 aws lambda
  module.exports = init;
}
```

你可以简单地把初始化代码包裹于可选的 [serverFactory](https://www.fastify.io/docs/latest/Server/#serverfactory) 选项里。

当执行 lambda 函数时，我们不需要监听特定的端口，因此，在这个例子里我们只要导出 `init` 函数即可。
在 [`lambda.js`](https://www.fastify.io/docs/latest/Serverless/#lambda-js) 里，我们会用到它。

当像往常一样运行 Fastify 应用，
比如执行 `node app.js` 时 *(可以用 `require.main === module` 来判断)*，
你可以监听某个端口，如此便能本地运行应用了。

### lambda.js

```js
const awsLambdaFastify = require('aws-lambda-fastify')
const init = require('./app');

const proxy = awsLambdaFastify(init())
// 或
// const proxy = awsLambdaFastify(init(), { binaryMimeTypes: ['application/octet-stream'] })

exports.handler = proxy;
// 或
// exports.handler = (event, context, callback) => proxy(event, context, callback);
// 或
// exports.handler = (event, context) => proxy(event, context);
// 或
// exports.handler = async (event, context) => proxy(event, context);
```

我们只需要引入 [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify) (请确保安装了该依赖 `npm i --save aws-lambda-fastify`) 以及我们写的 [`app.js`](https://www.fastify.io/docs/latest/Serverless/#app-js)，并使用 `app` 作为唯一参数调用导出的 `awsLambdaFastify` 函数。
以上步骤返回的 `proxy` 函数拥有正确的签名，可作为 lambda 的处理函数。
如此，所有的请求事件 (API Gateway 的请求) 都会被代理到 [aws-lambda-fastify](https://github.com/fastify/aws-lambda-fastify) 的 `proxy` 函数。

### 示例

你可以在[这里](https://github.com/claudiajs/example-projects/tree/master/fastify-app-lambda)找到使用 [claudia.js](https://claudiajs.com/tutorials/serverless-express.html) 的可部署的例子。


### 注意事项

- 你没法操作 [stream](https://www.fastify.io/docs/latest/Reply/#streams)，因为 API Gateway 还不支持它。
- API Gateway 的超时时间为 29 秒，请务必在此时限内回复。