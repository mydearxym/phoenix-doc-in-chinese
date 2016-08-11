# 会话 (Session)

Phoenix 从 Plug 中继承了`会话` (Session) 功能。Plug 支持两种会话存储方式 -- cookies 和 Erlang Term Storage (ETS)。

## Cookies

Phoenix 默认使用 Plug 的 cookie 会话存储。我们需要在两个地方配置 `secret_key_base` 来使其正常工作 -- 在
`config/config.exs` 以及在 endpoint 配置 `Plug.Session` 。

下面是刚生成的 Phoenix 应用的 `config/config.exs` 文件内容的 `secret_key_base` 部分。

```elixir
config :hello_phoenix, HelloPhoenix.Endpoint,
  url: [host: "localhost"],
  root: Path.dirname(__DIR__),
  secret_key_base: "some_crazy_long_string_phoenix_generated",
  debug_errors: false,
  pubsub: [name: HelloPhoenix.PubSub,
           adapter: Phoenix.PubSub.PG2]
```

Plug 使用 `secret_key_base` 的值来给每一个 cookie 签名，以防止其被篡改。

下面是 `lib/hello_phoenix/endpoint.ex` 中 `Plug.Session` 的默认配置:

```elixir
defmodule HelloPhoenix.Endpoint do
  use Phoenix.Endpoint, otp_app: :hello_phoenix
. . .
  plug Plug.Session,
    store: :cookie,
    key: "_hello_phoenix_key",
    signing_salt: "Jk7pxAMf"
. . .
end
```

## ETS

Phoenix 同时通过 `ETS` 支持服务端的会话管理。 为了配置 ETS 会话， 我们需要在应用启动的时候创建一个 ETS 表。我
们还需要重新配置一下  `Plug.Session`。

这里展示了我们怎样在 `lib/hello_phoenix.ex` 文件中在应用启动的时候创建 ETS 表。

```elixir
def start(_type, _args) do
  import Supervisor.Spec, warn: false
  :ets.new(:session, [:named_table, :public, read_concurrency: true])
. . .
```

为了重新配置 `Plug.Session`, 我们需要需要改变 `store` 字段，明确我们使用的是 ETS 表，并且明确声明我们存储会话
的表名。 对于 ETS 来说， `secret_key_base` 是不必须的。

这是在 `lib/hello_phoenix/endpoint.ex` 中的配置：

```elixir
defmodule HelloPhoenix.Endpoint do
  use Phoenix.Endpoint, otp_app: :hello_phoenix
  . . .
  plug Plug.Session,
    store: :ets,
    key: "sid",
    table: :session,
  . . .
end
```

尽管我们可以使用 ETS 作为会话存储，但这并不是最好的选择，下面内容节选自 `Plug.Session` 文档：

> 我们并不推荐将 ETS 作为会话存储在生产环境下使用，因为每次会话会存储在 ETS 但却没有清理机制，除非我们专门创建
> 一个任务来做这件事情。

> 同时，ETS 是在内存中的，这意味着它不能再多个服务器中被共享， 所以，当部署在多个服务器中时，并不是一个好的选择。

## 获取会话数据

当配置无误后，我们就可以在控制器中获取会话数据了。

这里是一个简单的设置一个回话数据并将其取回的例子。

我们可以像下面例子中那样在 `web/controllers/page_controller.ex` 文件中改变 `HelloPhoenix.PageController` 控制
器的 `index` action, 来让会话信息在浏览器服务器之间来回穿越一下。

```elixir
defmodule HelloPhoenix.PageController do
  use HelloPhoenix.Web, :controller

  def index(conn, _params) do
    conn = put_session(conn, :message, "new stuff we just set in the session")
    message = get_session(conn, :message)

    text conn, message
  end
end
```
