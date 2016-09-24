
# mix 任务

新生成的 Phoenix 应用中有很多内建的针对 Phoenix 以及 Ecto 的 mix 任务。同时，我们也可以根据需求自己创建。

## Phoenix 相关的 Mix 任务

```console
$ mix help | grep -i phoenix
mix phoenix.digest      # 编译静态资源
mix phoenix.gen.channel # 生成一个 phoenix 的通道
mix phoenix.gen.html    # 为指定资源生成控制器，模型，视图。
mix phoenix.gen.json    # 为一个资源生成控制器以及模型。
mix phoenix.gen.model   # 生成一个模型
mix phoenix.new         # 生成一个 phoenix 应用。
mix phoenix.routes      # 打印所有路由
mix phoenix.server      # 启动应用服务器
```

我们在之前的章节中已经或多或少的使用过它们，现在让我们再来仔细的看看。

#### `mix phoenix.new`

告诉 Phoenix 给我们生成一个应用，就像我们之前在
[起步指南](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/A_%E8%B5%B7%E6%AD%A5.html)
看到的那样。

在我们开始之前，我们需要知道 Phoenix 默认使用 [Ecto](https://github.com/elixir-lang/ecto) 作为 ORM, 用 [Brunch.io](http://brunch.io/)
作为资源编译打包工具。我们可以使用 `--no-ecto` 和 `--no-brunch` 来移除它们。

> 注意: 如果我们使用 Brunch, 我们需要在使用前先安装它。
> `phoenix.new` 默认会询问是否安装，当然你也可以手动使用 npm install 来安装。 否侧在编译打包资源的时候会报错。

我们需要给 `phoenix.new` 任务起个名字，惯例是使用小字母中间带下划线。

```console
$ mix phoenix.new task_tester
* creating task_tester/.gitignore
. . .
```

我们也可以使用相对或绝对路径。

相对路径的例子：

```console
$ mix phoenix.new ../task_tester
* creating ../task_tester/.gitignore
. . .
```

绝对路径的例子：

```console
$ mix phoenix.new /Users/me/work/task_tester
* creating /Users/me/work/task_tester/.gitignore
. . .
```

`phoenix.new` 会询问是否帮你安装依赖。

```console
Fetch and install dependencies? [Yn] y
* running npm install && node node_modules/brunch/bin/brunch build
* running mix deps.get
```

所以依赖安装完成后， `phoenix.new` 会告诉我们下一步：

```console
We are all set! Run your Phoenix application:

$ cd task_tester
$ mix phoenix.server

You can also run it inside IEx (Interactive Elixir) as:

$ iex -S mix phoenix.server
```

默认情况下 `phoenix.new` 会假设我们使用 ecto 来操作我们的模型。如果我们不想使用 `ecto` 使用 `--no-ecto` 选项即
可。

```console
$ mix phoenix.new task_tester --no-ecto
* creating task_tester/.gitignore
. . .
```

Phoenix 将不会把 ecto 或者 postgrex 作为我们项目的依赖，并不会创建 `repo.ex` 文件。

默认情况下， Phoenix 会将我们传递给 `phoenix.new` 的名字作为我们 OTP 应用的名字。如果你愿意，也可以使用
`--app` 选项指定一个不同的 OTP 应用的名字。

```console
$  mix phoenix.new task_tester --app hello_phoenix
* creating task_tester/config/config.exs
* creating task_tester/config/dev.exs
* creating task_tester/config/prod.exs
* creating task_tester/config/prod.secret.exs
* creating task_tester/config/test.exs
* creating task_tester/lib/hello_phoenix.ex
* creating task_tester/lib/hello_phoenix/endpoint.ex
* creating task_tester/priv/static/robots.txt
* creating task_tester/test/controllers/page_controller_test.exs
* creating task_tester/test/views/error_view_test.exs
* creating task_tester/test/views/page_view_test.exs
* creating task_tester/test/support/conn_case.ex
* creating task_tester/test/support/channel_case.ex
* creating task_tester/test/test_helper.exs
* creating task_tester/web/controllers/page_controller.ex
* creating task_tester/web/templates/layout/app.html.eex
* creating task_tester/web/templates/page/index.html.eex
* creating task_tester/web/views/error_view.ex
* creating task_tester/web/views/layout_view.ex
* creating task_tester/web/views/page_view.ex
* creating task_tester/web/router.ex
* creating task_tester/web/web.ex
* creating task_tester/mix.exs
* creating task_tester/README.md
* creating task_tester/lib/hello_phoenix/repo.ex
. . .
```

如果我们打开 `mix.exs` 文件，我们会发现目前项目的名字是 `hello_phoenix`。

```elixir
defmodule HelloPhoenix.Mixfile do
  use Mix.Project

  def project do
    [app: :hello_phoenix,
    version: "0.0.1",
. . .
```

大略看一下，我们的模块名都符合 `HelloPhoenix`。

```elixir
defmodule HelloPhoenix.PageController do
  use HelloPhoenix.Web, :controller
. . .
```

我们会发现项目名字在很多地方都有体现 -- 比如 `lib/` 下的文件, 测试文件等 -- 名字上都带有 `hello_phoenix`。

```console
* creating task_tester/lib/hello_phoenix.ex
* creating task_tester/lib/hello_phoenix/endpoint.ex
* creating task_tester/lib/hello_phoenix/repo.ex
* creating task_tester/test/hello_phoenix_test.exs
```

如果我们想改变模块的名字，可以使用 `--module` 标记。 注意传递给 `--module` 的名字必须是符合大写规范的，否则会报错。

```console
$  mix phoenix.new task_tester --module HelloPhoenix
* creating task_tester/config/config.exs
* creating task_tester/config/dev.exs
* creating task_tester/config/prod.exs
* creating task_tester/config/prod.secret.exs
* creating task_tester/config/test.exs
* creating task_tester/lib/task_tester.ex
* creating task_tester/lib/task_tester/endpoint.ex
* creating task_tester/priv/static/robots.txt
* creating task_tester/test/controllers/page_controller_test.exs
* creating task_tester/test/views/error_view_test.exs
* creating task_tester/test/views/page_view_test.exs
* creating task_tester/test/support/conn_case.ex
* creating task_tester/test/support/channel_case.ex
* creating task_tester/test/test_helper.exs
* creating task_tester/web/controllers/page_controller.ex
* creating task_tester/web/templates/layout/app.html.eex
* creating task_tester/web/templates/page/index.html.eex
* creating task_tester/web/views/error_view.ex
* creating task_tester/web/views/layout_view.ex
* creating task_tester/web/views/page_view.ex
* creating task_tester/web/router.ex
* creating task_tester/web/web.ex
* creating task_tester/mix.exs
* creating task_tester/README.md
* creating task_tester/lib/task_tester/repo.ex
. . .
```

注意所有和项目相关的文件名都是 `task_tester`。因为 `mix.exs` 中的项目名就是 `task_tester`, 但是左右的模块名是以 `HelloPhoenix`
开头的。

```elixir
defmodule HelloPhoenix.Mixfile do
  use Mix.Project

  def project do
    [app: :task_tester,
. . .
```

#### `mix phoenix.gen.html`

Phoenix 现在能通过生成器生成一个 HTML 应用的所有部分 -  Ecto 迁移，Ecto 模型, 控制器以及 actions, 视图，和模板，
这将给你节省很多时间。让我们来仔细看看。

`phoenix.gen.html` 需要一些参数， model 模型的名字， 资源名字和一串 `列：类型` 的属性。模块名字必须要符合  Elixir 关于
模型的命名规范, 即首字母大写。。

```console
$ mix phoenix.gen.html Post posts body:string word_count:integer
* creating priv/repo/migrations/20150523120903_create_post.exs
* creating web/models/post.ex
* creating test/models/post_test.exs
* creating web/controllers/post_controller.ex
* creating web/templates/post/edit.html.eex
* creating web/templates/post/form.html.eex
* creating web/templates/post/index.html.eex
* creating web/templates/post/new.html.eex
* creating web/templates/post/show.html.eex
* creating web/views/post_view.ex
* creating test/controllers/post_controller_test.exs
```

当 `phoenix.gen.html` 生成这些文件后，会提示你添加路由以及运行迁移任务。

```console
Add the resource to your browser scope in web/router.ex:

    resources "/posts", PostController

and then update your repository by running migrations:

$ mix ecto.migrate
```

注意： 如果我们不按照提示的做，我们的应用将不会被编译，并且还会报错。

```console
$ mix phoenix.server
Compiled web/models/post.ex

== Compilation error on file web/controllers/post_controller.ex ==
** (CompileError) web/controllers/post_controller.ex:27: function post_path/2 undefined
(stdlib) lists.erl:1336: :lists.foreach/2
(stdlib) erl_eval.erl:657: :erl_eval.do_apply/6
```

如果我们不希望给我们的资源创建模型，我们可以使用 `--no-model` 选项。

```console
$ mix phoenix.gen.html Post posts body:string word_count:integer --no-model
* creating web/controllers/post_controller.ex
* creating web/templates/post/edit.html.eex
* creating web/templates/post/form.html.eex
* creating web/templates/post/index.html.eex
* creating web/templates/post/new.html.eex
* creating web/templates/post/show.html.eex
* creating web/views/post_view.ex
* creating test/controllers/post_controller_test.exs
```

这时会让我们添加路由，但是不会提示我们运行 `ecto.migrate`, 因为我们跳过了创建模型。

```console
Add the resource to your browser scope in web/router.ex:

    resources "/posts", PostController
```

同样的，如果我们不按照提示来，会报错。

```console
$ mix phoenix.server

== Compilation error on file web/views/post_view.ex ==
** (CompileError) web/templates/post/edit.html.eex:4: function post_path/3 undefined
    (stdlib) lists.erl:1336: :lists.foreach/2
    (stdlib) erl_eval.erl:657: :erl_eval.do_apply/6
```

#### `mix phoenix.gen.json`

Phoenix 同样为我们准备了 JSON 资源的生成器 -  Ecto 迁移， Ecto 模型， 控制器、视图等。但是这个生成器不会生成模板文件。

`phoenix.gen.json` 生成器需要一些参数，模型的模块名 (the module name of the model), 资源名和一串 `列：类型` 的属性。
模块名字必须要符合 Elixir 中模块的命名规范 , 即首字母大写。

```console
$ mix phoenix.gen.json Post posts title:string content:string
* creating priv/repo/migrations/20150521140551_create_post.exs
* creating web/models/post.ex
* creating test/models/post_test.exs
* creating web/controllers/post_controller.ex
* creating web/views/post_view.ex
* creating test/controllers/post_controller_test.exs
* creating web/views/changeset_view.ex
```

当 `phoenix.gen.json` 生成这些文件后，会提示你添加路由以及运行迁移任务。

```console
Add the resource to your api scope in web/router.ex:

    resources "/posts", PostController, except: [:new, :edit]

and then update your repository by running migrations:

    $ mix ecto.migrate
```

注意： 如果我们不按照提示的做，我们的应用将不会被编译，并且还会报错。

```console
$ mix phoenix.server
Compiled web/models/post.ex

== Compilation error on file web/controllers/post_controller.ex ==
** (CompileError) web/controllers/post_controller.ex:27: function post_path/2 undefined
(stdlib) lists.erl:1336: :lists.foreach/2
(stdlib) erl_eval.erl:657: :erl_eval.do_apply/6
```

如果我们不希望给我们的资源创建模型，我们可以使用 `--no-model` 选项。

```console
$ mix phoenix.gen.json Post posts title:string content:string --no-model
* creating web/controllers/post_controller.ex
* creating web/views/post_view.ex
* creating test/controllers/post_controller_test.exs
* creating web/views/changeset_view.ex
```

这时会让我们添加路由，但是不会提示我们运行 `ecto.migrate`, 因为我们跳过了创建模型。

```console
Add the resource to your api scope in web/router.ex:

    resources "/posts", PostController, except: [:new, :edit]
```

注意： 如果我们不按照提示的做，我们的应用将不会被编译，并且还会报错。

```console
$ mix phoenix.server

== Compilation error on file web/controllers/post_controller.ex ==
** (CompileError) web/controllers/post_controller.ex:15: HelloPhoenix.Post.__struct__/0 is undefined, cannot expand struct HelloPhoenix.Post
    (elixir) src/elixir_map.erl:55: :elixir_map.translate_struct/4
    (stdlib) lists.erl:1352: :lists.mapfoldl/3
```

#### `mix phoenix.gen.model`

如果我们只想创建一个模型，我们可以使用 `phoenix.gen.model` 任务。它会生成一个模型，一个迁移任务以及一个测试用
例。

`phoenix.gen.model` 生成器需要一些参数， 模型的模块名称，复数形式的模型名称（用于 schema）, 以及一串 `列：类型` 的属性。

```console
$ mix phoenix.gen.model User users name:string age:integer
* creating priv/repo/migrations/20150527185323_create_user.exs
* creating web/models/user.ex
* creating test/models/user_test.exs
```

> 注意：如果我们想要给资源一个命名空间，我们只需要将第一个参数之前加上命名空间即可。

```console
$ mix phoenix.gen.model Admin.User users name:string age:integer
* creating priv/repo/migrations/20150527185940_create_admin_user.exs
* creating web/models/admin/user.ex
* creating test/models/admin/user_test.exs
```

#### `mix phoenix.gen.channel`

这个生成器会产生一个基本的 Phoenix 通道以及相应的测试用例。channel 接收一个
 module 名字作为参数：

```console
$ mix phoenix.gen.channel Room
* creating web/channels/room_channel.ex
* creating test/channels/room_channel_test.exs
```

当 `phoenix.gen.channel` 执行完毕，它会提示你在路由中添加相关信息。

```console
Add the channel to your `web/channels/user_socket.ex` handler, for example:

    channel "rooms:lobby", HelloPhoenix.RoomChannel
```

#### `mix phoenix.routes`

这个任务将列出我们给定路由中的所有路由信息，我们已经在之前的
[路由指南](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/C_%E8%B7%AF%E7%94%B1.html)
用过很多次了。

如果我们不指定具体路由，他会将当前应用作为默认的。

```console
$ mix phoenix.routes
page_path  GET  /  TaskTester.PageController.index/2
```

如果我们的工程中有多个应用，我们可以指定具体的一个
(We can also specify an individual router if we have more than one for our application.) ：

```console
$ mix phoenix.routes TaskTester.Router
page_path  GET  /  TaskTester.PageController.index/2
```

#### `mix phoenix.server`

这个任务将我们的应用启动起来，不需要任何参数。

```console
$ mix phoenix.server
[info] Running TaskTester.Endpoint with Cowboy on port 4000 (http)
```

如果你传递了参数它会默认忽略, 不产生错误或警告信息。

```console
$ mix phoenix.server DoesNotExist
[info] Running TaskTester.Endpoint with Cowboy on port 4000 (http)
```

在之前版本的 (0.8.x) Phoenix 中，我们是使用 `phoenix.start` 来启动应用, 这个任务已经废弃了，尝试使用会报错。

```console
$ mix phoenix.start
** (Mix) The task phoenix.start could not be found
```

我们还可以使用 `iex -S mix phoenix.server` 在 `iex` 中启动我们的应用。

```console
$ iex -S mix phoenix.server
Erlang/OTP 17 [erts-6.4] [source] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

[info] Running TaskTester.Endpoint with Cowboy on port 4000 (http)
Interactive Elixir (1.0.4) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)>
```

#### `mix phoenix.digest`

这个任务做两件事情， 给我们的静态资源创建一个资源并压缩它们。

"Digest" 具体是在我们的静态资源文件名后加一个 MD5 校验码，类似于一个指纹，如果这个"指纹"没有变化，浏览器和
`CDN` 会缓存这个文件，如果文件内容改变了，才会重新获取。

在看细节之前我们先来看看 Hello_phoenix 中的 两个目录。

首先是 `priv/static` 目录：

```text
├── images
│   └── phoenix.png
├── robots.txt
```

以及 `web/static` 目录：

```text
├── css
│   └── app.css
├── js
│   └── app.js
├── vendor
│   └── phoenix.js
```

这些都是我们的静态资源，现在我们试试运行 `mix phoenix.digest` 任务。

```console
$ mix phoenix.digest
Check your digested files at 'priv/static'.
```

我们可以按照提示来看看 `priv/static` 目录里的内容。 我们看到 `web/static/` 目录下的文件已经都拷贝到了这里，并
且每一个文件都有几个版本， 它们是：

* 原文件。
* gzip 压缩过的文件。
* 在文件名中包含指纹的文件。
* 压缩过的、在文件名中包含指纹的文件。

我们还可以使用 `:gzippable_exts` 选项来对某些后缀的文件启用 gzip 压缩 :

```elixir
config :phoenix, :gzippable_exts, ~w(.js .css)
```

> 注意: 我们可以给 `phoenix.digest` 任务指定目标编译目录（第一个参数） -- 如果我们想把他们放在其他目录的话。

```console
$ mix phoenix.digest priv/static -o www/public
Check your digested files at 'www/public'.
```

## Ecto 相关的 Mix 任务

默认生成的 Phoenix 应用包含 ecto 以及 postgrex 依赖 （除非我们使用了 `--no-ecto` 选项），mix 中包含了一些通用
的 Ecto 处理任务，让我们来一探究竟。

```console
$ mix help | grep -i ecto
mix ecto.create          # Create the storage for the repo
mix ecto.drop            # Drop the storage for the repo
mix ecto.gen.migration   # Generate a new migration for the repo
mix ecto.gen.repo        # Generates a new repository
mix ecto.migrate         # Runs migrations up on a repo
mix ecto.rollback        # Reverts migrations down on a repo
```

注意：我们可以给这些任务加上 `--no-start` 选项，让其光执行任务而不启动我们的应用。

#### `ecto.create`

这个任务会创建应用中指定的repo。 默认情况下repo 的名字会跟随应用的名字，但是我们也可以自己指定。

让我们看个实际的例子。

```console
$ mix ecto.create
The database for HelloPhoenix.Repo has been created.
```

如果我们要给另一个叫 `OurCustom.Repo` 的repo 创建数据库，我们可以这么做：

```console
$ mix ecto.create -r OurCustom.Repo
The database for OurCustom.Repo has been created.
```

运行 `ecto.create` 时可能会产生一些错误，比如我们的数据库没有一个 "postgres" 用户，我们会得到如下错误：

```console
$ mix ecto.create
** (Mix) The database for HelloPhoenix.Repo couldn't be created, reason given: psql: FATAL:  role "postgres" does not exist
```

我们可以在 `psql` 终端里创建 "postgres" 角色(用户)，并分配给其登陆以及创建数据库的权限。

```console
=# CREATE ROLE postgres LOGIN CREATEDB;
CREATE ROLE
```

如果 "postgres" 角色没有登陆的权限，会得到如下错误：

```console
$ mix ecto.create
** (Mix) The database for HelloPhoenix.Repo couldn't be created, reason given: psql: FATAL:  role "postgres" is not permitted to log in
```

解决办法是赋予 "postgres" 登陆的权限。

```console
=# ALTER ROLE postgres LOGIN;
ALTER ROLE
```

如果 "postgres" 角色没有创建数据库的权限，会得到如下错误：

```console
$ mix ecto.create
** (Mix) The database for HelloPhoenix.Repo couldn't be created, reason given: ERROR:  permission denied to create database
```

解决办法是赋予 "postgres" 创建数据库的权限。

```console
=# ALTER ROLE postgres CREATEDB;
ALTER ROLE
```

如果 "postgres" 角色使用的密码不是默认的 "postgres", 会得到如下错误：

```console
$ mix ecto.create
** (Mix) The database for HelloPhoenix.Repo couldn't be created, reason given: psql: FATAL:  password authentication failed for user "postgres"
```

To fix this, we can change the password in the environment specific configuration file. For the development
environment the password used can be found at the bottom of the `config/dev.exs` file.

#### `ecto.drop`

这个任务会删除我们 repo 指定的数据库，默认它会找和我们应用相同名字的那个，这个操作不会有提示，所以你一定要谨慎。

```console
$ mix ecto.drop
The database for HelloPhoenix.Repo has been dropped.
```

你可以使用 `-r` 选项指定一个不同的数据库。

```console
$ mix ecto.drop -r OurCustom.Repo
The database for OurCustom.Repo has been dropped.
```

#### `ecto.gen.repo`

很多应用需要不止一个数据库存储，对于每一个，我们需要创建一个新的 repo, 我们可以使用 `ecto.gen.repo` 自动创建。

如果我们将 repo 命名为 `OurCustom.Repo`, 创建的文件会在这里 `lib/our_custom/repo.ex`：

```console
$ mix ecto.gen.repo -r OurCustom.Repo
* creating lib/our_custom
* creating lib/our_custom/repo.ex
* updating config/config.exs
Don't forget to add your new repo to your supervision tree
(typically in lib/hello_phoenix.ex):

worker(OurCustom.Repo, [])
```

注意这个任务同时更新了 `config/config.exs` 文件。 它给我们的新 repo 增加了一个额外的配置块。

```elixir
. . .
config :hello_phoenix, OurCustom.Repo,
adapter: Ecto.Adapters.Postgres,
database: "hello_phoenix_repo",
username: "user",
password: "pass",
hostname: "localhost"
. . .
```

当然，我们需要自己更改一下登陆信息已经环境配置信息。

同时，我们根据提示打开 `lib/hello_phoenix.ex` 文件将新 repo 加入到监视树(supervision tree)中, 并将 repo 当作一
个 worker 加入的 `children` 列表中。

```elixir
. . .
children = [
  # Start the endpoint when the application starts
  supervisor(HelloPhoenix.Endpoint, []),
  # Start the Ecto repository
  worker(HelloPhoenix.Repo, []),
  # Here you could define other workers and supervisors as children
  # worker(HelloPhoenix.Worker, [arg1, arg2, arg3]),
  worker(OurCustom.Repo, []),
]
. . .
```

#### `ecto.gen.migration`

`迁移(Migrations)` 是一种编程的、可重复的改变数据库 schema 的方法。 `迁移` 同时实现上仅仅是模块，并且我们可以
使用 `ecto.gen.migration` 来生成。 让我们来给一个新的评论表创建一个迁移任务。

我们只需要一个 `snake_case` 规格的名字作为参数即可, 最佳实践是描述这个迁移任务的目的：

```console
mix ecto.gen.migration add_comments_table
* creating priv/repo/migrations
* creating priv/repo/migrations/20150318001628_add_comments_table.exs
```

注意迁移任务以一个日期时间字符串作为前缀。

我们来看一下 `ecto.gen.migration` 给我们创建的 `priv/repo/migrations/20150318001628_add_comments_table.exs`。

```elixir
defmodule HelloPhoenix.Repo.Migrations.AddCommentsTable do
  use Ecto.Migration

  def change do
  end
end
```

注意，迁移和回滚的部分都会由这里的 `change/0` 函数处理。我们可以使用 ecto 的领域专属语言(DSL) 来描述我们的
schema 的变动。 Ecto 会自动完成剩下的工作，无论是向前还是向后迁移 (whether we are rolling forward or rolling back)。

我们要做的是创建一个包含 `body` 、 `word_count` 以及 timestamp （包含`inserted_at` & `update_at`） 时间列的 `comments`数据表.

```elixir
. . .
def change do
  create table(:comments) do
    add :body,       :string
    add :word_count, :integer
    timestamps
  end
end
. . .
```

同样的，我们可以使用 `-r` 选项指定具体的 repo 。

```console
$ mix ecto.gen.migration -r OurCustom.Repo add_users
* creating priv/repo/migrations
* creating priv/repo/migrations/20150318172927_add_users.exs
```

更多关于 Ecto 迁移的领域专属语言 (DSL), 可以[查阅文档](http://hexdocs.pm/ecto/Ecto.Migration.html)。

现在我们可以运行我们的迁移任务了！

#### `ecto.migrate`

当我们的迁移模块写好了后，只要运行 `mix ecto.migrate` 应用到数据库即可。

```console
$ mix ecto.migrate
[info] == Running HelloPhoenix.Repo.Migrations.AddCommentsTable.change/0 forward
[info] create table comments
[info] == Migrated in 0.1s
```

当我们第一次运行 `ecto.migrate`, 它会创建一个叫 `schema_migrations` 的数据表。它的作用是记录我们运行的每一次
迁移任务, 并按照 migration 文件名上的时间戳来排序。

`schema_migrations` 表看上去是这个样子的。

```console
hello_phoenix_dev=# select * from schema_migrations;
version     |     inserted_at
----------------+---------------------
20150317170448 | 2015-03-17 21:07:26
20150318001628 | 2015-03-18 01:45:00
(2 rows)
```

当我们使用 `ecto.rollback` 回滚一个迁移任务时， 同时会将 `schema_migrations` 对应的记录移除掉。

默认情况下，`ecto.migrate` 任务会执行所有处于 pending 状态的迁移任务。 我们可以通过使用一些选项来精确的控制迁
移任务。

我们可以使用 `-n` 或者 `--step` 来明确执行处于 pending 状态的迁移任务的条数。

```console
$ mix ecto.migrate -n 2
[info] == Running HelloPhoenix.Repo.Migrations.CreatePost.change/0 forward
[info] create table posts
[info] == Migrated in 0.0s
[info] == Running HelloPhoenix.Repo.Migrations.AddCommentsTable.change/0 forward
[info] create table comments
[info] == Migrated in 0.0s
```

`--step` 作用是一样的。

```console
mix ecto.migrate --step 2
```

我们也可以指定具体的一个迁移任务，用 `-v` 选项。

```console
mix ecto.migrate -v 20150317170448
```

`--to` 的作用是一样的。

```console
mix ecto.migrate --to 20150317170448
```

#### `ecto.rollback`

`ecto.rollback` 任务会回滚我们最近运行的一次迁移任务，撤销 schema 的改动。 `ecto.migrate` 是 `ecto.rollback`
作业是相反的。

```console
$ mix ecto.rollback
[info] == Running HelloPhoenix.Repo.Migrations.AddCommentsTable.change/0 backward
[info] drop table comments
[info] == Migrated in 0.0s
```

`ecto.rollback` 和  `ecto.migrate` 接收同样的参数, `-n`, `--step`, `-v` 和 `--to` 选项与其在 `ecto.migrate` 中所表达
的意思是一样的。

## 自定义 Mix 任务

尽管 mix 以及一些第三方工具提供了很多有用的 mix 任务，但有时还是无法满足我们的一些特殊的针对性的需求， 不出意
外的，mix 允许我们自定义任务。 我们现在就来一探究竟。

首先我们需要在 `lib` 目录下创建一个 `mix/tasks` 目录。 这是存放我们自定义任务的地方。

```console
$ mkdir -p lib/mix/tasks
```

让我们在那里创建一个 `hello_phoenix.greeting.ex` 文件。

```elixir
defmodule Mix.Tasks.HelloPhoenix.Greeting do
  use Mix.Task

  @shortdoc "Sends a greeting to us from Hello Phoenix"

  @moduledoc """
    This is where we would put any long form documentation or doctests.
  """

  def run(_args) do
    Mix.shell.info "Greetings from the Hello Phoenix Application!"
  end

  # We can define other functions as needed here.
end
```

我们来快速浏览一下。

首先我们要给模块起个名字，为了给其一个合适的命名空间，我们以 `Mix.Tasks` 开头, 我们想以 `mix
hello_phoenix.greeting` 来运行这个`任务`，所以我们再加上 `HelloPhoenix.Greeting`

`use Mix.Task` 这行将 mix 的一些功能带入这里，使其行为表现上像一个 mix 任务。

模块属性 `@shortdoc` 定义了一个描述字符串，当我们运行 `mix help` 时会看到。

`@moduledoc` 和在其他模块中看到的功能一样，较长的描述信息会被放置在这里。

`run/1` 函数是 mix 任务的关键部分。 当用户触发`任务`，所有的工作都在这里完成，目前，我们只是简单的打印了一条
greeting 信息， 但是我们可以在 `run/1` 中实现我们的任何需求。 注意 `Mix.shell.info/1` 是将信息打印给用户的一个
惯例用法。

当然，我们的`任务`也仅仅是一个模块，所以你也可以在其中定义私有函数来将 `run/1` 分解开来编写。

任务模块编写完了，下一步是编译应用。

```console
$ mix compile
Compiled lib/tasks/hello_phoenix.greeting.ex
Generated hello_phoenix.app
```

现在我们应该可以在 `mix help` 中看到我们编写的任务了。

```console
$ mix help | grep hello
mix hello_phoenix.greeting # Sends a greeting to us from Hello Phoenix
```

`mix help` 会显示我们定义在 `@shortdoc` 中的信息。

在看看是否工作正常？

```console
$ mix hello_phoenix.greeting
Greetings from the Hello Phoenix Application!
```

成功了！

如果想在我们的 `任务` 中使用整个项目的基础设施(infrastructure), 我们需要先启动应用，这在比如想在 `任务` 中使
用数据库时会非常有用。感谢上帝， 这在 mix 中非常容易实现：

```elixir
  . . .
  def run(_args) do
    Mix.Task.run "app.start"
    Mix.shell.info "Now I have access to Repo and other goodies!"
  end
  . . .
```
