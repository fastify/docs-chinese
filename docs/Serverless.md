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

*注：这仅是一种可行方案。*

### app.js

```js
const fastify = require('fastify');

function init(serverFactory) {
  const app = fastify({ serverFactory });
  app.get('/', (request, reply) => reply.send({ hello: 'world' }));
  return app;
}

if (require.main === module) {
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
在 [`lambda.js`](https://www.fastify.io/docs/latest/Server/#lambda.js) 里，我们会用到它。

当像往常一样运行 Fastify 应用，
比如执行 `node app.js` 时 *(可以用 `require.main === module` 来判断)*，
你可以监听某个端口，如此便能本地运行应用了。

### lambda.js

```js
const awsServerlessExpress = require('aws-serverless-express');
const init = require('./app');

let server;
const serverFactory = (handler) => {
  server = awsServerlessExpress.createServer(handler);
  return server;
}
const app = init(serverFactory);

exports.handler = (event, context, callback) => {
  context.callbackWaitsForEmptyEventLoop = false;
  app.ready((e) => {
    if (e) return console.error(e.stack || e);
    awsServerlessExpress.proxy(server, event, context, 'CALLBACK', callback);
  });
};
```

我们自定义了一个 `serverFactory` 函数，在该函数内通过 [`aws-serverless-express`](https://github.com/awslabs/aws-serverless-express) 创建了新的服务器 (请确保安装了这个依赖 `npm i --save aws-serverless-express`)。
之后，我们调用从 [`app.js`](https://www.fastify.io/docs/latest/Server/#app.js) 导入的 `init` 函数，并传入唯一参数 `serverFactory`。
最后，在 lambda `handler` 函数中，我们等待 Fastify 应用 `ready`，再将所有请求事件 (API Gateway 的请求) 代理到 [`aws-serverless-express`](https://github.com/awslabs/aws-serverless-express) 的 `proxy` 函数。


### 示例

你可以在[这里](https://github.com/claudiajs/example-projects/tree/master/fastify-app-lambda)找到使用 [claudia.js](https://claudiajs.com/tutorials/serverless-express.html) 的可部署的例子。


### 注意事项

- 你没法操作 [stream](https://www.fastify.io/docs/latest/Reply/#streams)，因为 API Gateway 还不支持它。
- API Gateway 的超时时间为 29 秒，请务必在此时限内回复。