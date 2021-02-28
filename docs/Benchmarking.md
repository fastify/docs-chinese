<h1 align="center">Fastify</h1>

## 基准测试
基准测试对于衡量改动可能引起的性能变化很重要. 从用户和贡献者的角度, 我们提供了简便的方法测试你的应用. 这套配置可以自动化你的基准测试, 从不同的分支和不同的 Node.js 的版本.

我们将使用以下模块:
- [Autocannon](https://github.com/mcollina/autocannon): 用 Node 写的 HTTP/1.1 基准测试工具.
- [Branch-comparer](https://github.com/StarpTech/branch-comparer): 切换不同的 git 分支, 执行脚本并记录结果.
- [Concurrently](https://github.com/kimmobrunfeldt/concurrently): 并行运行命令.
- [Npx](https://github.com/npm/npx): NPM 包运行器，用于在不同的 Node.js 版本上执行运行脚本和运行本地的二进制文件. 在 npm@5.2.0 版本以上可用.

## 基本用法

### 在当前分支上运行测试
```sh
npm run benchmark
```

### 在不同的 Node.js 版本中运行测试 ✨
```sh
npx -p node@6 -- npm run benchmark
```

## 进阶用法

### 在不同的分支上运行测试
```sh
branchcmp --rounds 2 --script "npm run benchmark"
```

### 在不同的分支和不同的 Node.js 版本中运行测试 ✨
```sh
branchcmp --rounds 2 --script "npm run benchmark"
```

### 比较当前的分支和主分支 (Gitflow)
```sh
branchcmp --rounds 2 --gitflow --script "npm run benchmark"
```
or
```sh
npm run bench
```

### 运行不同的用例

```sh
branchcmp --rounds 2 -s "node ./node_modules/concurrently -k -s first \"node ./examples/asyncawait.js\" \"node ./node_modules/autocannon -c 100 -d 5 -p 10 localhost:3000/\""
```
