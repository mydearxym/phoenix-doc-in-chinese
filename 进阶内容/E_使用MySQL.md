# 使用 MySQL

Phoenix 默认是使用的数据库是 PostgreSQL, 但如果我们想换成 MySQL 呢？ 在这票指南中，我们将学习怎样切换 -- 不管
是从一个新的应用，还是在一个已经存在的应用上做切换。

如果我们创建一个新的应用，要使用 MySQL 非常简单, 只需要在 `phoenix.new` 的时候使用 `--database mysql` 选项即可，
phoenix 会为我们完成剩下的工作。

```console
$ mix phoenix.new hello_phoenix --database mysql
```

当我们用 `mix deps.get` 安装完必要的依赖后，我们就可以在应用中使用 Ecto 了。

如果我们在一个已经存在的应用中，我们也只需要更换一个适配器并在配置上做一些小改动。

关于更换适配器，我们需要删除依赖项 (`mix.exs`) 中的 `Postgrex`, 然后添加新的适配器： `Mariaex` 。

```elixir
defmodule HelloPhoenix.Mixfile do
  use Mix.Project

  . . .
  # Specifies your project dependencies
  #
  # Type `mix help deps` for examples and options
  defp deps do
    [{:phoenix, "~> 1.1.0"},
     {:phoenix_ecto, "~> 2.0"},
     {:mariaex, ">= 0.0.0"},
     {:phoenix_html, "~> 2.3"},
     {:phoenix_live_reload, "~> 1.0", only: :dev},
     {:cowboy, "~> 1.0"}]
  end
end
```

我们同样需要将应用列表里的 `:postgrex` 用 `:mariaex` 来取代，代码同样位于 `mix.exs` 文件中。

```elixir
defmodule HelloPhoenix.Mixfile do
  use Mix.Project

. . .
def application do
  [mod: {MysqlTester, []},
  applications: [:phoenix, :cowboy, :logger,
  :phoenix_ecto, :mariaex]]
end
. . .
```

接下来，我们需要配置新适配器， 让我们修改 `config/dev.exs` 。

```elixir
config :hello_phoenix, HelloPhoenix.Repo,
adapter: Ecto.Adapters.MySQL,
username: "root",
password: "",
database: "hello_phoenix_dev"
```

如果我们是在一个已存在的应用中，只需要修改相对应的字段即可，只需要确保适配器是使用 MySQL 即可 ：`adapter: Ecto.Adapters.MySQL` 。
`config/test.exs` 和 `config/prod.secret.exs` 的配置与此类似。

现在我们需要安装依赖：

```console
$ mix do deps.get, compile
```
当安装完成并且配置完毕，我们就可以来创建数据库了。

```console
$ mix ecto.create
The database for HelloPhoenix.repo has been created.
```

我们可以执行迁移任务了，或者使用 Ecto 做任何事情。

```console
$ mix ecto.migrate
[info] == Running HelloPhoenix.Repo.Migrations.CreateUser.change/0 forward
[info] create table users
[info] == Migrated in 0.2s
```
