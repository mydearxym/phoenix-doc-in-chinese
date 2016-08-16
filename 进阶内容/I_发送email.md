# 发送 Email

在 Phoenix 应用中发送 email 很容易。虽然 Phoenix 没有内建的发送 email 的功能，但是有很多第三方的包可供选择。

比如 [Phoenix Swoosh](https://github.com/swoosh/phoenix_swoosh) 和 [Bamboo](https://github.com/thoughtbot/bamboo)。

如果你想使用 Mailgun 服务，你也可以找到他的包：

## 使用 Mailgun

在开始之前，我们需要在 Mailgun 上注册一个账户 -- 当然我们不注册也可以发送邮件。
有了账户之后，剩下的步骤就很直观了。

首先，[注册 Mailgun](https://mailgun.com/signup)。它每月送你很多免费邮件，所以目前免费账号就够了。

账号就绪以后，我们会得到一个发送邮件的沙箱 (sandbox)。 沙箱的 url 默认是我们的域名，我们也可以通过 Mailgun 自
定义一个。

我们还需要在工程中添加 `mailgun` 依赖。 在 `mix.exs` 文件的 `deps/0` 的函数中。

```elixir
defp deps do
  [{:phoenix, "~> 1.2.0"},
   {:phoenix_ecto, "~> 3.0"},
   {:postgrex, ">= 0.0.0"},
   {:phoenix_html, "~> 2.3"},
   {:phoenix_live_reload, "~> 1.0", only: :dev},
   {:cowboy, "~> 1.0"},
   {:mailgun, "~> 0.1.2"}]
end
```

下一步，我们运行 `mix deps.get` 安装依赖。为了不和 `Poison` 包产生冲突，我们添加下面这行：

```
{:poison, "~> 2.1", override: true}
```

### 配置

我们需要在 `config/config.ex` 文件中配置 `:mailgun_domain` 和 `:mailgun_key`。

`:mailgun_domain` 是一个完整的 url , 比如`https://api.mailgun.net/v3/sandbox-our-domain.mailgun.org`.
`:mailgun_key` 则是一个长字串， 比如 "key-another-long-string".

处于安全原因，这些配置最好不要上传到公共的代码仓库，我们可以使用一些方法：

一种是直接在环境变量(包括开发环境，生产环境)中指定 `:mailgun_domain` 和  `:mailgun_key`，然后在配置文件
`config/config.exs` 中引用他们。

```elixir
config :hello_phoenix,
       mailgun_domain: System.get_env("MAILGUN_DOMAIN"),
       mailgun_key: System.get_env("MAILGUN_API_KEY")
```

还有一种稍微有点麻烦的办法。仿照 `config/prod.secret.exs` 文件创建一个 `config/config.secret.exs` -- 我们不使
用 `prod.secret.exs` 文件的原因是他仅仅针对生产环境起效果，而 `config.secret.exs` 作用于所有环境。

首先是在 `.gitignore` 中忽略 `config/config.secret.exs` 文件。

```elixir
. . .
# The config/prod.secret.exs file by default contains sensitive
# data and you should not commit it into version control.
#
# Alternatively, you may comment the line below and commit the
# secrets file as long as you replace its contents by environment
# variables.
/config/prod.secret.exs
/config/config.secret.exs
```

然后在 `config/config.secret.exs` 文件中配置 `mailgun`。

```elixir
use Mix.Config

config :hello_phoenix,
       mailgun_domain: "https://api.mailgun.net/v3/sandbox-our-domain.mailgun.org",
       mailgun_key: "key-another-long-string"
```

最后， 在 `config/config.exs` 中引入 `config.secret.exs` 文件。

```elixir
. . .
# Import environment specific config. This must remain at the bottom
# of this file so it overrides the configuration defined above.
import_config "#{Mix.env}.exs"
import_config "config.secret.exs"
```

因为`config/config.secret.exs` 不会出现在我们的代码仓库中，我们需要在部署是做一些额外的步骤，具体请查看
[Deployment Introduction Guide](http://www.phoenixframework.org/docs/deployment)

### 客户端模块

为了让我们的应用和 Mailgun 交互，我们来创建一个客户端模块 -- `lib/hello_phoenix/mailer.ex`。在第二行 `use`
`Mailgun.Client` 后，我们将配置信息传递给 `mailgun` 包，然后引入 `mailgun` 的 `send_email/1` 函数。

```elixir
defmodule HelloPhoenix.Mailer do
  use Mailgun.Client,
      domain: Application.get_env(:hello_phoenix, :mailgun_domain),
      key: Application.get_env(:hello_phoenix, :mailgun_key)
end
```

> 注意，我们的应用不会监视在 `lib` 目录中的文件改动，所以如果你改动了这里的代码，需要重新启动应用才能有效果。

接下来，我们需要编写发送 email 的函数。 web 应用会发送各种邮件 -- 欢迎邮件，密码重置，消息提醒等等。对于每一种
邮件，我们需要使用上面引入的`send_email/1` 来创建一个函数。

比如我们发送一封纯文本的欢迎邮件, 使用 `:text` 选项 -- 其他选项看字面意思很好理解。

```elixir
def send_welcome_text_email(email_address) do
  send_email to: email_address,
             from: "us@example.com",
             subject: "Welcome!",
             text: "Welcome to HelloPhoenix!"
end
```

现在我们可以在应用的任何地方发送这封邮件了（别忘了指定邮件地址）。

```elixir
HelloPhoenix.Mailer.send_welcome_text_email("us@example.com")
```

在我们的邮件部分变得复杂之前，我们最好先来看看怎么做测试。`mailgun` 可以允许我们在本地测试，在客户端模块，我们
可以添加 `:test` 字段并给一个接收`mailgun`响应(json 格式)的文件路径。

让我们在 `lib/hello_phoenix/mailer.ex` 中添加他们。

```elixir
defmodule HelloPhoenix.Mailer do
  use Mailgun.Client, domain: Application.get_env(:my_app, :mailgun_domain),
                      key: Application.get_env(:my_app, :mailgun_key),
                      mode: :test, # Alternatively use Mix.env while in the test environment.
                      test_file_path: "/tmp/mailgun.json"
  . . .
end
```

让我们在 `iex` 中做个测试。我们使用 `iex -S mix phoenix.server` 以便于我们正在运行的 Phoenix 应用交互。一旦我
们进入 `iex` 会话，我们就可以调用之前编写的发送欢迎邮件的函数了。

```console
$ iex -S mix phoenix.server
. . .
iex> HelloPhoenix.Mailer.send_welcome_text_email("us@example.com")
{:ok, "OK"}
```

在测试环境下, `send_email/1` 函数会总是返回 `{:ok, "OK"}` 。

现在，我们可以在文件中看到返回结果。

```console
$ more /tmp/mailgun.json
{"to":"us@example.com","text":"Welcome to HelloPhoenix!","subject":"Welcome!","from":"Mailgun Sandbox <postmaster@sandbox-our-domain.mailgun.org>"}
```

我们也可以使用 `:html` 选项发送 HTML 邮件。 `:html` 选项的值是一个 html 字符串。

```elixir
def send_welcome_html_email(email_address) do
  send_email to: email_address,
             from: "us@example.com",
             subject: "Welcome!",
             html: "<strong>Welcome to HelloPhoenix</strong>"
end
```

注意我们的代码里面 "form" 那行有重复，我们可以将其提取为模块属性。

```elixir
defmodule HelloPhoenix.Mailer do
  . . .
  @from "us@example.com"
  . . .
```

我们可以在 `:from` 后面这样使用：

```elixir
def send_welcome_text_email(email_address) do
  send_email to: email_address,
             from: @from,
             subject: "Welcome!",
             text: "Welcome to HelloPhoenix!"
end

def send_welcome_html_email(email_address) do
  send_email to: email_address,
             from: @from,
             subject: "Welcome!",
             html: "<strong>Welcome to HelloPhoenix</strong>"
end
```

当我们调用 `send_welcome_html_email/1` 函数， 我们得到的输出几乎一样，只是 html 取代了文本。

```console
$ iex -S mix phoenix.server

iex> HelloPhoenix.Mailer.send_welcome_html_email("us@example.com")
{:ok, "OK"}
```

这里是 `/tmp/mailgun.json` 中的内容。

```console
$ more /tmp/mailgun.json
{"to":"them@example.com","subject":"Welcome!","html":"<strong>Welcome to HelloPhoenix Test</strong>","from":"Mailgun Sandbox <postmaster@sandbox-our-domain.mailgun.org>"}
```

对很多 email 的使用场景来说，当客户端不能渲染 HTML 时最好能降级到纯文本渲染。 这种情况下，我们可以在函数中同时定义 `:html`
和 `:text` 字段。

```elixir
def send_welcome_email(email_address) do
  send_email to: email_address,
             from: @from,
             subject: "Welcome!",
             text: "Welcome to HelloPhoenix!",
             html: "<strong>Welcome to HelloPhoenix</strong>"
end
```

当我们测试调用这个函数，输出如下：

```console
$ more /tmp/mailgun.json

{"to":"us@example.com","text":"Welcome to HelloPhoenix!","subject":"Welcome!","html":"<strong>Welcome to HelloPhoenix Test</strong>","from":"Mailgun Sandbox <postmaster@sandbox-our-domain.mailgun.org>"}
```

让我们回到非测试模式 ， 去掉 `:mode` 和 `:test_file_path` 选项即可。

```elixir
defmodule HelloPhoenix.Mailer do
  use Mailgun.Client,
      domain: Application.get_env(:hello_phoenix, :mailgun_domain),
      key: Application.get_env(:hello_phoenix, :mailgun_key)
  . . .
```

当我们重启应用然后调用 `send_welcome_email/1` 函数，我们会从 Mailgun 得到一条信息，提示我们这封 email 已经进入
发送队列了(telling us our email has been queued)。

```console
iex> HelloPhoenix.Mailer.send_welcome_email("us@example.com")
{:ok,
 "{\n  \"id\": \"<20150820050046.numbers.more_numbers@sandbox-our-domain.mailgun.org>\",\n  \"message\": \"Queued. Thank you.\"\n}"}
```

不错！ 现在我们来看看我们的收件箱。

注意看这封 email 的源文件， 我们可以看到这封 `multipart email` 包含两个部分。 首先是带有 Contents-Type 字段
为 "text/plain" 的文本email， 第二封是带有 Content-Type 字段为 "text/html" 的 HTML 版本的 email 。

```
To: them@example.com
From: Mailgun Sandbox
 <postmaster@sandbox-our-domain.mailgun.org>
Subject: Welcome!
Mime-Version: 1.0
Content-Type: multipart/alternative; boundary="ab2eaf529cf8442b93154d6e3d98896e"

--ab2eaf529cf8442b93154d6e3d98896e
Content-Type: text/plain; charset="ascii"
Mime-Version: 1.0
Content-Transfer-Encoding: 7bit

Welcome to HelloPhoenix!
--ab2eaf529cf8442b93154d6e3d98896e
Content-Type: text/html; charset="ascii"
Mime-Version: 1.0
Content-Transfer-Encoding: 7bit

<strong>Welcome to HelloPhoenix Test</strong>
--ab2eaf529cf8442b93154d6e3d98896e--
```

### 收尾

目前为止我们做的还不错，但是对于真实世界的应用来说，这些简单的 email 内容就不够用了，随着文本或 html 的复杂，
我们的 `send_welcome_email/1` 函数很快会变得非常乱。 解决办法是将其封装在一个私有函数里, 起一个描述性的名字。

在我们的 `HelloPhoenix.Mailer` 模块里， 我们可以定义一个私有的 `welcome_text/0` 函数，使用 `heredoc` 语法定义
一串 email 的正文内容。

```elixir
. . .
defp welcome_text do
  """
  Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
  """
end
. . .
```

现在我们可以在 `send_welcome_email/1` 函数中使用：

```elixir
def send_welcome_email(email_address) do
  send_email to: email_address,
             from: @from,
             subject: "Welcome!",
             text: welcome_text,
             html: "<strong>Welcome to HelloPhoenix</strong>"
end
```

对于 html 我们也可以使用 `Phoenix.View.render_to_string/3` 将要渲染的 html 内容放在独立的文件里。

```elixir
def send_welcome_email(email_address) do
  send_email to: email_address,
             from: @from,
             subject: "Welcome!",
             text: welcome_text,
             html: Phoenix.View.render_to_string(HelloPhoenix.EmailView, "welcome.html", %{})
end
```

为了让上面的代码工作， 我们需要一些组件来渲染邮件模板。
首先，我们需要在 `web/views/email_view.ex` 文件中定义一个 `HelloPhoenix.EmailView`

```elixir
defmodule HelloPhoenix.EmailView do
  use HelloPhoenix.Web, :view
end
```

我们还需要在 `web/templates` 目录中定义一个 `email` 目录, 并创建一个 `welcome.html.eex` 模板。

```html
<div class="jumbotron">
  <h2>Welcome to HelloPhoenix!</h2>
</div>
```

> 注意： 如果你想在模板中使用 path 或 url helpers 的时候，我们需要传递 endpoint 而不是 conn 作为第一个参数。这
> 是因为我们并不是在一个请求中上下文中，所以 `@conn` 不可用，比如，我们需要这样写：

```elixir
alias HelloPhoenix
Router.Helpers.page_url(Endpoint, :index)
```

而不是这样：

```elixir
Router.Helpers.page_path(@conn, :index)
```

如果我们需要给模板传递参数，我们可以将其作为 `Phoenix.View.render_to_string/3` 函数的第三个参数。

```elixir
. . .
defp welcome_html do
  Phoenix.View.render_to_string(HelloPhoenix.EmailView, "welcome.html", %{})
end
. . .
```

现在我们的 `send_welcome_email/1` 函数看起来好多了。

```elixir
def send_welcome_email(email_address) do
  send_email to: email_address,
             from: @from,
             subject: "Welcome!",
             text: welcome_text,
             html: welcome_html
end
```

### 发送附件

Mailgun 同时允许我们发送附件。我们可以使用 `:attachments` 告诉 `mailgun` 我们需要发送一个或多个附件。值是一个
包含两个元素的map 的列表。一个元素是附件文件的路径，另一个是文件名。

下面的例子是给欢迎邮件加一个 Phoenix 框架的 logo 附件:

```elixir
def send_welcome_email(email_address) do
  send_email to: email_address,
             from: @from,
             subject: "Welcome!",
             text: welcome_text,
             html: welcome_html,
             attachments: [%{path: "priv/static/images/phoenix.png", filename: "phoenix.png"}]
end
```

现在改为测试模式，重启应用，然后调用 `send_welcome_email/1` 函数， 我们可以在响应的结尾部分看到附件信息。

```console
more mailgun.json
{"to":"us@example.com","text":"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.\n","subject":"Welcome!","html":"<div class=\"jumbotron\">\n  <h2>Welcome to HelloPhoenix!</h2>\n</div>","from":"Mailgun Sandbox <postmaster@sandbox-our-domain.mailgun.org>","attachments":[{"path":"priv/static/images/phoenix.png","filename":"phoenix.png"}]}
```

现在我们可以放心的在非测试环境使用了。
