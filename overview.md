
### overview

[Phoneix](http://www.phoenixframework.org/) 是一个基于 [Elixir](http://elixir-lang.org/) 语言的遵循 MVC 模式的web开发框架。因此如果你有其他web开发框架比如 RoR 或者 Django的使用经验，其中的很多组件和概念会看起来很眼熟。

`Phonex` 融合了两种开发模式的精华 -- 惊人的开发效率和极高的应用性能。她同时还提供了一些有用的新功能，比如面向实时需求的`channels` 和为提升性能的模板预编译技术。

如果您已经熟悉 Elixir 语言，恭喜你，你有了一个不错的开始。如果您暂时还不熟悉，那么[The Elixir guides](http://elixir-lang.org/getting-started/introduction.html)是一个很好的起点。 在 `Learning Elixir and Erlang Guide` 里同样有不错的学习资源。

这篇文档的主要目的就是让您对 Phonenix 的里里外外有一个提纲挚领的了解。

##### Phonex
Phoneix 建立在一些模块化的，扩展灵活的组件之上，包括我们接下来要谈到的 `Plug` 和 `Ecto`。而更基础的 Erlang HTTP server, Cowboy 组件，本文档将不会涉及。

Phoenix 的各组件分工明确，职责清晰。我们将在接下来的章节中详细介绍他们，这里仅做一个快速鸟瞰：


handles all aspects of requests up until the point where the router takes over
provides a core set of plugs to apply to all requests
dispatches requests into a designated router

* The Endpoint
    *  处理请求到知道其到达路由部分。
    *  提供一些针对到达请求的公用核心的组件。
    *  将请求分发给指定的路由。

* 路由 (`the Router`)
    *  处理请求并分配给指定的控制器和动作(`controller/action`)。
    *  生成一些 `helpers` 方法提示路由路径和指向的资源。
    *  defines named pipelines through which we may pass our requests
    *  Pipelines
       * allow easy application of groups of plugs to a set of routes

* 控制器 (`Controllers`)
    *  提供处理请求的函数(`function`),也就是`action`
    *  动作(`Action`)
       *  组织数据并传递到视图(`view`)中
       *  触发渲染模板
       *  处理重定向

* 视图(`views`)
   * 渲染模板
   * 充当展示层
   * 定义可以在模板中使用的一些`helper`方法

* 模板(`Templates`)
   * 如名字所示。
   * 通常被预编译，很快。

* 通道(`Channels`)
   * 管理 sockets 让实时应用更容易。
   * 和控制器概念类似，另外她支持长连接的双向通讯。

* 订阅发布(`PubSub`)
   * 在通道(`Chnannels`)层的下面，允许客户端订阅某一主题。
   * 为第三方的订阅和发布提供统一的适配器。
