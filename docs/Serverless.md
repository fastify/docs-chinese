<h1 align="center">Serverless</h1>

使用现有的 Fastify 应用运行无服务器 (serverless) 应用与 REST API。

### 目录

- [AWS Lambda](#aws-lambda)
- [Google Cloud Run](#google-cloud-run)
- [Netlify Lambda](#netlify-lambda)
- [Vercel](#vercel)

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

## Google Cloud Run

与 AWS Lambda 和 Google Cloud Functions 不同，Google Cloud Run 是一个无服务器**容器**环境。它的首要目的是提供一个能运行任意容器的底层抽象 (infrastucture-abstracted) 的环境。因此，你能将 Fastify 部署在 Google Cloud Run 上，而且相比正常的写法，只需要改动极少的代码。

*参照以下步骤部署 Google Cloud Run。如果你对 gcloud 还不熟悉，请看其[入门文档](https://cloud.google.com/run/docs/quickstarts/build-and-deploy)*。

### 调整 Fastify 服务器

为了让 Fastify 能正确地在容器里监听请求，请确保设置了正确的端口与地址：

```js
function build() {
  const fastify = Fastify({ trustProxy: true })
  return fastify
}

async function start() {
  // Google Cloud Run 会设置这一环境变量，
  // 因此，你可以使用它判断程序是否运行在 Cloud Run 之中
  const IS_GOOGLE_CLOUD_RUN = process.env.K_SERVICE !== undefined

  // 监听 Cloud Run 提供的端口
  const port = process.env.PORT || 3000

  // 监听 Cloud Run 中所有的 IPV4 地址
  const address = IS_GOOGLE_CLOUD_RUN ? "0.0.0.0" : undefined

  try {
    const server = build()
    const address = await server.listen(port, address)
    console.log(`Listening on ${address}`)
  } catch (err) {
    console.error(err)
    process.exit(1)
  }
}

module.exports = build

if (require.main === module) {
  start()
}
```

### 添加 Dockerfile

你可以添加任意合法的 `Dockerfile`，用于打包运行 Node 程序。在 [gcloud 官方文档](https://github.com/knative/docs/blob/2d654d1fd6311750cc57187a86253c52f273d924/docs/serving/samples/hello-world/helloworld-nodejs/Dockerfile)中，你能找到一份基本的 `Dockerfile`。

```Dockerfile
# 使用官方 Node.js 10 镜像。
# https://hub.docker.com/_/node
FROM node:10

# 创建并切换到应用目录。
WORKDIR /usr/src/app

# 拷贝应用依赖清单至容器镜像。
# 使用通配符来确保 package.json 和 package-lock.json 均被复制。
# 独立地拷贝这些文件，能防止代码改变时重复执行 npm install。
COPY package*.json ./

# 安装生产环境依赖。
RUN npm install --only=production

# 复制本地代码到容器镜像。
COPY . .

# 启动容器时运行服务。
CMD [ "npm", "start" ]
```

### 添加 .dockerignore

添加一份如下的 `.dockerignore`，可以将仅用于构建的文件排除在容器之外 (能减小容器大小，加快构建速度)：

```.dockerignore
Dockerfile
README.md
node_modules
npm-debug.log
```

### 提交构建

接下来，使用以下命令将你的应用构建成一个 Docker 镜像 (将 `PROJECT-ID` 和 `APP-NAME` 替换为 Google 云平台的项目 id 和 app 名称)：

```bash
gcloud builds submit --tag gcr.io/PROJECT-ID/APP-NAME
```

### 部署镜像

镜像构建之后，使用如下命令部署它：

```bash
gcloud beta run deploy --image gcr.io/PROJECT-ID/APP-NAME --platform managed
```

如此，便能从 Google 云平台提供的链接访问你的应用了。

## netlify-lambda

首先，完成与 **AWS Lambda** 有关的准备工作。

新建 `functions` 文件夹，在其中创建 `server.js` (应用的入口文件)。

### functions/server.js

```js
export { handler } from '../lambda.js'; // 记得将路径修改为你的应用中对应的 `lambda.js` 的路径
```

### netlify.toml

```toml
[build]
  # 构建站点时执行的命令
  command = "npm run build:functions"
  # 发布到 netlify CDN 的文件夹
  # 同时也是应用的前端
  # publish = "build"
  # 构建好的 Lambda 函数的目录
  functions = "functions-build" # 总是为构建后的 `functions` 文件夹名称加上 `-build` 后缀
```

### webpack.config.netlify.js

**别忘记添加这个文件，否则会有不少问题**

```js
const nodeExternals = require('webpack-node-externals');
const dotenv = require('dotenv-safe');
const webpack = require('webpack');

const env = process.env.NODE_ENV || 'production';
const dev = env === 'development';

if (dev) {
  dotenv.config({ allowEmptyValues: true });
}

module.exports = {
  mode: env,
  devtool: dev ? 'eval-source-map' : 'none',
  externals: [nodeExternals()],
  devServer: {
    proxy: {
      '/.netlify': {
        target: 'http://localhost:9000',
        pathRewrite: { '^/.netlify/functions': '' }
      }
    }
  },
  module: {
    rules: []
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env.APP_ROOT_PATH': JSON.stringify('/'),
      'process.env.NETLIFY_ENV': true,
      'process.env.CONTEXT': env
    })
  ]
};
```

### Scripts

在 `package.json` 的 *scripts* 里加上这一命令

```json
"scripts": {
...
"build:functions": "netlify-lambda build functions --config ./webpack.config.netlify.js"
...
}
```

这样就完成了。

## Vercel

[Vercel](https://vercel.com) 针对 Node.js 应用提供了零配置部署方案。要使用 now，只需要如下配置你的 `vercel.json` 文件：

```json
{
  "rewrites": [
    { 
      "source": "/(.*)", 
      "destination": "/api/serverless.js" 
    }
  ]
}
```

之后，写一个 `api/serverless.js` 文件：

```js
"use strict";

// 读取 .env 文件
import * as dotenv from "dotenv";
dotenv.config();

// 引入 Fastify 框架
import Fastify from "fastify";

// 实例化 Fastify
const app = Fastify({
  logger: true,
});

// 将应用注册为一个常规插件
app.register(import("../src/app"));

export default async (req, res) => {
  await app.ready();
  app.server.emit('request', req, res);
}
```