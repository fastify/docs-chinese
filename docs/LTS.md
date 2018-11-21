<h1 align="center">Fastify</h1>

<a name="lts"></a>

## 长期支持计划

Fastify 长期支持计划 (LTS) 以本文档为准：

1. 主要版本发布，[语义化版本][semver] X.Y.Z 发布版本中的 "X" 发布，至少有6个月的支持。特定版本的发布日期可以从[https://github.com/fastify/fastify/releases](https://github.com/fastify/fastify/releases)查到。

1. 主要版本将一直获得安全更新，直到下个主要版本发布后的6个月后。

"月" 意思为连续的30天。

[semver]: https://semver.org/

<a name="lts-schedule"></a>

### 计划

|  版 本  |  发布日期  |  长期支持结束  |
| :------ | :--------- | :------------- |
| 1.0.0   | 2018-03-06 | 待定           |

<a name="supported-os"></a>

### 经过 CI (持续集成) 测试的操作系统

| CI             | OS      | 版本           | 包管理器        | Node.js 版本   |
| :------------- | :------ | :------------- | :-------------- | :------------- |
| Travis         | Linux   | Ubuntu 14.04   | npm             | 6,8,9,10       |
| Azure pipeline | Linux   | Ubuntu 16.04   | yarn            | ~~6¹~~,8,10,11 |
| Azure pipeline | Windows | vs2017-win2016 | npm             | 6,8,10,11      |

_¹ yarn 只支持 node >= 8 的版本_