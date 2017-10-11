
### 起步

第一章的目标是让我们的`Phoenix`应用尽快跑起来。

在我们开始前，让我们先来看看 [phoenix 的 安装指南](http://www.phoenixframework.org/docs/installation), 在安装了必要的依赖后，我们就可以躁起来了。

现在，我们的机器上应该已经安装好了 `Elixir`, `Erlang`, `Hex` 和 `Phoenix工具集`, 同时 `PostgreSQL`和`Node.js`也应该准备就绪了。

让我们开始吧！

我们可以在任意目录运行` mix phx.new` 来建立一个工程，`Phoenix`可以识别绝对或相对
路径，假设项目名字为 hello，运行如下命令:

```console
$ mix phx.new hello
```

> 关于 [Brunch.io](http://brunch.io/): Phoenix 默认使用 Brunch.io 作为web资源管理工具，Brunch.io的安装是通过node.js而不是自身的mix, Brunch.io的依赖项会在我们执行 mix phx.new 后提示我们安装，如果我们输入 no 并且之后也不通过 npm 手动安装 Brunch, 我们的应用在启动的时候就会报错，我们的资源也不会正确的载入,如果你不需要Brunch.io，你可以在建立工程的时候传递 --no-brunch 参数, 比如: 'mix phx.new --no-brunch'.

> 关于 [Ecto](https://hexdocs.pm/phoenix/ecto.html): Ecto 允许我买的 phoenix 应用与数据库交互，比如 PostgreSQL 或者 MongoDB 等等。如果你的应用不需要，可以在 创建应用的时候使用 `--no-ecto`, 比如 'nex phx.new --no-ecto'。另外该参数可以同时一起和 `--no-brunch` 使用fd



```console
mix phx.new hello
* creating hello/config/config.exs
* creating hello/config/dev.exs
* creating hello/config/prod.exs
...
* creating hello/lib/hello_web/views/layout_view.ex
* creating hello/lib/hello_web/views/page_view.ex

Fetch and install dependencies? [Yn]
```

Phoenix 会为我们生成应用所需的所有文件，之后会询问我们是否安装相应的依赖，选择 yes

```console
Fetch and install dependencies? [Yn] Y
* running mix deps.get
* running mix deps.compile
* running cd assets && npm install && node node_modules/brunch/bin/brunch build

We are all set! Go into your application by running:

    $ cd hello

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server
```

一旦依赖安装完毕，命令行会提示我们进入项目目录,并启动应用。

Phoenix 会假设我们的 PostgreSQL 数据库有一个 `postgres` 账户（有相应的权限和密码），如果你对这个有疑问，
请查看 [ecto.create](phoenix_mix_tasks.html#ecto-specific-mix-tasks) 任务.

进入我们刚才创建的项目目录。

```bash
$ cd hello
```

然后创建项目的数据库。

```
$ mix ecto.create
The database for Hello.Repo has been created.
```

> 注意，如果你是第一次运行这个命令， Phoenix 会问你是否安装 Rebar,这是编译 Erlang 包的工具。按照提示安装即可.

最后，我们来启动 Phoenix 服务器。

```console
$ mix phx.server
[info] Running HelloWeb.Endpoint with Cowboy using http://0.0.0.0:4000
19:30:43 - info: compiled 6 files into 2 files, copied 3 in 2.1 sec
```

如果之前创建项目时没有安装对应的依赖，Phoenix 会在接下来的步骤之前提示我们是否安装。

```console
Fetch and install dependencies? [Yn] n

We are almost there! The following steps are missing:

    $ cd hello
    $ mix deps.get
    $ cd assets && npm install && node node_modules/brunch/bin/brunch build

Then configure your database in config/dev.exs and run:

    $ mix ecto.create

Start your Phoenix app with:

    $ mix phx.server

You can also run your app inside IEx (Interactive Elixir) as:

    $ iex -S mix phx.server
```

接下来用你最喜欢的浏览器打开 [http://localhost:4000](http://localhost:4000) ，你应该能看到 Phoenix 在欢迎你了。

[![Gyazo](https://i.gyazo.com/746a6528188bd543b2bc63fda6c88161.png)](https://gyazo.com/746a6528188bd543b2bc63fda6c88161)


如果你没有看到上面的页面,试试 [http://127.0.0.1:4000](http://127.0.0.1:4000) ， 然后确保你的操作系统的'localhost' 指向 '127.0.0.1'

现在，我们的应用跑在本地的一个 iex session 中， 我们可以按 ctrl-c 两次来停止应用。

下一步, 让我们做一些自定义的设置，了解一下 Phoenix 大致是怎样工作的。


