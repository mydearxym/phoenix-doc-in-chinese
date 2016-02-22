
Phoenix 视图 (views) 有两个主要的工作，第一个，也是最重要的一个是渲染 `模板(template)`, 这里用到的核心函数 `render/3` 是由 `Phoenix.View` 定义的。另外, 视图 (View) 提供一些函数将原始数据转换成 视图(templates) 易于识别的格式 (原文: Views also provide functions which take raw data and make it easier for templates to consume. ), 如果你熟悉装饰器或者 facade pattern (更好的翻译？), 你会发现这很类似。

Phoenix 遵循习惯优于配置的原则，即 `PageController` 需要一个 `PageView` 来渲染位于 `web/templates/page`目录下的模板。

如果你愿意，你甚至可以改变模板根目录 (the template root)。Phoenix 为我们提供了一个 `view/0` 函数( 用法是将目录名称赋值给 :key 键 ) 用来改变 root 目录，该函数定义在 `HelloPhoenix.Web` 模块的 `web/web.ex` 文件中。


一个新生成的 Phoenix 应用有三个默认视图模块 (view modules) - `ErrorView`, `LayoutView`, 以及 `PageView`, 它们位于 `web/views` 目录下。

让我们看看 `LayoutView`。

```elixir
defmodule HelloPhoenix.LayoutView do
  use HelloPhoenix.Web, :view
end
```

很简单，只有一行代码, `use HelloPhoenix.Web, :view`。这行代码调用了我们上面提到的 `view/0` 函数, 同时它也允许改变模板的根目录，`view/0` 使用了 `__using__` 宏( 定义在 `Phoenix.View` 中). 它同时会为我们处理好(下一步可能用到的)引入的模块或者 view 模块中的别名等。

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

