# 概述

一旦我们有了一个可工作的应用，我们就可以来部署了。或者你可以按照
[起步指南](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/A_%E8%B5%B7%E6%AD%A5.html)
章节来创建一个最基本的应用来继续本章的内容。

当我们准备部署应用的时候，主要有三个关注点：
  * 处理你应用中的私有配置文件
  * 编译压缩你的应用资源
  * 在生产模式下运行的服务器

这些和你手上的开发基础设施紧密相关。具体来讲，我们有 [Heroku](http://www.phoenixframework.org/docs/heroku) 和
[高级部署指南](http://www.phoenixframework.org/docs/advanced-deployment) 使用 Erlang 类型的部署。 在每种情况下，
本章节的中心思想将会有助于你。

让我们一步一步的来。

> 注意: 这篇指南假设你使用的是最新的 Elixir v1.0.4 , 这个版本对项目在生产模式下的编译和优化做了不少改进。


## 处理私有配置文件

所有的的 Phoenix 应用都有需要保密数据/配置，比如，的生产环境数据库的用户名密码，以及 phoenix 用来给重要信息加
密的 sign 等等。这种类型的信息被保存在 `config/prod.secret.exs` 并且默认不被添加到你的代码仓库中。

接下来我们要在生产环境下使用这些配置，下面是新项目默认生成的 `config/prod.secret.exs` 文件的内容：

```elixir
use Mix.Config

# In this file, we keep production configuration that
# you likely want to automate and keep it away from
# your version control system.

# You can generate a new secret by running:
#
#     mix phoenix.gen.secret
config :foo, Foo.Endpoint,
  secret_key_base: "A LONG SECRET"

# Configure your database
config :foo, Foo.Repo,
  adapter: Ecto.Adapters.Postgres,
  username: "postgres",
  password: "postgres",
  database: "foo_prod",
  size: 20 # The amount of database connections in the pool
```

将这些配置数据引入生产环境有几种方法，其中之一是将这里的配置的值用环境变量替代，然后在命令行启动生产服务器的时
候将这些值传递进去。我们在 [in the Heroku guides](http://www.phoenixframework.org/docs/heroku) 就是这么干的。

另一个办法是将这个文件与我们的生产环境的代码隔离开，比如放在 "/var/config.prod.exs" ，这样我们需要在
`config/prod.exs` 文件中使用 `import_config` 引入这个文件。

```elixir
import_config "/var/config.prod.exs"
```

这样我的私有信息就基本安全了。现在我们来配置资源。
开始之前还有一些准备工作，因为我们只讨论生产环境，我们需要安装以及编译生产环境下的依赖。

```console
$ mix deps.get --only prod
$ MIX_ENV=prod mix compile
```

## 编译应用资源

这一步只有当你的应用中有静态资源是才需要，包括图片，js代码，样式表以及类似的文件。默认情况下，Phoenix 使用
Brunch， 我们这里也使用 Brunch 。

编译资源分为两步：

```console
$ brunch build --production
$ MIX_ENV=prod mix phoenix.digest
Check your digested files at "priv/static".
```

这样就完了！ 第一行编译静态文件，第二行生成摘要(digests)以及 manifest 文件, 以便于 Phoenix 在生产环境下 serve
这些资源。

要注意的是，如果你没有运行上面两个两步，Phoenix 会报错：

```console
$ PORT=4001 MIX_ENV=prod mix phoenix.server
10:50:18.732 [info] Running MyApp.Endpoint with Cowboy on http://example.com
10:50:18.735 [error] Could not find static manifest at "my_app/_build/prod/lib/foo/priv/static/manifest.json". Run "mix phoenix.digest" after building your static files or remove the configuration from "config/prod.exs".
```

错误提示很清晰： Phoenix 没有找到静态 manifest 文件。你可以运行上面的命令来 fix 它，或者如果你的应用不关心静态
资源， 你可以在 `config/prod.exs` 文件中删除 `cache_static_manifest` 配置。

## 在生产环境下启动服务

我们可以设置 `PORT` 和 `MIX_ENV` 环境变量让 Phoenix 运行在生产环境下：

```console
$ PORT=4001 MIX_ENV=prod mix phoenix.server
10:59:19.136 [info] Running MyApp.Endpoint with Cowboy on http://example.com
```

如果出现了错误信息，确保你是按照文档做了，如果还不对，请报告一个 issue 。

我们也可以在交互式 shell 中运行服务器：

```console
$ PORT=4001 MIX_ENV=prod iex -S mix phoenix.server
10:59:19.136 [info] Running MyApp.Endpoint with Cowboy on http://example.com
```

或者你可以将其作为独立的 daemon 模式运行：
(Or run it detached from the iex console. This effectively daemonizes the process so it can run
independently in the background):

```elixir
MIX_ENV=prod PORT=4001 elixir --detached -S mix do compile, phoenix.server
```

这种模式下即便我们关闭 shell 应用服务器依然在运行。

## 总结

之前的章节概述了部署 Phoenix 应用的主要步骤，在实际环境下，你可能还需要一些额外的步骤。比如，如果你使用了数据
库，你还需要在启动服务器之前运行 `mix ecto.migrate` 来确保数据库已经就绪。

不管怎样，下面这个脚本是一个不错的开始：

```elixir
# Initial setup
$ mix deps.get --only prod
$ MIX_ENV=prod mix compile

# Compile assets
$ brunch build --production
$ MIX_ENV=prod mix phoenix.digest

# Custom tasks (like DB migrations)
$ MIX_ENV=prod mix ecto.migrate

# Finally run the server
$ PORT=4001 MIX_ENV=prod mix phoenix.server
```
