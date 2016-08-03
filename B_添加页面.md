
### 添加页面

我们的目标是给我们的 `Phoenix` 应用增加两个新的页面，一个是纯静态页面，另一个则从 `url` 里面截取一部分作为输入，
然后传递给模板显示。从这两个简单的例子中我们将会熟悉一个 `Phoenix` 应用的基本构成: 路由，控制器，视图以及模板。

当我们用命令产生一个 `Phoenix` 应用后，默认的目录结构如下：

```console
├── _build
├── config
├── deps
├── lib
├── priv
├── test
├── web
```
我们教程中涉及的大部分内容都在 `web` 目录下，她的结构展开如下图：

```console
├── channels
├── controllers
│   └── page_controller.ex
├── models
├── router.ex
├── templates
│   ├── layout
│   │   └── app.html.eex
│   └── page
│       └── index.html.eex
└── views
|   ├── error_view.ex
|   ├── layout_view.ex
|   └── page_view.ex
└── web.ex
```

在 `controllers`, `templates` 和 `views` 目录里的所有文件都是用于创建我们之前看到的那个
`Welcome to Phenix` 欢迎页面的,我们之后将重用这里的一些代码。根据惯例，在开发模式下，每一
次新的请求到达，`web` 目录都会自动重新编译。

我们应用所需的所有静态资源都在 `priv/static` 目录下（按照css,images或者js分类）, 编译过程
会把 `priv/static` 里对应的 js, css 文件分别编译到 `web/static` 目录的 `app.js` / `app.css`。
, 我们现在不会深入，只是留个印象。

```console
priv
└── static
    └── images
        └── phoenix.png

```
```
web
└── static
    ├── css
    |   └── app.scss
    ├── js
    │   └── app.js
    └── vendor
        └── phoenix.js
```

我们同样需要了解一下 `lib` 目录, 我们应用的 endpoint 位于 `lib/hello_phoenix/endpoint.ex`, 而我们的应用
启动文件(负责启动我们的应用以及 supervision 监测树)位于 `lib/hello_phoenix.ex`。

```console
lib
├── hello_phoenix
|   ├── endpoint.ex
│   └── repo.ex
└── hello_phoenix.ex
```

与 `web` 目录不同的是，Phoenix 不会为每次请求都重新编译(recompile)`lib` 目录。`web` 和 `lib` 目录提供了
两种不同的状态管理策略， `web` 目录处理的状态范围只限定在一次 web 请求中，而 `lib` 目录所负责管理的状态则包
含共享的模块之间和那些超出一次 web 请求之外的那些状态。

唠叨的差不多了，现在让我们编写一个新的 `Phoenix` 页面吧！

####  一个新路由

路由将给定并唯一的 `HTTP 动词/路径(verb/path)` 映射到处理他们的 `controller/action  `, `Phoenix`的路由
配置信息在 `web/router.ex`文件，让我们打开来看看。

欢迎页面对应的路由是这条语句：

```Elixir
get "/", PageController, :index
```

