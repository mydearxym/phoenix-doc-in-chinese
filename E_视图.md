# 视图

Phoenix 视图 (views) 有两个主要的工作，第一个，也是最重要的一个是渲染 `模板(template)`, 这里用到的核心函数
`render/3` 是由 `Phoenix.View` 定义的。另外, 视图 (View) 提供一些函数将原始数据转换成 视图(templates) 易
于识别的格式 (原文: Views also provide functions which take raw data and make it easier for templates
to consume. ), 如果你熟悉装饰器或者 facade pattern (更好的翻译？), 你会发现这很类似。

## 渲染模板

Phoenix 遵循约定优于配置的原则，即 `PageController` 需要一个 `PageView` 来渲染位于 `web/templates/page`目录下的模板。

如果你愿意，你甚至可以改变模板根目录 (the template root)。Phoenix 为我们提供了一个 `view/0` 函数( 用法是将目录
名称赋值给 :key 键 ) 用来改变 root 目录，该函数定义在 `HelloPhoenix.Web` 模块的 `web/web.ex` 文件中。

一个新生成的 Phoenix 应用有三个默认视图模块 (view modules) - `ErrorView`, `LayoutView`, 以及 `PageView`, 它们位于 `web/views` 目录下。

让我们看看 `LayoutView`。

```elixir
defmodule HelloPhoenix.LayoutView do
  use HelloPhoenix.Web, :view
end
```

很简单，只有一行代码, `use HelloPhoenix.Web, :view`。这行代码调用了我们上面提到的 `view/0` 函数, 同时它也允许
改变模板的根目录，`view/0` 使用了 `__using__` 宏( 定义在 `Phoenix.View` 中). 它同时会为我们处理好(下一步可能用
到的)引入的模块或者 view 模块中的别名等。

在这篇文档的最开头，我们提到了可以在视图（views）里放置一些在 templates 中使用的函数，我们来尝试一下。

我们打开文件 `templates/layout/app.html.eex` , 然后改变这行代码。

```html
<title>Hello Phoenix!</title>
```

让它能调用 `title/0` 函数，像这样。

```elixir
<title><%= title %></title>
```

然后在 `LayoutView` 中添加 `title/0` 函数。

```elixir
defmodule HelloPhoenix.LayoutView do
  use HelloPhoenix.Web, :view

  def title do
    "Awesome New Title!"
  end
end
```

当我们刷新那个欢迎页面，我们会看到一个新的标题。

