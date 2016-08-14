# 配合代理服务器使用 phoenix

为了在代理服务器比如 `nginx` 或者 `apache` 后面使用 Phoenix, 我们需要配置一个特殊的监听端口。

目前有两种方法。第一，如果你确定以后不更改，你可以在 `config/prod.exs` 配置中写死 `http: [port: 8080]`。

```elixir
use Mix.Config

. . .

config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: 8080],
  cache_static_manifest: "priv/static/manifest.json"

. . .
```

第二种，如果你想要端口的配置更加的灵活，我们可以将 port 的值从外面的环境变量传递进来。`config/prod.exs` 中的配
置如下：

```elixir
use Mix.Config

. . .

config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: {:system, "PORT"}],
  cache_static_manifest: "priv/static/manifest.json"

. . .
```

另外，从 `HelloPhoenix.Router.Helpers` 模块中使用 `_url` 函数生成的地址，比如使用 `user_url(conn, :index)` 生
成的 http://localhost:8080/users 会包含错误的端口，我们可以通过配置中的 `url` 字段来纠正这种行为。

```elixir
use Mix.Config

. . .

config :hello_phoenix, HelloPhoenix.Endpoint,
  http: [port: 8080],
  url: [host: "example.com", port: 80],
  cache_static_manifest: "priv/static/manifest.json"

. . .
```

这时当我们使用 `user_url(conn, :index)` 函数生成的地址会变成 http://example.com/users ， 注意这里生成的地址不
包含端口信息。 因为 `http` 协议默认端口是 80, `https` 默认是 `443`, 这两种情况下不必显示的将端口加在 url 之中，
而其他情况下，端口信息则会出现在最后生成的 url 之中。

### Nginx 配置

Nginx 需要针对基于 HTTP 请求的 channels 、Websockets  做一些额外的配置，来把普通的无状态的 HTTP `升级` 成持久
的 websocket 连接。

庆幸的是，这在 nginx 中是比较容易实现的。

下面是一个给定域名 `my-app.domain` 的 nginx 的典型配置。

```
// /etc/nginx/sites-enabled/my-app.domain
upstream phoenix {
  server 127.0.0.1:4000 max_fails=5 fail_timeout=60s;
}

server {
  server_name my-app.domain;
  listen 80;

  location / {
    allow all;

    # Proxy Headers
    proxy_http_version 1.1;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-Cluster-Client-Ip $remote_addr;

    # The Important Websocket Bits!
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";

    proxy_pass http://phoenix;
  }
}

```
这个配置包含两个部分 -- 代理端，即 `upstream`, 以及 `server` -- 监听特定的域名和端口。

`server` 是主要起作用的部分：它确保我们传递给 Phoenix 正确的 Headers （`Upgrade` 以及 `Connection` 头）以确保
channels 正常工作。

这些头设置并不能立即起效，你还需要在 js 客户端那边做一下工作, 这些 Headers 只是简单的确保 browser 到 Phoenix
的线路畅通无阻。
