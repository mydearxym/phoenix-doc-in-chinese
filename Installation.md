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

