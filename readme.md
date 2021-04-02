# GraphQL + Apollo Client (React) + Typescript

## 前言

社区里关于graphQL的介绍，入门，原理等文章已经非常的多，graphQL和Apollo client的官方网站也介绍的比较详细。社区中也有相当多的文章从各个方面分析了graphqlQL相对比于Restful API的优劣利弊。本文主要是从前端的角度出发，在如今前后端开发分离盛行的前提下，介绍我们如何利用graphql社区中的工具提高开发效率和工程质量的经验，希望对已经决心入坑graphql的朋友们有一些帮助。

## GraphQL Config

首先要提的是 [GraphQL Config](https://graphql-config.com/introduction), 一份配置文件支持所有和graphQL相关的工具。配置文件支持多种文件类型，例如json, yaml, yml, js, ts等等，并且社区里大部分工具都已支持graphql config配置文件，graphql官方也建议开发者使用graphql config来作为配置文件。

```
schema: './schema/*.graphql'
extensions:
  codegen:
    generates:
      ./src/types.ts:
        plugins:
          - typescript
          - typescript-resolvers
```

在使用相关Graphql生态中的工具时，分享和复用这些配置时是非常方便的。

## GraphQL Playground

大多数人第一次接触Graphql的时候肯定接触过GraphQL Server提供的[Playground](https://github.com/graphql/graphql-playground). Playground是一个完整的浏览器端GraphQL IDE, 包括Documentation, Schema和一个可以编写和测试Graphql请求的编辑器。

![graphQL playground](img/graphql-playground.png "Playground")

对于使用graphql的开发团队, Playground基本代替了类似swagger的api文档和类似Postman的接口测试工具。是涉及接口较为完备的开发手册，而且0成本使用, Graphql 社区绝大多数Apollo Server的实现都会内置Playground。


## Apollo Client Devtool

![apollo client devtool](/img/devtool.jpg "apollo client devtool")

[Apollo Client Devtool](https://chrome.google.com/webstore/detail/apollo-client-devtools/jdkknkkbebbapilgoeccciglkfbmbnfm) 是 Apollo Client 提供的一款Chrome插件，可以在浏览器console中捕捉页面发送的query和mutation，并且提供了编辑，修改和重试这些请求，还有查看graphql缓存的功能。因为所有的graphql都是发送给统一的/graphql接口，query body也是字符串的形式可读性很差, Apollo Client Devtool 相较于通过传统的Networks方式来查看请求是比较方便的.

## Graphql Eslint

![graphql-eslint](img/eslint.gif "graphql eslint")

[graphql eslint](https://github.com/dotansimha/graphql-eslint) 支持graphql-config, 通过syntax检查和schema对比提供提示和报错，开发必备。

## GrahpQL VSCode Plugin

![graphql-plugn](img/plugin.gif "graphql plugin")

VScode Plugin [GraphQL](https://marketplace.visualstudio.com/items?itemName=GraphQL.vscode-graphql) 提供了相当齐全的功能，包括语法检查，语法高亮，引用跳转，Schema悬浮提示，autocomplete等。基本是一个必装的插件。

这里比较重要的一点是在定义Schema的时候对每一个参数所定义的描述（有的实现是通过代码注释的方式插入Schema）是可以通过这个插件的提示在IDE里展示的，也就是原来需要在前后两端都需要的注释，或者需要维护的api文档现在可以只用在Schema里定义一遍就可以了。

![graphql-plugin-4](img/plugin_4.png "graphql plugin")

![graphql-plugin-3](img/plugin_3.png "graphql plugin")

该Plugin还提供了一个可以在IDE内执行query和mutation的功能。

![graphql-plugin-1](img/plugin_1.png "graphql plugin")

但是个人认为实用性有限，对于不含参数的query查询可以直接执行，对于需要参数的query和mutation需要在vscode的弹窗中输入参数，如果是较为简单的字符串或者数字参数没有问题，但是如果是较为复杂的结构体就需要输入stringfy过后的JSON格式，并且很多情况下链式请求的参数来自于上一个请求，请求参数就更不好获取了。

![graphql-plugin-2](img/plugin_2.png "graphql plugin")

以上工具可以帮助我们更方便的使用和查询Graphql Schema, 但只有这些功能还是远远不够的，我们需要更多的工具将基于Schema提供的数据结构和Typescript结合起来，编写更健壮的前端代码。

## GraphQL Code Generator 

[GraphQL Code-Generator ](https://www.graphql-code-generator.com/) 是一个根据Schema生成类型文件的cli工具, 工具本身是通过preset和plugin的方式来实现针对不同语言和功能的需求，官方本身也提供了很丰富的plugin可以满足大部分的开发需求，自定义plugin也有较为详细的文档和示例。本文只讨论typescript和相关的plugin.

![graphql generator](img/generator.png "generator")

最核心的一点是code generator可以通过Schema生成完整的type文件。所以我们在使用graphql返回的数据类型的时候是有完整的类型的，很实用的一点类型描述也会在包含在类型文件里。

![graphql generator-1](img/generator-1.png "generator-1")

所以在VSCode Typescript插件的帮助下我们在使用`useQuery` hook返回的数据是是有完整数据类型提醒，autocomplete和定义Schema时对字段的具体描述的。这种方式在开发体验和效率上相比传统在项目和文档间来回切换提高了很多，也同时增强了代码的健壮性。

但是这种方式也存在一个问题，就是细心的朋友可能发现我们在定义query的时候只请求了events对象的name属性，但是自动提示却提示了events包含的所有可能字段，并没有完全实现我们的要求。

我们看一下`useQuery`方法的类型定义:

```ts
useQuery<TData = any, TVariables = OperationVariables>(query: DocumentNode | TypedDocumentNode<TData, TVariables>, options?: QueryHookOptions<TData, TVariables>): QueryResult<TData, TVariables>;
```

接受两个伪类 `TData` 和 `TVaraibles`，我们需要在使用hooks的时候传入这两个参数如果要根据query来自定义返回值我们可能需要在第一个参数传入类似被使用Pick, Exclude包装过后的类型，如果query或mutation有传入参数也需要从根路径的type.ts去引用Input的类型，非常的麻烦。

```ts
  { events: Pick<Event, 'name'>[] }
```

code-generator针对这个问题也有相应的plugin: `typescript-operations` 可以根据query文件生成对应的type.

```graphql
query GetEvents{
  events {
    id
    name
  }
}
```

```ts
export type GetEventsQueryVariables = SchemaTypes.Exact<{ [key: string]: never; }>;

// With Pick 
export type GetEventsQuery = { events?: SchemaTypes.Maybe<Array<SchemaTypes.Maybe<Pick<SchemaTypes.Event, 'id' | 'name'>>>> };

// Without Pick
export type GetEventsQuery = { events?: SchemaTypes.Maybe<Array<SchemaTypes.Maybe<{ id: string, name?: SchemaTypes.Maybe<string> }>>> };

```
对于使用React + Apollo Client的项目，`typescript-react-apollo` plugin可以生成和query相对应的获取数据的hook 包括封装好的useQuery, useLazyQuery等，可以在保证代码类型完整的前提下极大的提高开发效率。

![graphql generator-2](img/generator-2.png "generator-2")

Code generator的插件很多，配置项也很多，自定义plugin也比较容易，还是需要根据项目需要，配置出适合项目适合团队的自动化流程。

## GraphQL Introspection 

Graphql社区工具绝大部分是依赖GraphQL的 [Introspection](https://graphql.org/learn/introspection/), 在对graphql请求的根部永远是__schema节点，可以通过这个节点来查询整个Schema的内容。感兴趣的朋友也可以参考graphql官方和graphql-config来开发自定义工具。