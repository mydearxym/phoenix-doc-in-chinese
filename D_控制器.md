Phoenix 控制器是一个中间人模块，里面的函数称为 atcion ,它响应路由的 HTTP 请求，收集必要的数据并处理 view 层渲染模板或返回 JSON 数据，

Phoenix 控制器同时建立在 Plug 包基础上，也就是它们自己的 plugs, 控制器几乎提供了我们在编写 action 中所需要的所有工具，如果我们发现某一项功能是 Phoenix 控制器没有提供的，我们可能就要去 Plug 中自己寻找了，请查阅 [Plug Guide](http://www.phoenixframework.org/docs/understanding-plug) 或者 [Plug Documentation](http://hexdocs.pm/plug/)。

一个新生成的 Phoenix 应用会包含一个简单的控制器---`PageController`,我们可以在 `web/controllers/page_controller.ex` 找到它，内容如下：


```elixir
defmodule HelloPhoenix.PageController do
  use HelloPhoenix.Web, :controller

  def index(conn, _params) do
    render conn, "index.html"
  end
end
```


在定义模块下方的第一行 `use ...` 触发了`HelloPhoenix.Web` 模块的 `__using_/1` 宏， 它会引入一些有用的模块。

`index` action 用来根据在路由中定义的默认规则显示 Phoenix 的欢迎页面。

### Actions

控制器中的 actions 只是普通的函数，我们可以将它命名为符合 Elixir 命名规则的任何名字。 唯一要求是我们必须满足 action 名字和路由中定义的 route 相对应。

比如，在`web/router.ex`中，我们可以将默认路由中的 action 名字改一下：

```elixir
get "/", HelloPhoenix.PageController, :index
```

改成 test:

```elixir
get "/", HelloPhoenix.PageController, :test
```

只要我们同样将 `PageController ` 中的 action 名字改成 `test`, 欢迎页面仍旧会正常显示。


```elixir
defmodule HelloPhoenix.PageController do
  . . .

  def test(conn, _params) do
    render conn, "index.html"
  end
end
```

尽管我们可以将 action 命名成任何我们想到的名字，但我们还是应当尽量遵循一些惯例，我们在[Routing Guide](http://www.phoenixframework.org/docs/routing)提到过，现在再来快速回顾一下：

- index   - 按照给定的数据源渲染一组条目。
- show    - 渲染一个给定id的独立条目。
- new     - 渲染一个创建新条目所需的表单。
- create  - 接收创建的新条目并将它存储起来。
- edit    - 接收给定id的条目，并将其显示在form中以供编辑。
- update  - 接收修改过的 item 并存储起来。
- delete  - 接收给定id的条目并将其从数据库中删除。

每个 actions 需要两个参数，Phoenix 会为我们自动填充。

第一个参数是 `conn`----一个存储着请求信息的结构体，包括但不限于域名(host) , path element, 端口(port)，请求字符串(query string).. `conn`来源于 Elixir 的 Plug 中间件，更多请查看：
[plug's documentation](http://hexdocs.pm/plug/Plug.Conn.html).


The second parameter is `params`. Not surprisingly, this is a map which holds any parameters passed along in the HTTP request. It is a good practice to pattern match against params in the function signature to provide data in a simple package we can pass on to rendering. We saw this in the [Adding Pages guide](http://www.phoenixframework.org/docs/adding-pages) when we added a messenger parameter to our `show` route in `web/controllers/hello_controller.ex`.

第二个参数是 `params`. 字如其意，它对HTTP请求中的所有参数都做了映射，这里的最佳实践是
通常的做法是使用模式匹配将 params 里需要的字段提取出来以方便之后渲染的需要。比如我们之前在 [Adding Pages guide](http://www.phoenixframework.org/docs/adding-pages) 中，当我们添加一个messenger 参数路由中的 `show` 中时(`web/controllers/hello_controller.ex`).

```elixir
defmodule HelloPhoenix.HelloController do
  . . .

  def show(conn, %{"messenger" => messenger}) do
    render conn, "show.html", messenger: messenger
  end
end
```

在某些案例中---通常是在 `index` actions中，我们并不关心请求参数因为我们的输出并不依赖于它们，在这种情况下，我们只需在变量前添加一个下划线前缀，`_params`，这样编译器就不会抱警告信息了。


### （收集数据）Gathering Data


尽管 Phoenix 没有自带数据管理层，但 Elixir 工程 [Ecto](http://hexdocs.pm/ecto) 提供了一个很好的解决方案，尤其对于使用 [Postgres](http://www.postgresql.org/) 关系型数据库。（已有计划提供其他数据库的适配器），我们已经在[Ecto Models Guide](http://www.phoenixframework.org/docs/ecto-models) 讲解了如何使用在 Phoenix 项目中使用 Ecto.

当然了，关于数据层(data access)还有其他很多的选项，[Ets](http://www.erlang.org/doc/man/ets.html) 和 [Dets](http://www.erlang.org/doc/man/ets.html) 是 [OTP](http://www.erlang.org/doc/) 内建的键-值数据库。 `OTP` 同时提供关系型数据库`mnesia`, 它同时有自己的查询语言 `QLC`.  Elixir / Erlang 同时有众多的库支持现在流行的数据存储方案。

你可以随意选择，但我们不会详细讨论这些话题了。


### Flash Messages

很多时候我们需要在 action 的处理过程中和用户沟通，比如在更新模型的时候有错误，又或者我们需要在应用中显示欢迎信息等等，这时，我们需要 flash messages.

`Phoenix.Controller` 模块提供 `put_flash/3` 和 `get_flash/2` 函数帮助我们通过 `键值对` 的方式生成和获取 flash messages ，让我们来给 `HelloPhoenix.PageController` 生成两条 flash messages 来一探究竟。

在 `index` action 中我们修改如下。

```elixir
defmodule HelloPhoenix.PageController do
  . . .
  def index(conn, _params) do
    conn
    |> put_flash(:info, "Welcome to Phoenix, from flash info!")
    |> put_flash(:error, "Let's pretend we have an error.")
    |> render("index.html")
  end
end
```

`Phoenix` 的 controller 模块并不限制我们在 flash 中使用的 keys 的名字，你只需要遵守正常的命名规则就可以了，但是，一般来讲 `:info` 和 `:error` 之类的名字比较常见。


为了看到我们的 flash messages, 我们需要在 template/layout 中接收并显示他们，其中一种方法是使用 `get_flash/2`，它需要两个参数: `conn` 和我们关心的 flash 信息的 key， 它会返回这个 key 所对应的值。

幸运的是，我们应用的 layout ：`web/templates/layout/app.html.eex`, 已经包含了显示 flash messages 的相关代码了。

```html
<p class="alert alert-info" role="alert"><%= get_flash(@conn, :info) %></p>
<p class="alert alert-danger" role="alert"><%= get_flash(@conn, :error) %></p>
```

当我们刷新[Welcome Page](http://localhost:4000/), 我们的flash message 信息应该出现在 "Welcom to Phoenix" 上方。

除了 `put_flash/3` 和 `get_flash/2`, `Phoenix.Controller` 模块还有其他有用的函数值得了解：  `clear_flash/1` (需要 conn 参数) , 它删除 session 中存储的任何 flash messages。


### 渲染 Rendering

控制器有一些方法渲染内容，最简单的一种是使用 Phoenix 提供的 `text/2` 方法渲染纯文本。

比如说我们有个 `show` action , 它从参数映射 ( 原文: params map ) 里接收一个 id , 我们只简单的返回这个 id, 我们可以这样写：

```elixir
def show(conn, %{"id" => id}) do
  text conn, "Showing id #{id}"
end
```

假设我们把这个 `show` action 绑定给路由 `get "/our_path/:id"`，当我们用浏览器访问 `/our_path/15`, 就会看到纯文本内容 `Showing id 15`。

进一步的，我们可以用 `json/2` 渲染纯 JSON 内容，我们需要用  [Poison library](https://github.com/devinus/poison) 解析成 JSON。（Posion 是 Phoenix 的依赖项之一）。

```elixir
def show(conn, %{"id" => id}) do
  json conn, %{id: id}
end
```

现在我们在浏览器再次访问 `our_path/15`，我们可以看到一个 JSON 代码块：

```elixir
{
  id: "15"
}
```

Phoenix 控制器同样可以不需要模板渲染出 HTML 内容，你可能已经猜到，这个函数是 `html/2`, 现在，我们把 `show` action 重写如下：

```elixir
def show(conn, %{"id" => id}) do
  html conn, """
     <html>
       <head>
          <title>Passing an Id</title>
       </head>
       <body>
         <p>You sent in id #{id}</p>
       </body>
     </html>
    """
end
```

现在访问 `/our_path/15` 会渲染我们刚才在 `show` 中定义的 HTML 字符串， 注意我们不是使用`eex` 模板，这是一个多行字符串，所以我们使用的是插值字符串语法 `#{id}` 而不是模板的语法 `<%= id %>`

注意 `text/2`, `json/2` 以及 `html/2` 函数在渲染操作时都不需要 view 或者 template。

对写 APIs 来说 `json/2` 函数非常方便，另外两个函数也是方便的工具，但是根据我们传入的参数渲染到指定的模板是最常见的用法。

Phoenix 为我们提供了 `render/3` 函数。

有趣的是， `render/3` 是在 `Phoenix.View` 模块而不是在 `Phoenix.Controller` 中定义的，但是为了方便, 在 `Phoenix.Controller` 中也提供别名形式访问。

我们已经在[Adding Pages Guide](http://www.phoenixframework.org/docs/adding-pages) 中见到过 render 函数,  我们的 `show` action (web/controllers/hello_controller.ex) 内容入下：

```elixir
defmodule HelloPhoenix.HelloController do
  use HelloPhoenix.Web, :controller

  def show(conn, %{"messenger" => messenger}) do
    render conn, "show.html", messenger: messenger
  end
end
```

为了让 `render/3` 函数正确的工作，有几点需要特别注意： 控制器必须和一个单独的 view 取同一个名字，独立的 view 也必须有个同样名字的模板目录，并且里面包含 `show.html.eex`, 也就是说 `HelloController` 需要 `HelloView`, 另外 `HelloView` 需要项目中存在 `web/templates/hello` 目录，并且里面有 `show.html.eex`。

`render/3` 同时会将 `show` action 收到的哈希参数传递到模板里。

除了像上面例子那样传递字典以外，我们还可以使用 `Plug.Conn.assign/3`, 它会方便的返回 `conn`。


```elixir
def index(conn, _params) do
  conn
  |> assign(:message, "Welcome Back!")
  |> render("index.html")
end
```

注意： `Phoenix.Controller` 模块导入了 `Plug.Conn` , 所以我们可以直接调用 `assign/3`。

我们可以在 `index.html.eex` 模板或者布局（layout）中用 `<%= @message %>` 访问 message。

传递两个以上的参数可以用  pipeline 的形式将 `assign/3` 串联起来.


```elixir
def index(conn, _params) do
  conn
  |> assign(:message, "Welcome Back!")
  |> assign(:name, "Dweezil")
  |> render("index.html")
end
```

这样，`@message` 和 `@name` 在 `index.html.eex` 中都可以被放问到。

如果我们想构建一个欢迎信息(原文: if we want to plug `assign_welcome_message`)，可以被一些的 action 重写，这也很简单，我们可以这样写：

```elixir
plug :assign_welcome_message, "Welcome Back"

def index(conn, _params) do
  conn
  |> assign(:name, "Dweezil")
  |> render("index.html")
end

defp assign_welcome_message(conn, msg) do
  assign(conn, :message, msg)
end
```

如果我们只想在 `index` 和 `show` action 应用这个欢迎信息，我们可以这么写：

```elixir
defmodule HelloPhoenix.PageController do
  use HelloPhoenix.Web, :controller

  plug :assign_welcome_message, "Hi!" when action in [:index, :show]
. . .
```


### 直接返回响应(Sending responses directly)


如果以上的选项还不能满足你，我们可以使用 Plug 提供的一些函数组合起来去满足我们的需求。比如说我们想发送一个 "201" 状态并且 body 内容为空，我们可以使用 `send_resp/3`。


```elixir
def index(conn, _params) do
  conn
  |> send_resp(201, "")
end
```

刷新 [http://localhost:4000](http://localhost:4000) 会看到一个空页面，而浏览器的开发者工具上会显示一个 “201” 状态码。

如果我们想进一步指定响应内容的类型，我们可以用 `put_resp_content_type/2` 结合 `send_resp/3`  一起使用


```elixir
def index(conn, _params) do
  conn
  |> put_resp_content_type("text/plain")
  |> send_resp(201, "")
end
```

像这样，我们可以是使用 Plug 提供的函数组合出我们的需求。

渲染过程并不随 template 的结束而结束，默认情况下， template 的渲染结果会被插入到 layout 中，后者同样会被渲染。

[Templates and layouts](http://www.phoenixframework.org/docs/templates) 有完整的介绍，我们不详细展开了，接下来我们将看看如何在控制器中指定不同的布局（layout）。


### 指定布局（Assigning Layouts）

布局（Layout）是模板（templates）的特殊子集，我们可以在 `/web/templates/layout` 中找到它们，我们创建应用时 Phoenix 会自动为我们创建一个，叫 `app.html.eex` , 所有的模板默认都会按照这个布局（layout）进行渲染。

布局和模板没什么不一样，他们需要一个 view 去渲染他们，也就是在`web/views/layout_view.ex` 中 `LayoutView`模块。因为 Phoenix 我们自动生成了，我们不必自己创建，只要我们把布局（Layouts）放置在 `/web/templates/layouts` 目录中就行了。

在我们创建新的布局之前，让我们看看最简单的没有布局的模板。

`Phoenix.Controller` 模块提供了 `put_layout/2` 函数来切换布局 ( 原文： switch layout )。

该函数接收两个参数，一个 `conn` 另一个是布局的名称，如果传入 `false` 则表示不需要 layout 。

在新产生的 Phoenix 应用中，编辑 `PageController` 模块的 `index` action ( 在 web/controller/page_controller.ex ), 使其看起来如下：

```elixir
def index(conn, params) do
  conn
  |> put_layout(false)
  |> render "index.html"
end
```

在浏览器中刷新 [http://localhost:4000/](http://localhost:4000/) 后, 我们会看到一个不同的页面，没有标题，logo图片，以及 css 样式。

特别注意！在 pipeline 管道中调用函数时，比如上面的 `put_layout/2` 一定要用使用圆括号方式，否则会导致奇怪的错误。


如果我们看到这样错误。

```text
**(FunctionClauseError) no function clause matching in Plug.Conn.get_resp_header/2

Stacktrace

    (plug) lib/plug/conn.ex:353: Plug.Conn.get_resp_header(false, "content-type")
```
我们首先要检查是否正确使用了圆括号的函数调用方式。

这是正确的：

```elixir
def index(conn, params) do
  conn
  |> put_layout(false)
  |> render "index.html"
end
```

这是错误的：

```elixir
def index(conn, params) do
  conn
  |> put_layout false
  |> render "index.html"
end
```

现在，让我们实际创建一个布局（layout）并将 index 模块渲染到其中。比如说我们针对管理员有一个不同的布局 ，这个布局不会有 logo 图像。我们在`web/templates/layout` 目录中复制一份已存在的 `app.html.eex`到 `admin.html.eex`, 然后删除显示 logo 的代码段。

```html
<span class="logo"></span> <!-- remove this line -->
```

然后，在文件 `web/controllers/page_controller.ex` 中将布局（layout）的名字传递给 `put_layout/2` 函数


```elixir
def index(conn, params) do
  conn
  |> put_layout("admin.html")
  |> render "index.html"
end
```

### 复写渲染格式 ( Overriding Rendering Formats )

通过模板渲染 HTML 内容没有问题，但是如果我们需要动态的改变输出类型该怎么办？ 比如说有时候我们需要 HTML， 有时需要纯文本，有的时候需要 JSON 数据，怎么处理？

Phoenix 允许我们使用 `_format` 请求字符串动态的改变渲染类型。这需要在相同目录下存在符合规范的 视图（view ）和模板（templates）。

我们新建一个 app 作为例子， 默认的 `PageController` 渲染 html 页面如下：

```elixir
def index(conn, _params) do
  render conn, "index.html"
end
```

我们在相同的目录添加 `/web/templates/page/index.text.eex`. 内容如下：

```elixir
"OMG, this is actually some text."
```

要让这个例子工作正常，我们还需要告诉路由需要接收 `text` 格式，具体做法是在 `:broswer` pipeline 中同时添加 `html` 和 `text` 字符串，如下所示；

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html", "text"]
    plug :fetch_session
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end
. . .
```

我们同样需要告诉控制器按照 `Phoenix.Controller.get_format/1` 返回的格式去渲染模板，具体的做法是将原来的字符串版本的 `"index.html"` 改成`:index` 的原子版本。

```elixir
def index(conn, _params) do
  render conn, :index
end
```

这时我们访问 [http://localhost:4000/?_format=text](http://localhost:4000/?_format=text) ， 就会看到 `OMG, this is actually some text.`

当然了，我们也可以像其中传递数据，让我们将 `_params` 改成 `params` 以便接收参数，现在将 action 修改如下：

```elixir
def index(conn, params) do
  render conn, "index.text", message: params["message"]
end
```

再修改一下我们 text 版本的模板：

```elixir
"OMG, this is actually some text." <%= @message %>
```

现在，当我们访问  `http://localhost:4000/?_format=text&message=CrazyTown`, 我们将看到 "OMG, this is actually some text. CrazyTown"

### 指定内容类型（Setting the Content Type）

与 `_format` 查询参数相似的，我们通过改变 HTTP 接收头 ( HTTP Accepts Header ) 并提供相应的模板来渲染我们想要的任何格式的数据。

如果我们想要渲染一个 xml 格式的 `index` action, 我们可以在 `web/page_controller.ex` 中实现如下：

```elixir
def index(conn, _params) do
  conn
  |> put_resp_content_type("text/xml")
  |> render "index.xml", content: some_xml_content
end
```

然后我们需要创建一个提供合法 xml 数据格式的 `index.xml.eex` 模板文件。这样就可以了。

关于哪些是合法的 content mime-types, 请查看 Plug 中间件中的 [mime.types](https://github.com/elixir-lang/plug/blob/master/lib/plug/mime.types) 文档。

### 设置 HTTP 状态码 (Setting the HTTP Status)

我们可以方便的用类似的方法改变 HTTP 状态码， 被所有控制器默认引入的 `Plug.Conn` 模块提供了 `put_status/2` 函数。

`put_status/2`接收两个参数，第一个是 `conn` ， 第二个是一个整数或者是 "读友好" 原子类型，我们可以在这里找到这些支持的类型 [friendly names](https://github.com/elixir-lang/plug/blob/master/lib/plug/conn/status.ex#L7-L63).

让我们改变一下我们 `PageController` 中的 `index` 函数。


```elixir
def index(conn, _params) do
  conn
  |> put_status(202)
  |> render("index.html")
end
```

我们传入的整型也就是状态码必须是合法的，请查阅 -- [Cowboy](https://github.com/ninenines/cowboy),如果不合法，当 Phoenix 启动时，会抛出错误。当我们重载页面时，通过开发者工具（ development logs 也就是 iex session 或者 浏览器的 network 工具 ）会看到状态码已经改变。

如果 action 发送一个 response --- 不管是渲染还是重定向 -- 仅仅改变了状态码，是不会改变输出的行为的。 比方说，如果我们把状态码变为 404 或者 500, 然后 `render "index.html"`, 我们不会得到错误页面，同样的,单将状态码置为 300 也不会真的重定向（它不知道重定向到哪里，即使状态码确实影响到了行为）。

下面在  `HelloPhoenix.PageController` `index` action, 中的写法是 _不会_  渲染默认的 404 页面的！

```elixir
def index(conn, _params) do
  conn
  |> put_status(:not_found)
  |> render("index.html")
end
```

正确的渲染 404 页面的方法如下：

```elixir
def index(conn, _params) do
  conn
  |> put_status(:not_found)
  |> render(HelloPhoenix.ErrorView, "404.html")
end
```

### 重定向（Redirection）

有些情况下，我们需要在处理请求的过程中重定向到一个新的 url , 比如说一个在一个成功的 `create` action ，会重定向到 `show` action 来显示我们刚才创建的数据模型(model), 另外的我还可以重定向到 `index` 来显示所有的条目，等等。

无论是怎样的使用场景， Phoenix 控制器会提供方便的 `redirect/2` 函数提供重定向功能。Phoenix 会区分重定向到应用内的路径，和重定向到外部的 url。

为了尝试 `redirect/2`, 让我们创建一个新的路由：`web/router.ex`。

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router
  . . .

  scope "/", HelloPhoenix do
    . . .
    get "/", PageController, :index
  end

  # New route for redirects
  scope "/", HelloPhoenix do
    get "/redirect_test", PageController, :redirect_test, as: :redirect_test
  end
  . . .
end
```

然后我们改变 `index`, 仅仅让他重定向到新的路由。


```elixir
def index(conn, _params) do
  redirect conn, to: "/redirect_test"
end
```


最后，我们创建一个新的 action ，它仅仅返回纯文本 `Rediect!`。

```elixir
def redirect_test(conn, _params) do
  text conn, "Redirect!"
end
```

当我们刷新[Welcome Page](http://localhost:4000), 我们会看到页面已经被重定向并显示 `Redirect`,  成功了！

我们可以打开浏览器的 `开发者工具` 看看发生了什么, 点击 network 标签页，然后再次访问我们的根路由。我们可以看到两个请求 -- 一个是状态码为 `302` 的 get 请求，一个是状态码为 `200` 的 get `/redirect_test` 请求。

注意重定向函数返回接收两个参数，一个 `conn` 和一个代表路径的字符串，它同样接收一个全路径的 url 字符串。

```elixir
def index(conn, _params) do
  redirect conn, external: "http://elixir-lang.org/"
end
```

我们可以使用 [Routing Guide](http://www.phoenixframework.org/docs/routing) 中提到的 path helpers。

```elixir
defmodule HelloPhoenix.PageController do
  use HelloPhoenix.Web, :controller

  def index(conn, _params) do
    redirect conn, to: redirect_test_path(conn, :redirect_test)
  end
end
```

注意我们在这里不能用 url helper , 因为 `redirect/2` 使用原子： `:to` , 接收的是 path, 所以下面的写法会失败。

```elixir
def index(conn, _params) do
  redirect conn, to: redirect_test_url(conn, :redirect_test)
end
```

如果我们要用在 `redireact/2` 中使用 url helper, 我们必须使用 `:external`。注意这个 url 并不一定必须是正的外部的 url ，比如下面的例子。


```elixir
def index(conn, _params) do
  redirect conn, external: redirect_test_url(conn, :redirect_test)
end
```
