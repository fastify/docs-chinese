<h1 align="center">Fastify</h1>

<a name="lts"></a>

## 长期支持计划

Fastify 长期支持计划 (LTS) 以本文档为准：

1. 主要版本发布，[语义化版本][semver] X.Y.Z 发布版本中的 "X" 发布，至少有6个月的支持。特定版本的发布日期可以从[https://github.com/fastify/fastify/releases](https://github.com/fastify/fastify/releases)查到。

1. 主要版本将一直获得安全更新，直到下个主要版本发布后的6个月后。在这之后我们依然会发布安全补丁，只要社区有提供，且不会破坏其他约束，例如，支持的 Node.js 最低版本。

1. 主要版本会针对其长期支持期内的所有[长期支持的 Node.js 版本](https://github.com/nodejs/Release)进行测试验证。

"月" 意思为连续的30天。

> ## 安全版本与语义化 Security Releases and Semver
>
> 由于为主要版本提供长期支持十分重要，
> 我们有时需要在 _次要版本_ 上发布重大的改动。
> 这些变更 _总是_ 会记录在[发布记录](https://github.com/fastify/fastify/releases)里。
>
> 要避免自动升级到重大的安全更新版本，
> 你可以使用波浪号 (`~`) 来标识版本范围。
> 例如，将依赖标识为 `"fastify": "~3.15.x"`，
> 可以为 3.15 更新补丁，也避免了自动升级到 3.16。
> 这么做会使你的应用变得脆弱，因此请谨慎使用。

[semver]: https://semver.org/

<a name="lts-schedule"></a>

### 计划

| 版本    | 发布日期   | 长期支持结束 |  Node.js 版本   |
| :------ | :----------- | :-------------- | :------------------- |
| 1.0.0   | 2018-03-06   | 2019-09-01      | 6, 8, 9, 10, 11      |
| 2.0.0   | 2019-02-25   | 2021-01-31      | 6, 8, 10, 12, 14     |
| 3.0.0   | 2020-07-07   | 待定            | 10, 12, 14, 16       |

<a name="supported-os"></a>

### 经过 CI (持续集成) 测试的操作系统

Fastify 使用 GitHub Actions 来进行 CI 测试，请参阅 [GitHub 相关文档](https://docs.github.com/cn/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources)来获取下表 YAML 工作流标签所对应的最新虚拟机环境。

| 系统    | YAML 工作流标签        | 包管理器                  | Node.js      |
|---------|------------------------|---------------------------|--------------|
| Linux   | `ubuntu-latest`        | npm                       | 10,12,14,16  |
| Linux   | `ubuntu-18.04`         | yarn,pnpm                 | 10,12        |
| Windows | `windows-latest`       | npm                       | 10,12,14,16  |
| MacOS   | `macos-latest`         | npm                       | 10,12,14,16  |

使用 [yarn](https://yarnpkg.com/) 命令需添加 `--ignore-engines`。