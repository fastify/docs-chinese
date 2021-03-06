<h1 align="center">Fastify</h1>

<a name="lts"></a>

## 长期支持计划

Fastify 长期支持计划 (LTS) 以本文档为准：

1. 主要版本发布，[语义化版本][semver] X.Y.Z 发布版本中的 "X" 发布，至少有6个月的支持。特定版本的发布日期可以从[https://github.com/fastify/fastify/releases](https://github.com/fastify/fastify/releases)查到。

1. 主要版本将一直获得安全更新，直到下个主要版本发布后的6个月后。在这之后我们依然会发布安全补丁，只要社区有提供，且不会破坏其他约束，例如，支持的 Node.js 最低版本。

1. 主要版本会针对其长期支持期内的所有[长期支持的 Node.js 版本](https://github.com/nodejs/Release)进行测试验证。

"月" 意思为连续的30天。

[semver]: https://semver.org/

<a name="lts-schedule"></a>

### 计划

| 版本    | 发布日期   | 长期支持结束 |  Node.js 版本   |
| :------ | :----------- | :-------------- | :------------------- |
| 1.0.0   | 2018-03-06   | 2019-09-01      | 6, 8, 9, 10, 11      |
| 2.0.0   | 2019-02-25   | 2021-01-31      | 6, 8, 10, 12, 14     |
| 3.0.0   | 2020-07-07   | 待定            | 10, 12, 14           |

<a name="supported-os"></a>

### 经过 CI (持续集成) 测试的操作系统

| 系统    | 版本                   | 包管理器                  | Node.js   |
|---------|------------------------|---------------------------|-----------|
| Linux   | Ubuntu 16.04           | npm                       | 10,12,14  |
| Linux   | Ubuntu 16.04           | yarn,pnpm                 | 10,12     |
| Windows | Windows Server 2016 R2 | npm                       | 10,12,14  |
| MacOS   | macOS X Mojave 10.14   | npm                       | 10,12,14  |

使用 [yarn](https://yarnpkg.com/) 命令需添加 `--ignore-engines`。