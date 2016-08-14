# 自定义错误

Phoenix 提供了一个 `ErrorView`, `web/views/error_view.ex`, 来渲染我们应用中的错误。完整的模块名会包含项目自身
的名字，比如： `HelloPhoenix.ErrorView` 。

Phoenix 会检测项目中任何 400 或者 500 状态码然后使用 `ErrorView` 中的 `render/2` 函数来渲染一个合适的错误模板。
默认只实现了 404 和 500 HTML 错误，但是我们可以根据需要增加 `render/2` 分支语句来实现我们自己的需要，如果错误都没
有命中那么就会执行 `template_not_found/2`。

我们可以根据需要自定义所有函数的实现，下面是 `ErrorView` 的内容：

```elixir
defmodule HelloPhoenix.ErrorView do
  use HelloPhoenix.Web, :view

  def render("404.html", _assigns) do
    "Page not found"
  end

  def render("500.html", _assigns) do
    "Server internal error"
  end

  # In case no render clause matches or no
  # template is found, let's render it as 500
  def template_not_found(_template, assigns) do
    render "500.html", assigns
  end
end
```

> 注意: 在开发环境下，这些行为会被改写以提供更多的调试信息。 如果想要看 `ErrorView` 我们需要将
> `config/dev.exs` 文件中的 `debug_errors` 设置成 `false` 。并且服务器重启才能生效。

```elixir
config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: 4000],
  debug_errors: false,
  code_reloader: true,
  cache_static_lookup: false,
  watchers: [node: ["node_modules/brunch/bin/brunch", "watch"]]
```

更多关于自定义错误页面的内容，请查看 [The Error View](http://www.phoenixframework.org/docs/views#section-the-errorview)。

#### 自定义错误

Elixir 有一个内建宏 `defexception` 用来自定义 `exceptions`, `Exceptions` 被表示成一个结构体，并且该结构体需要在模块内
部被定义。

为了创建一个自定义的错误，我们需要定义一个新的模块，根据惯例，名字里最好包含 'Error' , 在这个模块里我们需要使
用 `defexception` 来定义一个新的 `exception` 。

我们同样可以在模块中定义模块来为里面的模块提供命名空间。
(We can also define a module within a module to provide a namespace for the inner module.)

我们可以看看 [Phoenix.Router](https://github.com/phoenixframework/phoenix/blob/master/lib/phoenix/router.ex)
中的一个例子.

```elixir
defmodule Phoenix.Router do
  defmodule NoRouteError do
    @moduledoc """
    Exception raised when no route is found.
    """
    defexception plug_status: 404, message: "no route found", conn: nil, router: nil

    def exception(opts) do
      conn   = Keyword.fetch!(opts, :conn)
      router = Keyword.fetch!(opts, :router)
      path   = "/" <> Enum.join(conn.path_info, "/")

      %NoRouteError{message: "no route found for #{conn.method} #{path} (#{inspect router})",
      conn: conn, router: router}
    end
  end
. . .
end
```

Plug 专门为添加状态码给 exception 结构体提供了  `Plug.Exception` 协议。

如果我们想为 `Ecto.NoResultsError` 提供一个 404 的错误码，我们可以使用 `Plug.Exception` 协议实现如下：

```elixir
defimpl Plug.Exception, for: Ecto.NoResultsError do
  def status(_exception), do: 404
end
```

注意这里仅仅是一个例子: Phoenix
[已经实现了](https://github.com/phoenixframework/phoenix_ecto/blob/master/lib/phoenix_ecto/plug.ex)
 `Ecto.NoResultsError`。所以你并不需要这么做了。
