---
date: "2018-11-08 18:28:13"
title: graphql
---

graphql经常被认为是聚焦于前端的技术。

# 核心概念

## SDL：schema definition language（模式定义语言）

如：

```typescript
type Person{
    name: String!
    age: Int!
}
```

这个类型有两个字段，name和age，他们的类型是String和Int。！的意思代表他们是必需的。

```typescript
type Post{
    title: String!
    author: Person!
}
```

接下来的Post也有两个字段，其中Person也是可以作为一个类型。

也可以这样，在Person中添加一个post：

```tsx
type Person{
    name: String!
    age: Int!
    posts: [Post!]!
}
```

## 通过Query获取数据

### 基本查询

客户端发送下面的数据给服务器

```typescript
{
    allPersons {
        name
    }
}
```

allPersons是根字段（root field），它下面的成为查询的payload，这里仅包含了一个name。

服务器返回的结果会是这样的：

```tsx
{
  	"allPersons": [
    	{ "name": "Johnny" },
    	{ "name": "Sarah" },
    	{ "name": "Alice" }
  	]
}
```

可以看到只返回了name字段，age字段是不会返回的。

如果使用如下的payload就会返回：

```tsx
{
  	allPersons {
    	name
    	age
  	}
}
```

还可以查询posts中的title：

```
{
  allPersons {
    name
    age
    posts {
      title
    }
  }
}
```

### 带参数查询

在graphql中每个字段都有0或者更多个参数。比如allPerson有一个last参数，只返回最后两个人的信息，这里就是查询的语句：

```
{
  allPersons(last: 2) {
    name
  }
}
```

## 通过Mutation写数据

- 创建
- 更新
- 删除

mutation和query类似，只是需要加上mutation关键字。如：

```tsx
mutation {
  createPerson(name: "Bob", age: 36) {
    name
    age
  }
}
```

mutation也有一个根字段，叫createPerson。我们知道这个字段有两个参数name和age。返回值会像这样：

```tsx
{
  "data": {
    "createPerson": {
      "name": "Bob",
      "age": 36
    }
  }
}
```

graphql会给每个记录新增一个唯一的ID字段，我们也可以这样设置Person类型：

```tsx
type Person {
  id: ID!
  name: String!
  age: Int!
}
```

然后当一个新的Person对象创建时，就可以访问到id。

## 通过订阅实时更新

graphql提供了实时订阅更新。

当客户端订阅一个事件的时候，将会保持一个和服务器的稳定连接，当有变化时会告诉客户端。

```ts
subscription {
  newPerson {
    name
    age
  }
}
```

因此当有个用户创建或者修改时都会告诉客户端：

```
{
  "newPerson": {
    "name": "Jane",
    "age": 23
  }
}
```

## 定义一个模式

有几个特殊的根类型：

```ts
type Query { ... }
type Mutation { ... }
type Subscription { ... }
```

API的根字段都是在上面这三个之下，如：

```ts
type Query {
  allPersons: [Person!]!
}
```

allPersons也可以有参数：

```ts
type Query {
  allPersons(last: Int): [Person!]!
}
```

类似的mutation也是：

```ts
type Mutation {
  createPerson(name: String!, age: Int!): Person!
}
```

订阅也是：

```ts
type Subscription {
  newPerson: Person!
}
```

把他们放在一起就是：

```ts
type Query {
  allPersons(last: Int): [Person!]!
}

type Mutation {
  createPerson(name: String!, age: Int!): Person!
}

type Subscription {
  newPerson: Person!
}

type Person {
  name: String!
  age: Int!
  posts: [Post!]!
}

type Post {
  title: String!
  author: Person!
}
```

## 架构图

graphql直连数据库

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fx0v777rc9j30lr0573yq.jpg)

graphql连接层连接多个服务

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fx0v7pe3qkj30p60j2t9u.jpg)

graphql混连数据库和服务

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fx0v867hy3j30k30jrwff.jpg)

## 解析函数

每个字段其实都有一个解析器，叫resolver。

当服务器收到一个请求时，会调用字段的resolver函数，一旦resolver函数有返回，服务器就会把数据包装成要返回的字段。

有这样一个类型：

```ts
type Query {
  author(id: ID!): Author
}

type Author {
  posts: [Post]
}

type Post {
  title: String
  content: String
}
```

当执行一个query的时候：

```ts
query {
  author(id: "abc") {
    posts {
      title
      content
    }
  }
}
```

会以如下的方式执行：

```ts
Query.author(root, { id: 'abc' }, context) -> author
Author.posts(author, null, context) -> posts
for each post in posts
  Post.title(post, null, context) -> title
  Post.content(post, null, context) -> content
```

# 实战

