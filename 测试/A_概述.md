# 概述

测试已经成为现代软件开发的重要一环，能不能容易的写出有意义的测试也成为了现代 Web 框架的一个重要指标。 Phoenix
也不例外，内建了很多工具保证了应用的主要部分能被很容易的测试。它提供了一个与独立于其他模块的生产级别的
`测试模块` 来帮你更好的测试。

Elixir 内建一个测试框架 [ExUnit](http://elixir-lang.org/docs/stable/ex_unit/)。ExUnit 力争做到清晰明了，尽量减
少 "黑魔法" 的使用。 Phoenix 也是用 ExUnit 作为测试工具。

ExUnit 将一个待测试的模块称为一个 `测试用例` (test case), Phoenix 中也一样。

我们来看看实际的用法。

> 注意： 在我们开始之前，先必须确保 PostgreSQL 已经安装并运行在我们的系统上。我们还需要正确的配置 repo。
> 关于这些内容，请查看 [Mix 任务中的 ecto.create 章节](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/%E8%BF%9B%E9%98%B6%E5%86%85%E5%AE%B9/D_mix%E4%BB%BB%E5%8A%A1.html)
> 其中具体的实现细节请查看 [Ecto 模型指南](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/H_ecto%E6%A8%A1%E5%9E%8B.html)

在新产生的应用中，我们在根目录运行 `mix test` (关于生成新项目，请查看
[A_起步](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/A_%E8%B5%B7%E6%AD%A5.html))


```console
$ mix test
....

Finished in 0.2 seconds (0.2s on load, 0.00s on tests)
4 tests, 0 failures

Randomized with seed 652656
```

我们已经准备好测试了！

实际上，我们已经有了一个用于测试的目录，包含一个 test helper 和一些支持文件。

> 注意： 我们并不需要为 test 数据库创建或者运行迁移任务， test helper 已经帮我们都弄好了。

```console
test
├── channels
├── controllers
│   └── page_controller_test.exs
├── models
├── support
│   ├── channel_case.ex
│   ├── conn_case.ex
│   └── model_case.ex
├── test_helper.exs
└── views
    ├── error_view_test.exs
    ├── layout_view_test.exs
    └── page_view_test.exs
```

默认生成的测试用例 (test cases) 包括 `test/controllers/page_controller_test.exs`,
`test/views/error_view_test.exs`以及 `test/views/page_view_test.exs`.

在我们详细展开之前，我们先来简单的看看这几个文件热热身。

首先来看 `test/controllers/page_controller_test.exs`。

```elixir
defmodule HelloPhoenix.PageControllerTest do
  use HelloPhoenix.ConnCase

  test "GET /" do
    conn = get conn(), "/"
    assert html_response(conn, 200) =~ "Welcome to Phoenix!"
  end
end
```

这里有些东西需要提一下。

`get/2` 函数返给我们一个"已经" 请求过 "/" 的连接结构体。帮我们干了很多脏活累活。

这里的断言（assertion） 测试了三个点 - 一世我们的返回是 html 格式的（通过检查返回的 content-type 是不是
"text/html"）, 检查返回的状态码是 200, 还有检查返回的内容里包含 "Welcome to Phoenix!"。

我们再来看看错误视图的测试用例, `test/views/error_view_test.exs` 。

```elixir
defmodule HelloPhoenix.ErrorViewTest do
  use HelloPhoenix.ConnCase, async: true

  # Bring render/3 and render_to_string/3 for testing custom views
  import Phoenix.View

  test "renders 404.html" do
    assert render_to_string(HelloPhoenix.ErrorView, "404.html", []) ==
           "Page not found"
  end

  test "render 500.html" do
    assert render_to_string(HelloPhoenix.ErrorView, "500.html", []) ==
           "Server internal error"
  end

  test "render any other" do
    assert render_to_string(HelloPhoenix.ErrorView, "505.html", []) ==
           "Server internal error"
  end
end
```

`HelloPhoenix.ErrorViewTest` 设置 `async: true` 意思是测试是并行运行的，这样可以大大加快测试的速度。可以这么做
的原因是我们这里的测试用例并没有共享什么状态，如果共享了状态，比如数据库中的数据，那么我们的测试就可能会冲突。

我们同时引入了 `Phoenix.View` 以便于使用 `render_to_string/3` 函数。 有了它，所有的断言可以与字符串进行比较。

page 的视图, `test/views/page_view_test.exs` 默认并没有包含测试，但我们随时可以在 `HelloPhoenix.PageView` 模块
中添加对其的测试。

```elixir
defmodule HelloPhoenix.PageViewTest do
  use HelloPhoenix.ConnCase, async: true
end
```

接下来我们看看 Phoenix 内建的一些 helper 文件。

默认的测试 helper 文件，`test/test_helper.exs`, 为我们生成了测试数据库并完成了迁移任务。并且会在每次测试运行完
成后将数据库的内容回滚回去。

如果我们需要的话，test helper 也可以包含只针对测试环境的配置

```elixir
ExUnit.start

Mix.Task.run "ecto.create", ["--quiet"]
Mix.Task.run "ecto.migrate", ["--quiet"]
Ecto.Adapters.SQL.begin_test_transaction(HelloPhoenix.Repo)
```

`test/support` 目录下的文件作用是将我们的模块转为待测试的状态 (get our modules into a testable state)。它提供
了一些方便的函数来执行我们的任务，比如设置好连接结构体，在 Ecto changeset 中发现错误等。我们在接下来的章节中会
仔细探讨它们。

### 运行测试

现在我们知道了我们的测试是干什么的，我们来看看怎么运行它们。

在上面的部分可以看到，我们可以使用 `mix test` 来运行我们的测试。

```console
$ mix test
....

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
4 tests, 0 failures

Randomized with seed 540755
```

我们可以给测试运行指定目录，这样它只运行该目录下的测试用例。

```console
$ mix test test/controllers/
.

Finished in 0.2 seconds (0.1s on load, 0.04s on tests)
1 tests, 0 failures

Randomized with seed 652376
```

也可以指定一个单独的文件。

```console
$ mix test test/views/error_view_test.exs
...

Finished in 0.2 seconds (0.2s on load, 0.00s on tests)
3 tests, 0 failures

Randomized with seed 220535
```

我们甚至可以通过 `：数字` 的方式指定测试单个文件中的某个函数。

比如我们指向测试 `HelloPhoenix.ErrorView` 渲染 `500.html`的函数(在第 12 行), 我们可以这样写

```console
$ mix test test/views/error_view_test.exs:12
Including tags: [line: "12"]
Excluding tags: [:test]

.

Finished in 0.1 seconds (0.1s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 288117
```

### 使用标签运行测试

ExUnit 允许我们给测试用例在模块级别或函数级别打`标签`(ExUnit allows us to tag our tests at the case level or
on the individual test level.), 我们可以在运行测试是指定或排除该标签代表的测试用例。

我们来看一个实际的例子。
首先，我们加一个 `@moduletag` 给 `test/views/error_view_test.exs` 。

```elixir
defmodule HelloPhoenix.ErrorViewTest do
  use HelloPhoenix.ConnCase, async: true

  @moduletag :error_view_case
  ...
end
```

如果我们的模块标签是一个 `原子`, ExUnit 会假设标签的值是 `true`。我们也可以显示的指定一个值。

```elixir
defmodule HelloPhoenix.ErrorViewTest do
  use HelloPhoenix.ConnCase, async: true

  @moduletag error_view_case: "some_interesting_value"
  ...
end
```

我们还是使用默认的原子形式：`@moduletag :error_view_case`。

我们可以在运行是使用 `--only error_view_case` 选项来只运行包含该标签的测试用例。

```console
$ mix test --only error_view_case
Including tags: [:error_view_case]
Excluding tags: [:test]

...

Finished in 0.1 seconds (0.1s on load, 0.00s on tests)
3 tests, 0 failures

Randomized with seed 125659
```

> 注意： 如果我们留意一下提示信息， 会发现之前指定数字的那种方式其实也是被当作标签对待了。

```console
$ mix test test/views/error_view_test.exs:12
Including tags: [line: "12"]
Excluding tags: [:test]

.

Finished in 0.2 seconds (0.2s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 364723
```

给 `error_view_case` 一个 `true` 值，结果还是一样。

```console
$ mix test --only error_view_case:true
Including tags: [error_view_case: "true"]
Excluding tags: [:test]

...

Finished in 0.1 seconds (0.1s on load, 0.00s on tests)
3 tests, 0 failures

Randomized with seed 833356
```

但给 `error_view_case` 指定一个 `false` 的值则会失败，因为没有标签能匹配 `error_view_case: false`。

```console
$ mix test --only error_view_case:false
Including tags: [error_view_case: "false"]
Excluding tags: [:test]



Finished in 0.1 seconds (0.1s on load, 0.00s on tests)
0 tests, 0 failures

Randomized with seed 622422
```

类似的，我们可以使用 `--exclude` 选项排除一些测试用例。

```console
$ mix test --exclude error_view_case
Excluding tags: [:error_view_case]

.

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
1 tests, 0 failures

Randomized with seed 682868
```

给标签指定值的用法对 `--exclude` 和 `--only` 同样其效果。

我们还可以更细粒度的给测试用例中的测试打标签，让我们来看一个例子。

```elixir
defmodule HelloPhoenix.ErrorViewTest do
  use HelloPhoenix.ConnCase, async: true

  @moduletag :error_view_case

  # Bring render/3 and render_to_string/3 for testing custom views
  import Phoenix.View

  @tag individual_test: "yup"
  test "renders 404.html" do
    assert render_to_string(HelloPhoenix.ErrorView, "404.html", []) ==
           "Page not found"
  end

  @tag individual_test: "nope"
  test "render 500.html" do
    assert render_to_string(HelloPhoenix.ErrorView, "500.html", []) ==
           "Server internal error"
  end
  ...
end
```

如果我们只想运行 `individual_test` 标签的测试，不考虑它的标签值，我们可以这样：

```console
$ mix test --only individual_test
Including tags: [:individual_test]
Excluding tags: [:test]

..

Finished in 0.1 seconds (0.1s on load, 0.00s on tests)
2 tests, 0 failures

Randomized with seed 813729
```

我们也可以指定标签值：

```console
$ mix test --only individual_test:yup
Including tags: [individual_test: "yup"]
Excluding tags: [:test]

.

Finished in 0.1 seconds (0.1s on load, 0.00s on tests)
1 tests, 0 failures

Randomized with seed 770938
```

类似的，排除某些测试：

```console
$ mix test --exclude individual_test:nope
Excluding tags: [individual_test: "nope"]

...

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
3 tests, 0 failures

Randomized with seed 539324
```

再看一个有趣的例子，我们排除所有错误视图中的测试用例，只运行带 `individual_test` 标签的。

```console
$ mix test --exclude error_view_case --include individual_test
Including tags: [:individual_test]
Excluding tags: [:error_view_case]

...

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
3 tests, 0 failures

Randomized with seed 41241
```

这次运行了带 `individual_test` 标签的两个测试，同时还有 `test/controllers/page_controller_test.exs`。

我们可以更具体的排除所有的错误视图中的测试用例，只保留标签是 `individual_test` 且值为 "yup" 的用例。

```console
$ mix test --exclude error_view_case --include individual_test:yup
Including tags: [individual_test: "yup"]
Excluding tags: [:error_view_case]

..

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
2 tests, 0 failures

Randomized with seed 61472
```

最后，我们可以在 `test/test_helper.exs`文件中将 `exclude` `error_view_case` 写入配置中。

```elixir
ExUnit.start

Mix.Task.run "ecto.create", ["--quiet"]
Mix.Task.run "ecto.migrate", ["--quiet"]
Ecto.Adapters.SQL.begin_test_transaction(HelloPhoenix.Repo)

ExUnit.configure(exclude: [error_view_case: true])
```

现在当我们运行 `mix test`, 只会运行 `page_controller_test.exs` 中的测试了。

```console
$ mix test
Excluding tags: [error_view_case: true]

.

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
1 tests, 0 failures

Randomized with seed 186055
```

我们也可以使用 `--include` 来复写这个行为。

```console
$ mix test --include error_view_case
Including tags: [:error_view_case]
Excluding tags: [error_view_case: true]

....

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
4 tests, 0 failures

Randomized with seed 748424
```

### 随机化

将测试按照随机顺寻运行是一个检测我们的测试是否独立的好办法。如果我们在测试时得到一些零星的错误，这可能是因为之
前的测试用例改变了应用中的某些状态，且没有复原，所以影响了下一个测试用例。这种错误一般是在特定的运行顺序下出现
的。

ExUnit 默认是以随机顺序运行测试的，用一个整数当作随机生成器的种子 (using an integer to seed the randomization)。
如果我们发现一个特定的随机种子会导致测试失败，我们可以使用相同的种子去使用同样的顺序来重现这个错误, 从而帮助我
们发现潜在的问题。

```console
$ mix test --seed 401472
....

Finished in 0.2 seconds (0.1s on load, 0.03s on tests)
4 tests, 0 failures

Randomized with seed 401472
```

### 生成测试

让我们看看当我们生成一个 HTML 资源是会发生什么。

使用之前的那个 User 例子, 在项目的根目录用以下参数运行 `mix phoenix.gen.html`

```console
$ mix phoenix.gen.html User users name:string email:string bio:string number_of_pets:integer

...

Generated hello_phoenix app
* creating priv/repo/migrations/20150519043351_create_user.exs
* creating web/models/user.ex
* creating test/models/user_test.exs
* creating web/controllers/user_controller.ex
* creating web/templates/user/edit.html.eex
* creating web/templates/user/form.html.eex
* creating web/templates/user/index.html.eex
* creating web/templates/user/new.html.eex
* creating web/templates/user/show.html.eex
* creating web/views/user_view.ex
* creating test/controllers/user_controller_test.exs

Add the resource to your browser scope in web/router.ex:

    resources "/users", UserController

Remember to update your repository by running migrations:

    $ mix ecto.migrate
```

现在根据它的提示将资源信息加到路由文件 `web/router.ex` 中。

```elixir
defmodule HelloPhoenix.Router do
  use HelloPhoenix.Web, :router

  ...

  scope "/", HelloPhoenix do
    pipe_through :browser # Use the default browser stack

    get "/", PageController, :index
    resources "/users", UserController
  end

  # Other scopes may use custom stacks.
  # scope "/api", HelloPhoenix do
  #   pipe_through :api
  # end
end
```

现在当我们再次运行 `mix test` , 我们会发现我们已经有十六个测试用例啦！

```console
$ mix test
Compiled lib/hello_phoenix.ex
Compiled web/models/user.ex
Compiled web/views/page_view.ex
Compiled web/views/layout_view.ex
Compiled web/views/error_view.ex
Compiled web/router.ex
Compiled web/controllers/page_controller.ex
Compiled web/controllers/user_controller.ex
Compiled lib/hello_phoenix/endpoint.ex
Compiled web/views/user_view.ex
Generated hello_phoenix app
................

Finished in 0.5 seconds (0.4s on load, 0.1s on tests)
16 tests, 0 failures

Randomized with seed 537537
```

这为我们接下来的测试搭了一个很好的架子，我们可以完善其中的细节，并添加一些我们自己的测试。