让我们看看这代表什么意思呢，浏览器访问 [http://localhost:4000/](http://localhost:4000/) 会给我们网站的根
目录发出一个`GET`请求，这个请求会被 `HelloPhoenix.pageController` 中的 `index` 函数处理，后者定义在
`web/controllers/page_controller.ex`文件中.

我们要建立的页面功能是：当浏览器访问 [http://localhost:4000/hello](http://localhost:4000/hello) 时，简单
的在页面上显示 'Hello World from Phoenix'.

首先我们要为这个页面增加一个路由，现在`web/route.ex`看起来如下：

```Elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  pipeline :api do
    plug :accepts, ["json"]
  end

  scope "/", HelloPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
  end

  # Other scopes may use custom stacks.
  # scope "/api", HelloPhoenix do
  #   pipe_through :api
  # end
end

```

现在，我们先不管 `pipelines` 和 `scope` 部分（如果你好奇的话，可以阅读 [路由指北](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/C_%E8%B7%AF%E7%94%B1.html))
）.

让我们为 `/hello` 的 `GET` 请求创建一个路由吧, 对应的文件 --- `HelloPhoenix`文件稍后创建。

```Elixir
get "/hello", HelloController, :index
```

添加后，`router.ex` 文件的 `scope '/'` 部分看起来应该是这个样子：

```Elixir
scope "/", HelloPhoenix do
  pipe_through :browser # Use the default browser stack

  get "/", PageController, :index
  get "/hello", HelloController, :index
end
```
#### 一个新控制器

`控制器` 里面其实就是 Elixir 的模块、`action` 和定义在其中的`Elixir` 函数，`action` 的作用是
收集和处理渲染页面所需的数据和某些操作 (gather any data and perform any tasks needed for rendering )。
我们现在要建立一个模块 `HelloPhoenix.HelloController`, 里面包含 `index/2` 这个action ( /2，意思是需要两个参数的意思)。

具体的，我们建立文件`web/controllers//hello_controller.ex`, 然后加入以下内容：

```Elixir
defmodule HelloPhoenix.HelloController do
  use HelloPhoenix.Web, :controller

  def index(conn, _params) do
    render conn, "index.html"
  end
end
```

我们会在 [控制器指北](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/D_%E6%8E%A7%E5%88%B6%E5%99%A8.html)
章节中详细讨论 `HelloPhoenix.Web, :controller` 语句，现在我们先来看看 `index/2 `函数。

所有的 `controller actions` 都接受两个参数，第一个是 `conn`--一个包含了大量`请求（request）`信息的结构体。
第二个是`params`--请求参数，我们的例子不需要`params`, 所以加一个前缀 `_`表示忽略该参数，否则编译过程会产生警告。

这个 `action` 的核心语句是 `render conn, "index.html"`, Phoenix 会根据这条语句寻找一个叫做 `index.html.eex`
的模板并且渲染出来，查找规则是遵循 `controller` 的命名,即 `web/templates/hello` 目录。

>*注意* 这里用原子(atom)的简写法也是可以的，比如`render conn, :index`,这是渲染机制就会根据请求头的规则自动寻找合适的模板,如 `index.html` 或者 `index.json`等等。

负责具体渲染的模块是`视图(views)`,我们现在就来建立一个。

#### 一个新的视图

`视图(views)` 负责几个重要的工作，她渲染模板，并且作为 `controller` 里 `裸数据(raw data)` 的展示层(presentation layer)，
以及一些有用的函数将这些数据处理后供模板使用。

举个栗子，比如我们有一个包含了 `first_name` 字段和 `last_name` 的用户数据，然后在模板里，我们想显示这个
用户的全名。我们可以在模板里写函数把它们拼接起来，但更好的办法是在 `视图(view)` 里写一个函数
去做这件事，然后在模板里调用这个函数，这种解决方案会让模板变得干净且更加灵活。

为了能让我们的 `HelloController` 渲染模板，我们需要一个 `HelloView`。这里的命名同样具有特殊意义-- 'Hello' 要和
控制器的'Hello'相对应，让我们先创建一个 `web/views/hello_view.ex` 文件（稍后解释细节），内容如下：

```Elixir
defmodule HelloPhoenix.HelloView do
  use HelloPhoenix.Web, :view
end
```

####  一个新模板

Phoenix 默认使用的模板引擎是`eex`,意思是
[嵌入式 Elixir (Embedded Elixir)](http://elixir-lang.org/docs/stable/eex/),
所以我们的模板文件都会带有`.eex`后缀。

模板被视图所限定，视图又被控制器所限定，在实际开发中，我们通常根据控制器的名字在 `web/templates` 目录下建立一
个文件夹。在我们这个hello应用中，这意味着我们需要在`web/templates`目录下面建立一个hello文件夹，再在里面建立一
个`index.html.eex`文件，内容如下。

```html
<div class="jumbotron">
  <h2>Hello World, from Phoenix!</h2>
</div>

```
现在我们有了`路由`，`控制器`,`视图` 和 `模板`，我们应该可以通过浏览器访问
[http://localhost:4000/hello](http://localhost:4000/hello)
看到新的欢迎页面了！（如果你之前停止了服务器可以通过 `mix phoenix.server` 重新启动）。

[![Gyazo](https://i.gyazo.com/f167bf8b89fc8d7bd7aa39f22642a15f.png)](https://gyazo.com/f167bf8b89fc8d7bd7aa39f22642a15f)

这里有些知识点需要注意：首先，改动代码以后，我们并不需要重启服务器，`Phoenix` 有代码热更新 ( hot code re-loading) 机制。另外，
尽管我们的 `index.html.eex` 文件只包含了一个`div`标签，我们依然得到了一个"完整的"页面，我们的模板被渲染进了应
用布局(application layout) -- `web/templates/layout/app.html.eex`。 如果你打开她，就会发现有这样一行内容`<%=
@inner %>`, 意思在被送往浏览器之前，你自己的模板会被"插入"到这里。


#### 另一个新页面

让我们为我们的应用增加一点复杂性。我们建立一个新的页面，她的功能是识别我们url的一部分，将她作为'信息'通过
给控制器传递给模板，然后显示出来。

就像我们之前做的那样，先添加一个路由。

#### 新的路由
在这个练习中，我们复用之前创建的 `HelloController` 然后添加一个新的 `show` action, 增加的路由如下：

```Elixir
scope "/", HelloPhoenix do
  pipe_through :browser # Use the default browser stack.

  get "/", PageController, :index
  get "/hello", HelloController, :index
  get "/hello/:messenger", HelloController, :show
end
```

>*注意* 这个原子写法：`:messenger`, Phoenix 会把 messenger 作为键，并把用户输入在这个地址后的任何输入作为值，组成一个`字典结构(dict)` 传递给 controller。

比如，如果我们在浏览器输入 [http://localhost:4000/hello/Frank](http://localhost:4000/hello/Frank), ":messenger" 的值就会是 "Frank" 。

#### 新的 action

新的请求会被 `HelloPhoenix.HelloController` 的 `show` action 处理，我们复用之前的
`web/controllers/hello_controller.ex`文件，
在其中添加 show action 如下（注意其中字典的写法）：

```Elixir
def show(conn, %{"messenger" => messenger}) do
  render conn, "show.html", messenger: messenger
end
```
>*注意* 如果你想要得到 params 的所有参数并绑定 messenger 变量，我们可以这样定义（类似js es6 的解构写法）：

```Elixir
def show(conn, %{"messenger" => messenger} = params) do
  ...
end
```

> 注意所有的 keys 都是字符串，还有这里的 `＝` 不是赋值符号，而是[模式匹配(pattern match)](http://elixir-lang.org/getting-started/pattern-matching.html) 。

#### 新的模板

终于到最后一步了，一个新的模板。根据命名规则，`Phoenix`会去 `web/templates/hello` 文件夹下寻找
`show.html.eex`文件。她和之前的 `index.eex` 模板长的很像，除了一个 Elixir 表达式：`<%= %>` ,
注意标签里有个`<%=`符号。这表示这个便签里的 Elixir 代码会被执行并且结果会覆盖这个标签，如果我们不
加 `=` , 里面的 Elixir 代码依然会执行，但是结果却不会出现在页面上。

`show.html.eex`文件的内容如下：

```html
<div class="jumbotron">
  <h2>Hello World, from <%= @messenger %>!</h2>
</div>
```

这里的@messenger并不是什么模块的属性，而是一个元编程的语法糖，表示 `dict.get(assigns, :messenger)`。只是这种写法对于模板来说更加友好。

一切就绪，现在让我们打开 [http://localhost:4000/hello/Frank](http://localhost:4000/hello/Frank) 来检验成果吧！

[![Gyazo](https://i.gyazo.com/d8d266a9bf995f7e97636b7734c0d773.png)](https://gyazo.com/d8d266a9bf995f7e97636b7734c0d773)

