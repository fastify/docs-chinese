<h1 align="center">Fastify</h1>

# 如何写一个好的插件
首先，要感谢你决定为 Fastify 编写插件。Fastify 本身是一个极简的框架，插件才是它强大功能的来源，所以，谢谢你。<br>
Fastify 的核心原则是高性能、低成本、提供优秀的用户体验。当编写插件时，这些原则应当被遵循。因此，本文我们将会分析一个优质的插件所具有的特征。

*需要一些灵感？你可以在 issue 中使用 ["plugin suggestion"](https://github.com/fastify/fastify/issues?q=is%3Aissue+is%3Aopen+label%3A%22plugin+suggestion%22) 标签！*

## 代码
Fastify 运用了不同的技术来优化代码，其大部分都被写入了文档。我们强烈建议你阅读 [插件指南](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins-Guide.md) 一文，以了解所有可用于构建插件的 API 及其用法。

存有疑虑或寻求建议？我们非常高兴能帮助你！只需在我们的 [求助仓库](https://github.com/fastify/help) 提一个 issue 即可！

一旦你向我们的 [生态列表](https://github.com/fastify/fastify/blob/master/docs/Ecosystem.md) 提交了一个插件，我们将会检查你的代码，需要时也会帮忙改进它。

## 文档
文档相当重要。假如你的插件没有好的文档，我们将拒绝将其加入生态列表。缺乏良好的文档会提升用户使用插件的难度，并有可能导致弃用。<br>
以下列出了一些优秀插件文档的示例：
- [`fastify-caching`](https://github.com/fastify/fastify-caching)
- [`fastify-compress`](https://github.com/fastify/fastify-compress)
- [`fastify-cookie`](https://github.com/fastify/fastify-cookie)
- [`point-of-view`](https://github.com/fastify/point-of-view)
- [`under-pressure`](https://github.com/fastify/under-pressure)

## 许可证
你可以为你的插件使用自己偏好的许可，我们不会强求。<br>
我们推荐 [MIT 许可证](https://choosealicense.com/licenses/mit/)，因为我们认为它允许更多人自由地使用代码。其他可替代的许可证参见 [OSI list](https://opensource.org/licenses) 或 GitHub 的 [choosealicense.com](https://choosealicense.com/)。

## 示例
总在你的仓库里添加一个示例文件。这对于用户是相当有帮助的，也提供了一个快速的手段来测试你的插件。使用者们会为此感激的。

## 测试
彻底地测试一个插件，来验证其是否正常执行，是极为重要的。<br>
缺乏测试会影响用户的信任感，也无法保证代码在不同版本的依赖下还能正常工作。

我们不强求使用某一测试工具。我们使用的是 [`tap`](http://www.node-tap.org/)，因为它提供了开箱即用的并行测试以及代码覆盖率检测。

## 代码检查
这一项不是强制的，但我们强烈推荐你在插件中使用一个代码检查工具。这可以帮助你保持统一的代码风格，同时避免许多错误。

我们使用 [`standard`](https://standardjs.com/)，因为它不需任何配置，并且容易与测试集成。

## 持续集成
这一项也不是强制的，但假如你开源发布你的代码，持续集成能保证其他人的参与不会破坏你的插件，并检查插件是否如预期般工作。[Travis](https://travis-ci.org/) 持续集成系统对开源项目免费，且易于安装配置。<br>
此外，你还可以启用 [Greenkeeper](https://greenkeeper.io/) 等服务，它可以帮你将依赖保持在最新版本，并检查在 Fastify 的新版本上你的插件是否存在问题。

## 让我们开始吧！
棒极了，现在你已经了解了如何为 Fastify 写一个好插件！
当你完成了一个插件（或更多）之后，请让我们知道！我们会将其添加到 [生态](https://github.com/fastify/fastify#ecosystem) 一节中！

想看更多真实的例子？请参阅：
- [`point-of-view`](https://github.com/fastify/point-of-view)
Fastify 的模板渲染 (*ejs, pug, handlebars, marko*) 插件。
- [`fastify-mongodb`](https://github.com/fastify/fastify-mongodb)
Fastify 的 MongoDB 连接插件，通过它你可以在你服务器的每个部分共享同一个 MongoDb 连接池。
- [`fastify-multipart`](https://github.com/fastify/fastify-multipart)
为 Fastify 提供 mltipart 支持。
- [`fastify-helmet`](https://github.com/fastify/fastify-helmet)
重要的请求头安全插件。