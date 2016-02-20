
### Plug

`Plug` 运行在 phoenix 的 HTTP 核心层，我们在处理连接的每个生命周期都和 Plugs 打交道，Phoenix 的 Endpoints, 路由，控制器也都属于 Plug 的范畴，下面我们就来看看神奇的 Plug。

[Plug](https://github.com/elixir-lang/plug) 是一个在不同web应用中编写可组合模块的一套规格规范，她同时是不同 web server 的连接中间层。Plug 的基本概念就是统一"连接"的概念，这和 Rack (rails 的中间件) 那种把请求和响应分开的中间件是不同的。


#### Plug 规范（The Plug Specification）

简单的来说，Plug 规范包括两种类型, `函数 plugs` 和 `模块 plugs`

### 函数 Plugs (Function Plugs)
一个符合 plug 的函数需要接受一个连接结构体( %Plug.Conn{}) 和相关选项作为参数，这个函数还需返回这个连接结构体，任何符合这个规范的函数都可以，让我们看一个例子：

```elixir
def put_headers(conn, key_values) do
  Enum.reduce key_values, conn, fn {k, v}, conn ->
    Plug.Conn.put_resp_header(conn, k, v)
  end
end
```

很直观，对吧？
我们就是用这种方法在 Phoenix 的连接上进行一系列的操作的。

```elixir
defmodule HelloPhoenix.MessageController do
  use HelloPhoenix.Web, :controller

  plug :put_headers, %{content_encoding: "gzip", cache_control: "max-age=3600"}
  plug :put_layout, "bare.html"

  ...
end
```
通过遵循 plug 的规范， `put_headers/2`, `put_layout/2`甚至 `action/2`都可以把一个请求转换成一个像流水线一样的流程，非常的高效，让我们来看一个例子：假设有一个场景，我们需要对请求做一系列的验证逻辑，并根据结果要么重定向，要么禁止访问，我们可以这样写（常规写法）：

```elixir
defmodule HelloPhoenix.MessageController do
  use HelloPhoenix.Web, :controller

  def show(conn, params) do
    case authenticate(conn) do
      {:ok, user} ->
        case find_message(params["id"]) do
          nil ->
            conn |> put_flash(:info, "That message wasn't found") |> redirect(to: "/")
          message ->
            case authorize_message(conn, params["id"])
              :ok ->
                render conn, :show, page: find_message(params["id"])
              :error ->
                conn |> put_flash(:info, "You can't access that page") |> redirect(to: "/")
            end
        end
      :error ->
        conn |> put_flash(:info, "You must be logged in") |> redirect(to: "/")
    end
  end
end
```
你可能已经看到，仅仅是几步权限和验证的工作，代码已经充满了嵌套和重复了。让我们用几个 plugs 来重构一下吧：


```elixir
defmodule HelloPhoenix.MessageController do
  use HelloPhoenix.Web, :controller

  plug :authenticate
  plug :find_message
  plug :authorize_message

  def show(conn, params) do
    render conn, :show, page: find_message(params["id"])
  end

  defp authenticate(conn, _) do
    case Authenticator.find_user(conn) do
      {:ok, user} ->
        assign(conn, :user, user)
      :error ->
        conn |> put_flash(:info, "You must be logged in") |> redirect(to: "/") |> halt
    end
  end

  defp find_message(conn, _) do
    case find_message(params["id"]) do
      nil ->
        conn |> put_flash(:info, "That message wasn't found") |> redirect(to: "/") |> halt
      message ->
        assign(conn, :message, message)
    end
  end

  defp authorize_message(conn, _) do
    if Authorizer.can_access?(conn.assigns[:user], conn.assigns[:message]) do
      conn
    else
      conn |> put_flash(:info, "You can't access that page") |> redirect(to: "/") |> halt
    end
  end
end
```
将原先嵌套的代码块用 plug 扁平化以后，我们可以让这些功能更加的模块化，更干净，更好的被复用。
（译者： 请求在到达在这个 controller 的里的每个action时，都要顺序执行顶部列出的 plug 代码，相当于一个过滤层。）

现在，让我们看看 plug 的另一种类型： 模块 plugs （module plugs）.

### 模块 Plugs 

模块 plugs 是另一种类型的 Plug 实现，通常我们把她定义在一个模块里，并且需要实现两个函数接口：

* `init/1` 用于初始化传给 `call/2` 的参数或者选项 
* `call/2` 处理 connection 的转换工作， 和我们之前看到的 Plug 没什么两样。

举个例子，让我们写一个模块 plug ，功能是把 `:locale` 键值对放到连接流里，以便让后面的其他 plugs， 控制器和页面等也能使用。

```elixir
defmodule HelloPhoenix.Plugs.Locale do
  import Plug.Conn

  @locales ["en", "fr", "de"]

  def init(default), do: default

  def call(%Plug.Conn{params: %{"locale" => loc}} = conn, _default) when loc in @locales do
    assign(conn, :locale, loc)
  end
  def call(conn, default), do: assign(conn, :locale, default)
end

defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
    plug HelloPhoenix.Plugs.Locale, "en"
  end
  ...
```

我们可以在 brower pipeline 代码块中添加一句 `plug HelloPhoenix.Plugs.locale, "en"`来启用她。在`init/1`函数中，我们传递一组默认的 locala。我们还在`call/2`运用了模式匹配去设置 locale,如果匹配失败则默认回滚到 "en".

Plug 就介绍到这里。Phoenix框架中大量运用了 plug 的思想，值得我们好好揣摩。