`<%=` 和 `%>` 来自 Elixir [EEx](http://elixir-lang.org/docs/stable/eex/) 工程，他们把可执行的 Elixir 代码包裹在其中，`=` 符号告诉 EEx 输出出结果，如果不加 `=` 符号, EEx 依然会执行代码，只是不会将结果输出出来。在这个例子中，我们调用 `LayoutView` 中的 `title/0` 函数，然后将结果输出到模板的 title 标签 ( title tag ) 去中。


Note that we didn't need to fully qualify `title/0` with `HelloPhoenix.LayoutView` because our `LayoutView` actually does the rendering.


由于我们使用了 `use HelloPhoenix.Web, :view`, 我们还得到了额外的好处，因为 `view/0` 函数 imports 了 `HelloWhoenix.Router.Helpers`, 我们就不用再在 templates 显式的引用 path helpers 了，我们改变一下 欢迎页面的 template 来看一个实际的例子。

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

不错， `page_path/2` 按我们希望的被编译成了 `/`, 并且我们没有显式的引入 `HelloPhoenix.View` (原文：and we didn't need to qualify it with `HelloPhoenix.View`.)


### More About Views

You might be wondering how views are able to work so closely with templates.

The `Phoenix.View` module gains access to template behavior via the `use Phoenix.Template` line in its `__using__/1` macro. `Phoenix.Template` provides many convenience methods for working with templates - finding them, extracting their names and paths, and much more.

Let's experiment a little with one of the generated views Phoenix provides us, `web/views/page_view.ex`. We'll add a `message/0` function to it, like this.

```elixir
defmodule HelloPhoenix.PageView do
  use HelloPhoenix.Web, :view

  def message do
    "Hello from the view!"
  end
end
```

Now let's create a new template to play around with, `web/templates/page/test.html.eex`.

```html
This is the message: <%= message %>
```

This doesn't correspond to any action in our controller, but we'll exercise it in an `iex` session. At the root of our project, we can run `iex -S mix`, and then explicitly render our template.

```console
iex(1)> Phoenix.View.render(HelloPhoenix.PageView, "test.html", %{})
  {:safe, [["" | "This is the message: "] | "Hello from the view!"]}
```
As we can see, we're calling `render/3` with the individual view responsible for our test template, the name of our test template, and an empty map representing any data we might have wanted to pass in.

The return value is a tuple beginning with the atom `:safe` and the resultant string of the interpolated template.

"Safe" here means that Phoenix has escaped the contents of our rendered template. Phoenix defines its own `Phoenix.HTML.Safe` protocol with implementations for atoms, bitstrings, lists, integers, floats, and tuples to handle this escaping for us as our templates are rendered into strings.

What happens if we assign some key value pairs to the third argument of `render/3`? In order to find out, we need to change the template just a bit.

```html
I came from assigns: <%= @message %>
This is the message: <%= message %>
```

Note the `@` in the top line. Now if we change our function call, we see a different rendering after recompiling `PageView` module.

```console
iex(2)> r HelloPhoenix.PageView
web/views/page_view.ex:1: warning: redefining module HelloPhoenix.PageView
{:reloaded, HelloPhoenix.PageView, [HelloPhoenix.PageView]}

iex(3)> Phoenix.View.render(HelloPhoenix.PageView, "test.html", message: "Assigns has an @.")
{:safe,
  [[[["" | "I came from assigns: "] | "Assigns has an @."] |
  "\nThis is the message: "] | "Hello from the view!"]}
 ```
Let's test out the HTML escaping, just for fun.

```console
iex(4)> Phoenix.View.render(HelloPhoenix.PageView, "test.html", message: "<script>badThings();</script>")
{:safe,
  [[[["" | "I came from assigns: "] |
     "&lt;script&gt;badThings();&lt;/script&gt;"] |
    "\nThis is the message: "] | "Hello from the view!"]}
```

If we need only the rendered string, without the whole tuple, we can use the `render_to_iodata/3`.

 ```console
 iex(5)> Phoenix.View.render_to_iodata(HelloPhoenix.PageView, "test.html", message: "Assigns has an @.")
 [[[["" | "I came from assigns: "] | "Assigns has an @."] |
   "\nThis is the message: "] | "Hello from the view!"]
  ```

### A Word About Layouts

Layouts are just templates. They have a view, just like other templates. In a newly generated app, this is `web/views/layout_view.ex`. You may be wondering how the string resulting from a rendered view ends up inside a layout. That's a great question!

If we look at `web/templates/layout/app.html.eex`, just about in the middle of the `<body>`, we will see this.

```html
<%= render @view_module, @view_template, assigns %>
```

This is where the view module and its template from the controller are rendered to a string and placed in the layout.

### The ErrorView

Phoenix recently added a new view to every generated application, the `ErrorView` which lives in `web/views/error_view.ex`. The purpose of the `ErrorView` is to handle two of the most common errors - `404 not found` and `500 internal error` - in a general way, from one centralized location. Let's see what it looks like.

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

Before we dive into this, let's see what the rendered `404 not found` message looks like in a browser. In the development environment, Phoenix will debug errors by default, showing us a very informative debugging page. What we want here, however, is to see what page the application would serve in production. In order to do that we need to set `debug_errors: false` in `config/dev.exs`.

```elixir
use Mix.Config

config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: 4000],
  debug_errors: false,
  code_reloader: true,
  . . .
```

After modifying our config file, we need to restart our server in order for this change to take effect. After restarting the server, let's go to [http://localhost:4000/such/a/wrong/path](http://localhost:4000/such/a/wrong/path) for a running local application and see what we get.

Ok, that's not very exciting. We get the bare string "Page not found", displayed without any markup or styling.

Let's see if we can use what we already know about views to make this a more interesting error page.

The first question is, where does that error string come from? The answer is right in the `ErrorView`.

```elixir
def render("404.html", _assigns) do
  "Page not found"
end
```

Great, so we have a `render/2` function that takes a template and an `assigns` map, which we ignore. Where is this `render/2` function being called from?

The answer is the `render/5` function defined in the `Phoenix.Endpoint.ErrorHandler` module. The whole purpose of this module is to catch errors and render them with a view, in our case, the `HelloPhoenix.ErrorView`.

Now that we understand how we got here, let's make a better error page.

Phoenix generates an `ErrorView` for us, but it doesn't give us a `web/templates/error` directory. Let's create one now. Inside our new directory, let's add a template, `not_found.html.eex` and give it some markup - a mixture of our application layout and a new `div` with our message to the user.


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

Now we can use the `render/2` function we saw above when we were experimenting with rendering in the `iex` session.

Our `render/2` function should look like this when we've modified it.

```elixir
def render("404.html", _assigns) do
  render("not_found.html", %{})
end
```

When we go back to [http://localhost:4000/such/a/wrong/path](http://localhost:4000/such/a/wrong/path), we should see a much nicer error page.

It is worth noting that we did not render our `not_found.html.eex` template through our application layout, even though we want our error page to have the look and feel of the rest of our site. The main reason is that it's easy to run into edge case issues while handling errors globally.

If we want to minimize duplication between our application layout and our `not_found.html.eex` template, we can implement shared templates for our header and footer. Please see the [Template Guide](http://www.phoenixframework.org/docs/templates#section-shared-templates-across-views) for more information.

Of course, we can do these same steps with the `def render("500.html", _assigns) do` clause in our `ErrorView` as well.

We can also use the `assigns` map passed into any `render/2` clause in the `ErrorView`, instead of discarding it, in order to display more information in our templates.
