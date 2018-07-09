<h1 align="center">Fastify</h1>

## 装饰器

想给 Fastify 实例新增功能？`decorate` 正是你所需要的 API！

`decorate` 允许你向 Fastify 实例添加新的属性。属性的值并不限于函数，它也可以是对象、字符串等。

<a name="usage"></a>
### 使用方法
<a name="decorate"></a>
**decorate**
你只需调用 `decorate` 函数，并将新属性的名称与值作为参数传递即可。
```js
fastify.decorate('utility', () => {
  // 新功能的代码
})
```

正如上文所述，你还可以传递非函数的值：
```js
fastify.decorate('conf', {
  db: 'some.db',
  port: 3000
})
```

一旦添加了一个装饰器，你就可以通过其名称访问它的值了：
```js
fastify.utility()

console.log(fastify.conf.db)
```

装饰器是 *不可覆盖* 的。如果你声明一个已存在的 *(存在同名)* 装饰器，`decorate` 将会抛出一个异常。

装饰器接受特别的 "getter/setter" 对象。这些对象拥有着名为 `getter` 与 `setter` 的函数 (尽管 `setter` 是可选的)。这么做便可以通过装饰器来定义属性。例如：

```js
fastify.decorate('foo', {
  getter () {
    return 'a getter'
  }
})
```

上例会在 *Fastify* 实例中定义一个 `foo` 属性：

```js
console.log(fastify.foo) // 'a getter'
```

<a name="decorate-reply"></a>
**decorateReply**
顾名思义，当你需要向 `Reply` 核心对象添加新方法时，使用 `decorateReply` API。同 `decorate` 一样，将新属性与其值作为参数传递便大功告成了：
```js
fastify.decorateReply('utility', function () {
  // 新功能的代码
})
```

注：使用箭头函数会破坏 `this` 和 Fastify `reply` 实例的绑定。

<a name="decorate-request"></a>
**decorateRequest**
同理，使用 `decorateRequest` 可向 `Request` 核心对象新增方法。传递的参数同样也是新属性的名称以及值：
```js
fastify.decorateRequest('utility', function () {
  // 新功能的代码
})
```

注：使用箭头函数会破坏 `this` 和 Fastify `reply` 实例的绑定。

<a name="usage_notes"></a>
#### Usage Notes
`decorateReply` 与 `decorateRequest` 是分别用于向 `Reply` 和 `Request` 对象新增方法或属性的。你可以通过对象直接访问这些属性来更新它们。

让我们向 `Request` 对象添加一个用户属性：

```js
// 添加一个 'user' 请求装饰器
fastify.decorateRequest('user', '')

// 更新属性
fastify.addHook('preHandler', (req, reply, next) => {
  req.user = 'Bob Dylan'
  next()
})
// 最后，访问装饰器
fastify.get('/', (req, reply) => {
  reply.send(`Hello ${req.user}!`)
})
```
注：在这个例子里，使用 `decorateReply` 或 `decorateRequest` 不是必要的，但这么做能提升 Fastify 的性能。

<a name="sync-async"></a>
#### 同步与异步
`decorate` 是 *同步* 的 API。如果你需要添加一个 *异步* 引导的装饰器，Fastify 可能会在该装饰器准备妥当前启动。为了避免这种情况，你应该将 `register` 与 `fastify-plugin` 一起使用。更多内容，请看[插件](https://github.com/fastify/docs-chinese/blob/master/docs/Plugins.md)一文。

<a name="dependencies"></a>
#### 依赖项
如果你的装饰器依赖于其他装饰器，你可以轻易地将其他装饰器声明为依赖项。你要做的只是将一个字符串数组 (表示依赖项的名称) 作为函数的第三个参数而已：
```js
fastify.decorate('utility', fn, ['greet', 'log'])
```

如果依赖关系不满足，`decorate` 会抛出异常。但请别担心：依赖检查是在服务器启动之前执行的，因此，在运行时不会发生该问题。

<a name="has-decorator"></a>
#### hasDecorator
使用 `hasDecorator` API 检查一个装饰器是否存在：
```js
fastify.hasDecorator('utility')
```

<a name="has-request-decorator"></a>
#### hasRequestDecorator
使用 `hasRequestDecorator` API 检查一个请求的的装饰器是否存在：
```js
fastify.hasRequestDecorator('utility')
```

<a name="has-reply-decorator"></a>
#### hasReplyDecorator
使用 `hasReplyDecorator` API 检查一个回复的装饰器是否存在：
```js
fastify.hasReplyDecorator('utility')
```
