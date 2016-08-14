# 填充数据

在我们创建应用的时候,一个很常见的需求是能给我们的数据库一些初始数据, 比如在开发模式或者测试模式的时候。

不出意外的， Phoenix 已经提供给了我们一个解决办法, Phoenix 默认情况下会生成一个脚本 `priv/repo/seeds.exs`, 我
们可以使用它来填充我们的数据库。

注意在填充数据之前，我们要先执行完迁移任务并且根据
[Ecto 模型指南](http://www.phoenixframework.org/docs/ecto-models) 提到的更新我们的 `router.ex` 文件。

我们可以在 `seeds.exs` 编写代码来向数据库中填充数据， `seeds.exs` 已经在文件中为我们生成了模板：

```elixir
  <%= application_module %>.Repo.insert!(%<%= application_module %>.SomeModel{})
```

比如，我们创建了一个叫 Linker 的应用并想在 Link 表中添加一些数据 (links), 我们可以简单的在 `seeds.exs` 中添加
如下代码：

```elixir
  ...
  alias Linker.Repo
  alias Linker.Link

  Repo.insert! %Link{
    title: "Phoenix Framework",
    url: "http://www.phoenixframework.org/"
  }

  Repo.insert! %Link{
    title: "Elixir",
    url: "http://elixir-lang.org/"
  }

  Repo.insert! %Link{
    title: "Erlang",
    url: "https://www.erlang.org/"
  }
  ...
```

当我们运行这段脚本时，就会在数据库中插入数据。

```elixir
  mix run priv/repo/seeds.exs
```

注意如果我们想删除所有先前的 Link 表中的填充数据, 我们可以在 `Repo.insert!` 之前添加 `Repo.delete_all Link`。

我们还可以创建一个模块来填充我们的数据。 这样做的优势是可以在 IEx 中直接创建，并且让代码更加的模块化。
比如:

```elixir
defmodule <%= application_name %>.DatabaseSeeder do
  alias <%= application_name %>.Repo
  alias <%= application_name %>.Link

  @titles_list ["Erlang", "Elixir", "Phoenix Framework"] // list of titles
  @links_list ["http://www.erlang.org", "http://www.elixir-lang.org", "http://www.phoenix-framework.org"] // list of links

  def insert_link do
    Repo.insert! %Link{
      title: (@titles_list |> Enum.take_random),
      url: (@urls_list |> Enum.take_random)
    }
  end

  def clear do
    Repo.delete_all
  end
end

(1..100) |> Enum.each(fn _ -> <%= application_name %>.DatabaseSeeder.insert_link end)
```

现在，我们可以直接在 IEx 中向我们的数据库中添加数据了。

```elixir
$ iex -S mix
iex(1)> <%= application_name %>.DatabaseSeeder.add_link
iex(2)> <%= application_name %>.Link |> <%= application_name %>.Repo.all
#=> [%<%= application_name %>.Link{...}]
```

在很多情况下，在开发是直接使用 IEx 来干这事是很方便的。

#### Models are Initialized

遵循这里的惯例，Phoenix 会确保我的 Model 被正确的初始化。 并且只要我们使用带感叹号的函数 (bang functions) , 比
如`insert!`, `update!` 等，他们会在失败的时候报错。

这个很有用，如果我们在编码出错时（比如试图给数据表里的一个唯一字段添加重复数据），数据库也不会失去其一致性。数
据库会拒绝执行请求，并且 Ecto 会抛出一个错误，比如：

```elixir
  ** (Ecto.ConstraintError) constraint error when attempting to insert model:
```

在开发环境下，我们同时可以更改我们的脚本让其在执行 bang functions 之前检查出错误，以防止失败。
