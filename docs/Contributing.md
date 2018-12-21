## 翻译指南

如果你发现了翻译的错误，或文档的滞后，请创建一个 issue 来告诉我们，或是按如下方法直接参与翻译。感谢！

* Fork https://github.com/fastify/docs-chinese
* 创建 Git Branch
* 开始翻译
* 创建 PR，尽量保证一篇文档一个 commit 一个 PR

### 风格规范

为了保证翻译风格的统一，请按照如下规范进行翻译。
你可以在[这里](https://github.com/fastify/docs-chinese/issues/66)一起参与规范的讨论。

- 所有翻译的文本内容采用 markdown 编写，支持 GMF，不熟悉的话可以参考如下链接：
  - https://help.github.com/articles/getting-started-with-writing-and-formatting-on-github/
  - https://help.github.com/categories/writing-on-github/
- 中文与英文之间需要加一个空格，例如 `你好 hello 你好`
- 中文与数字之间需要加一个空格，例如 `2018 年 12 月 20 日`
- 正文中的英文标点需要转换为对应的中文标点，例如 `,` => `，`
  - 例外：英文括号修改为半角括号加一个空格的形式，例如 `()` => ` () `
  - 注意：英文逗号可能需要转换为中文顿号，而非中文逗号
- 代码片段中的注释需要翻译
- 不做翻译的内容：
  - 常见的缩写内容，例如 `UI`, `HTTP`
  - 常见的框架、平台、语言等，例如 `Express`, `Node`, `Java`
  - 用于表示工程代码中的一些实例，例如 `Promise`, Node 中的 `Request`
- 对于可能产生误解的名词翻译，如果在术语表中有，请按照对应的内容做翻译；如果术语表中未出现，那么推荐翻译后括号注解原文，例如 `运行时（Runtime）`

### 术语表