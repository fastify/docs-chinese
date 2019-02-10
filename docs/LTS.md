<h1 align="center">Fastify</h1>

<a name="lts"></a>

## 长期支持计划

Fastify 长期支持计划 (LTS) 以本文档为准：

1. 主要版本发布，[语义化版本][semver] X.Y.Z 发布版本中的 "X" 发布，至少有6个月的支持。特定版本的发布日期可以从[https://github.com/fastify/docs-chinese/releases](https://github.com/fastify/docs-chinese/releases)查到。

1. 主要版本将一直获得安全更新，直到下个主要版本发布后的6个月后。在这之后我们依然会发布安全补丁，只要社区有提供，且不会破坏其他约束，例如，支持的 Node.js 最低版本。

1. 主要版本会针对其长期支持期内的所有[长期支持的 Node.js 版本](https://github.com/nodejs/Release)进行测试验证。

"月" 意思为连续的30天。

[semver]: https://semver.org/

<a name="lts-schedule"></a>

### 计划

| 版本    | 发布日期   | 长期支持结束 |  Node.js 版本   |
| :------ | :--------- | :----------- | :-------------- |
| 1.0.0   | 2018-03-06 | 2019-06-01   | 6, 8, 9, 10, 11 |
| 2.0.0   | 待定       | 待定         | 6, 8, 10, 11    |

<a name="supported-os"></a>

### 经过 CI (持续集成) 测试的操作系统

| CI              | OS      | 版本           | 包管理器        | Node.js 版本   |
| :-------------- | :------ | :------------- | :-------------- | :------------- |
| Travis          | Linux   | Ubuntu 14.04   | npm             | 6,8,9,10       |
| Azure pipelines | Linux   | Ubuntu 16.04   | yarn            | ~~6¹~~,8,10,11 |
| Azure pipelines | Windows | vs2017-win2016 | npm             | 6,8,10,11      |

_¹ yarn 只支持 node >= 8 的版本_