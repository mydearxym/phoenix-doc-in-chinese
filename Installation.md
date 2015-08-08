#### Intallation

在之前的 Overview 章节中我们已经大致了解了 Phoenix 的生态系统和各组件之间的相互关联。现在，在开始正式学习之前，让我们来安装一些必要的软件。

请确保下面列出的软件包已经安装在您的系统中，以避免由此导致的令人沮丧的问题。

##### Elixir

`Phoenix`和我们即将编写的应用都是基于 Elixir语言。如果你需要帮助，Elixir 的官网上有专门的[安装引导](http://elixir-lang.org/install.html)。

如果我们是第一次安装`Elixir`,我们还需要同时安装 `Hex 包管理工具` ,她对于 `Phoenix`的运行是至关重要的（自身和第三方的依赖安装以及管理）。

安装 `Hex`: 
```Bash
$ mix local.hex
```

##### Erlang

Elixir 代码会编译成 Erlang 的字节码，如果没有 Erlang，Elixir将无法运行在Erlang虚拟机上。

[Elixir的安装指导](http://elixir-lang.org/install.html)里有明确的步骤，这里不再赘述。

##### Phoenix 

Once we have Elixir and Erlang, we are ready to install the Phoenix mix archive. A mix archive is a zip file which contains an application as well as its compiled beam files. It is tied to a specific version of the application. The archive is what we will use to generate a new, base Phoenix application which we can build from.

一旦我们安装玩 Elixir 和 Erlang 后，我们就可以安装`Phoenix mix`集合了(archive)，mix集合是一个包含了应用程序（以及beam files）的zip格式的文件，她绑定特定版本的应用，可以让我们建立一个新的基于 `Phoenix`的应用。

安装 `Phoenix archive`: 
```Bash
$ mix archive.install https://github.com/phoenixframework/phoenix/releases/download/v0.16.1/phoenix_new-0.16.1.ez

```
> 注意如果这个自动安装脚本出错，你可以在浏览器中下载文件保存到本地。
> 然后运行：
> ```Bash
> $ mix archive.install /path/to/local/phoenix_new.ez
> ```


##### Plug, Cowboy, and Ecto
这些是 Phonenix 自带的组件，我们并不需要显示的去安装他们，如果我们使用`mix`安装第三方依赖以及生成项目的话，`mix`会自动帮我们安装好，否则，`Phoenix`会给予提示。

##### node.js  (>= 0.12.0)
Node is an optional dependency. Phoenix will use brunch.io to compile static assets (javascript, css, etc), by default. Brunch.io uses the node package manager (npm) to install its dependencies, and npm requires node.js.

If we don't have any static assets, or we want to use another build tool, we can pass the --no-brunch flag when creating a new application and node won't be required at all.

node是一个可选的依赖包。 `Phoenix`使用 [brunch.io](http://brunch.io/) 打包静态文件（javascript,css,等等），而 brunch 工具依赖于 node.js.

如果我们的应用没有静态资源或者我们想用其他的构建打包工具，我们可以在创建项目的时候传递`--no-brunch`选项，在这种情况下，不需要安装node.js。

这里译者省去了`node.js`的安装部分。。。

##### PostgreSQL

`Phonenix` 默认使用`PostgreSQL`关系型数据库, 但我们也可以在创建项目时使用 `--database mysal` 选项转而使用MySQL。

在接下来的章节中使用 Ecto models 时，我们会默认使用 `PostgerSQL` 和 `Postgrex 适配器`,为了跟上这些例子，我们应该安装 `PostgerSQL`, 安装指南[参见这里](https://wiki.postgresql.org/wiki/Detailed_installation_guides)

`Postgrex` 是 `Phoenix`的默认依赖，会在我们启动项目的时候自动安装。

##### inotify-tools (for linux users)

这个工具是 `Phoenix` 监测文件系统用于实现实时代码刷新的(`live code reloading`)。(Mac 和 Win用户不需要理会)。

Linux用户的[安装指南](https://github.com/rvoicilas/inotify-tools/wiki)

