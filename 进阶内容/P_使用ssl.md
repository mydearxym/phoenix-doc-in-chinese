# 使用 SSL


为了使我们的应用支持 SSL 请求，我们需要一些配置并设置两个环境变量。我们让我们的 SSL 正常工作，我们需要一个 key
文件和从认证机构获得的证书文件。 环境变量就是这两个文件的路径信息。

配置中的新字段 `https:` 是一个包含端口，key 文件路径，认证文件路径的关键字列表。如果我们添加了 `opt_app:` 键，
它的值则是我们应用的名字， Plug 会从我们的项目根目录开始寻找它们，我们可以将上述文件放在 `priv` 目录并把路径指
向 `priv/our_keyfile.key` 和 `priv/our_cert.crt`。

这里是 `config/prod.exs` 中的一个例子：

```elixir
use Mix.Config

. . .
config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: {:system, "PORT"}],
  url: [host: "example.com"],
  cache_static_manifest: "priv/static/manifest.json",
  https: [port: 443,
          otp_app: :hello_phoenix,
          keyfile: System.get_env("SOME_APP_SSL_KEY_PATH"),
          certfile: System.get_env("SOME_APP_SSL_CERT_PATH"),
          cacertfile: System.get_env("INTERMEDIATE_CERTFILE_PATH") # OPTIONAL Key for intermediate certificates
          ]

```

如果没有 `opt_app:` 键，我们需要提供上述文件的绝对路径使 Plug 能找到他们。

```elixir
Path.expand("../../../some/path/to/ssl/key.pem", __DIR__)
```

另外还需确保我们的 `mix.exs` 中包含 ssl 应用:

```elixir
def application do
    [mod: {HelloPhoenix, []},
    applications: [:phoenix, :phoenix_html, :cowboy, :logger, :gettext,
                 :phoenix_ecto, :postgrex, :ssl]]
end
```

否则你可能会得到如下错误： `** (MatchError) no match of right hand side value: {:error, {:ssl, {'no such file or directory', 'ssl.app'}}}`