`<%=` 和 `%>` 来自 Elixir [EEx](http://elixir-lang.org/docs/stable/eex/) 工程，他们把可执行的 Elixir 代码
包裹在其中，`=` 符号告诉 EEx 输出出结果，如果不加 `=` 符号, EEx 依然会执行代码，只是不会将结果输出出来。在这个
例子中，我们调用 `LayoutView` 中的 `title/0` 函数，然后将结果输出到模板的 title 标签 ( title tag ) 去中。


Note that we didn't need to fully qualify `title/0` with `HelloPhoenix.LayoutView` because our `LayoutView` actually does the rendering.


由于我们使用了 `use HelloPhoenix.Web, :view`, 我们还得到了额外的好处，因为 `view/0` 函数 imports 了
`HelloWhoenix.Router.Helpers`, 我们就不用再在 templates 显式的引用 path helpers 了，我们改变一下 欢
迎页面的 template 来看一个实际的例子。

我们打开 `templates/page/index.html.eex` 看一看。

```html
<div class="jumbotron">
  <h2>Welcome to Phoenix!</h2>
  <p class="lead">A productive web framework that<br>does not compromise speed and maintainability.</p>
</div>
```

现在我们加一行超链接使其能返回同一页（这个功能没什么实际的意义，只是为了演示 path helpers 在 template 中是怎样工作的。）

```html
<div class="jumbotron">
  <h2>Welcome to Phoenix!</h2>
  <p class="lead">A productive web framework that<br>does not compromise speed and maintainability.</p>
  <p><a href="<%= page_path @conn, :index %>">Link back to this page</a></p>
</div>
```

刷新页面后，当我们查看网页源代码时，我们会看到：

```html
<a href="/">Link back to this page</a>
```

不错， `page_path/2` 按我们希望的被编译成了 `/`, 并且我们没有显式的引入 `HelloPhoenix.View` (原文：and we
didn't need to qualify it with `HelloPhoenix.View`.)

### 关于视图的更多话题

你也许会好奇视图(views) 和模板(templates) 是怎样一起紧密协同工作的。

`Phoenix.View` 通过这行 `use Phoenix.Template` 宏获得模板（template, 也就是 `Phoenix.Template` ）的提供的
各种方便的方法，比如 -- 查找，抽象名字和路径等等。

我们在 Phoenix 默认生成的 `web/views/page_view.ex` 文件中做个小实验，我们增加一个 `message/0` 函数，像这样：

```elixir
defmodule HelloPhoenix.PageView do
  use HelloPhoenix.Web, :view

  def message do
    "Hello from the view!"
  end
end
```

然后我们创建一个新的模板 `web/templates/page/test.html.eex`。

```html
This is the message: <%= message %>
```
这个模板并不对应我们 controller 中的任何 action, 但我们可以在交互式的 `iex` 中运行它，在项目根目录下，运行
`iex -S mix`, 然后明确的渲染我们的模板。

```console
iex(1)> Phoenix.View.render(HelloPhoenix.PageView, "test.html", %{})
  {:safe, [["" | "This is the message: "] | "Hello from the view!"]}
```

如你所见, 我们调用的 `render/3` 函数接受三个参数，独立的视图(HelloPhoenix.PageView), 我们模板的名字，以及一个
供传递可能参数的键值对。

返回值是一个以 `:safe` 原子开头的元组 (tuple), 包含模板中插值字符的返回值(原文：the resultant string of the interpolated template)。

这里的 "Safe" 是指 Phoenix 已经帮我们转义了 (escaped)  模板中返回的内容。Phoenix 定义了自己的
`Phoenix.HTML.Safe` 协议，并将其实现到 atoms, bitstrings, list, integers, floats, 和 tuples 来接收从模板
内容转义到字符串的内容。(原文：Phoenix defines its own `Phoenix.HTML.Safe` protocol with implementations
for atoms, bitstrings, lists, integers, floats, and tuples to handle this escaping for us as our templates
are rendered into strings.)

如果我们给`render/3` 传递第三个参数会发生什么呢？ 我们先改变一下模板 (template)。

```html
I came from assigns: <%= @message %>
This is the message: <%= message %>
```

注意上面那行中的 `@` 符号, 现在当我们改变函数调用，就会看到 `PageView` 模块渲染出了不同的结果。

```console
iex(2)> r HelloPhoenix.PageView
web/views/page_view.ex:1: warning: redefining module HelloPhoenix.PageView
{:reloaded, HelloPhoenix.PageView, [HelloPhoenix.PageView]}

iex(3)> Phoenix.View.render(HelloPhoenix.PageView, "test.html", message: "Assigns has an @.")
{:safe,
  [[[["" | "I came from assigns: "] | "Assigns has an @."] |
  "\nThis is the message: "] | "Hello from the view!"]}
 ```
 我们再测试一下 HTML 的转义, just for fun 。

```console
iex(4)> Phoenix.View.render(HelloPhoenix.PageView, "test.html", message: "<script>badThings();</script>")
{:safe,
  [[[["" | "I came from assigns: "] |
     "&lt;script&gt;badThings();&lt;/script&gt;"] |
    "\nThis is the message: "] | "Hello from the view!"]}
```

如果我们只想得到字符串而不是整个元组，我们可以使用 `render_to_iodata/3`。

 ```console
 iex(5)> Phoenix.View.render_to_iodata(HelloPhoenix.PageView, "test.html", message: "Assigns has an @.")
 [[[["" | "I came from assigns: "] | "Assigns has an @."] |
   "\nThis is the message: "] | "Hello from the view!"]
  ```

### 关于布局 ( A Word About Layouts )

布局 (Layouts) 实际上就是 模板 (templates), 所以它也有视图(view), 就像其他模板一样。 在新生成的应用中，就是
`web/views/layout_view.ex`。你也许会好奇渲染出的内容是怎么被塞进布局 (Layouts) 中的。

我们看看 `web/templates/layout/app.html.eex` 文件，大概在 `<body>` 的中间部分，有这样一行代码。

```html
<%= render @view_module, @view_template, assigns %>
```
这里就是模板渲染成字符串后被装进 Layout 的地方。


### 错误页面 (The ErrorView)

Phoenix 最近为每个生成的应用添加了一个新的视图 (view), 即`ErrorView` (位置在 `web/views/error_view.ex` )。它的
作用主要是处理两种最常见的错误 -- `404 not found` 以及 `500 internal error` -- 让我们看看这个文件的内容。

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

在我们深入探讨之前，先来看看这个 `404 not found` 在浏览器中是怎样的。在开发环境(development enviroment)下,
Phoenix 会默认调试错误，并展示给我们一个详细的 debug 页面，这也是我们想要的，但是，我现在想看的是在生产环境下
的页面的样子，我们需要设置 `config/dev.exs` 文件中的 `debug_errors : false`。


```elixir
use Mix.Config

config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: 4000],
  debug_errors: false,
  code_reloader: true,
  . . .
```

改变配置后，我们需要重启一下服务器，然后访问 [http://localhost:4000/such/a/wrong/path](http://localhost:4000/such/a/wrong/path) 。

我们只是看到了一个没有任何标签的  "Page not found" 裸字符串。

现在让我们用已经学到的一些 views 的知识来装修一下这个页面。

首先，我们要搞清楚这个字符串来自哪里？ 答案很明显，在 `ErrorView` ：


```elixir
def render("404.html", _assigns) do
  "Page not found"
end
```

注意这里的  `render` 函数, 它接收一个模板的名字以及一个 `assigns` 键值对（这个例子中被忽略）。 这个 render
函数实在什么地方被调用的呢？

`render` 函数定义在 `Phoenix.Endpoint.ErrorHandler` 模块中。这个模块的使命就是捕捉错误并用一个视图将它们渲染
出来，在这里，就是 `HelloPhoenix.ErrorView`。

知道了所以然，我们来编写一个更好的错误页面吧。

Phoenix 默认为我们提供了 `ErrorView`, 但是却并没有为我们生成 `web/templates/error` 目录。现在我们自己创建这个
目录，并在其中添加一个模板 `not_found.html.eex`, 内容如下:


```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Welcome to Phoenix!</title>
    <link rel="stylesheet" href="/css/app.css">
  </head>

  <body>
    <div class="container">
      <div class="header">
        <ul class="nav nav-pills pull-right">
          <li><a href="http://www.phoenixframework.org/docs">Get Started</a></li>
        </ul>
        <span class="logo"></span>
      </div>

      <div class="jumbotron">
        <p>Sorry, the page you are looking for does not exist.</p>
      </div>

      <div class="footer">
        <p><a href="http://phoenixframework.org">phoenixframework.org</a></p>
      </div>

    </div> <!-- /container -->
    <script src="/js/app.js"></script>
  </body>
</html>
```

现在我们可以在之前的 `iex` 会话中使用  `render/2` 函数了，改动如下：

```elixir
def render("404.html", _assigns) do
  render("not_found.html", %{})
end
```

我们重新访问 [http://localhost:4000/such/a/wrong/path](http://localhost:4000/such/a/wrong/path), 会得到
一个不错的页面了。

需要指出的是，尽管我们想让错误页面和这个网站的风格保持一致，但这里并没有将 `not_found.html.eex` 模板装入应用的
布局中。 (之后一句不知道怎么翻：The main reason is that it's easy to run into edge case issues while handling errors globally.)

如果我们想在应用的布局已经 `not_found.html.eex` 模板之间减少重复，我们可以复用 header 和 footer 的部分，详情
可以参考 [Template Guide](http://www.phoenixframework.org/docs/templates#section-shared-templates-across-views)。

类似的我们可以在 `ErrorView` 中定义 `def render("500.html", _assigns) do` 。

如果我们想在模板中显示更多信息，还可以使用 `assigns` 传递参数给 `ErrorView` 中的 `render/2` 函数.

## 渲染 JSON

除了渲染模板之外，视图的另一个工作就是渲染 JSON。 Phoenix 使用 [Poison](https://github.com/devinus/poison) 将
Maps 转化为 JSON 格式， 所以我们要做的就是将我们在视图中想要返回的数据装换成 Map, Phoenix 会完成剩下的工作。

尽管在控制器中跳过视图直接返回 JSON 数据是合法的，但是，我国我们仔细想一想控制器的职责是接收请求并抓取返回数据，
对数据的操作和格式化其实并不在这个责任范围之内。这些工作应该交给视图来负责。

让我们看一个 `PageController` 的例子，它返回 JSON 格式而不是之前的 HTML 。

```elixir
defmodule HelloPhoenix.PageController do
  use HelloPhoenix.Web, :controller

  def show(conn, _params) do
    page = %{title: "foo"}

    render conn, "show.json", page: page
  end

  def index(conn, _params) do
    pages = [%{title: "foo"}, %{title: "bar"}]

    render conn, "index.json", pages: pages
  end
end
```

这里，我们使用 `show/2` 和 `index/2` action 返回页面数据。和之前我们将 `"show.html"` 传递给 `render/2` 函数不
同， 这次我们传递 `"show.json"` 。 这样，我们就可以在视图中使用模式匹配灵活的处理 html 和 json 类型了。

```elixir
defmodule HelloPhoenix.PageView do
  use HelloPhoenix.Web, :view

  def render("index.json", %{pages: pages}) do
    %{data: render_many(pages, HelloPhoenix.PageView, "page.json")}
  end

  def render("show.json", %{page: page}) do
    %{data: render_one(page, HelloPhoenix.PageView, "page.json")}
  end

  def render("page.json", %{page: page}) do
    %{title: page.title}
  end
end
```

在视图中我们看到 `render/2` 函数`模式匹配` 了 `"index.json"`, `"show.json"` 和 `"page.json"`。在我们的控制器
`show/2` action 中， `render conn, "show.json", page: page` 将会被视图中的 `render/3` 函数匹配。

也就是 `render conn, "index.json", pages: pages` 会调用视图中的 `render("index.json", %{pages: pages})`。

`render_many/3` 函数有参数，`pages` 数据，一个`视图` 以及一个可被`render/3`函数模式匹配的字符串（文件名）。 它
会遍历 `pages` 里的每一条键值数据，然后传递给匹配的`render/3` 函数。

`render_one/3` 与之类似，最终使用 `render/3` 匹配 `page.json` 来决定最终每个 `page` 的样子。

`render/3` 匹配的 `"index.json"` 返回的 JSON 数据如下：

```javascript
  {
    "data": [
      {
       "title": "foo"
      },
      {
       "title": "bar"
      },
   ]
  }
```

`"show.json"` 如下：

```javascript
  {
    "data": {
      "title": "foo"
    }
  }
```

这样有个很大的好处是 `视图` 是可以被组合的。比如我们的 `Page` 有很多(`has_many`) `Author` ,并且根据请求参数，
我们需要将 `author` 和 `page` 信息一起发送回去。我们可以像下面这样：

```elixir
defmodule HelloPhoenix.PageView do
  use HelloPhoenix.Web, :view

  def render("page_with_authors.json", %{page: page}) do
    %{title: page.title,
      authors: render_many(page.authors, HelloPhoenix.AuthorView, "author.json")}
  end

  def render("page.json", %{page: page}) do
    %{title: page.title}
  end
end
```
