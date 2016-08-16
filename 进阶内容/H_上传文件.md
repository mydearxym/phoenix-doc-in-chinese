
# 文件上传

Web 应用中一个常见的需求是文件上传。 文件可以是图像、视频、PDF或者其他类型的文件， 为了在 HTML 页面上上传，我
们需要在表单 (multipart form) 的输入标签上包含 `file` 选项。

Plug 内建了一个 `Plug.Upload` 结构来处理 `file` 输入框中的数据。 当用户通过表单提交一个文件时， `Plug.Upload`
结构体会自动出现在请求参数中。

让我们来仔细看看。在 [`Ecto 模型指南`](https://mydearxym.gitbooks.io/phoenix-doc-in-chinese/content/H_ecto%E6%A8%A1%E5%9E%8B.html)
中，我们生成了一个 users `资源`。我们可以复用这里的表单来演示怎样在 Phoenix 中上传文件。

首先是将普通的表单改为多功能的表单 (multipart form)。 `form_for/4` 函数接收选项的关键字列表， 我们可以在其中指
定。

这里是更改之后的 `web/templates/user/form.html.eex` 文件。

```elixir
<%= form_for @changeset, @action, [multipart: true], fn f -> %>
. . .
```

当我们有了多功能表单 (multipart form), 我们还需要一个 `file` 输入框。 我们改变一下 `form.html.eex`。

```html
. . .
  <div class="form-group">
    <label>Photo</label>
    <%= file_input f, :photo, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
```

这是渲染成 HTML 后的样子：

```html
<div class="form-group">
  <label>Photo</label>
  <input class="form-control" id="user_photo" name="user[photo]" type="file">
</div>
```

注意 `file` 输入框中的 `name` 属性。这会在 `user_params` 请求参数中生成一个 `"photo"` 键，我们可以在控制器中使
用。

基本差不多了，现在当用户提交这个表单，一个 `POST` 请求会被路由给 `HelloPhoenix.UserController` `create/2` action.

> 注意：photo 并不非得是我们模型的一部分，如果我们想将 photo 持久化到数据库中，我们需要将其添加到
> `HelloPhoenix.User` 模型的 schema 中。

首先，现在 `HelloPhoenix.create/2` action 的最上面加上一行 `IO.inspect user_params`。这会在开发日志中打印
`user_params` 的详细内容。

```elixir
. . .
  def create(conn, %{"user" => user_params}) do
    IO.inspect user_params
. . .
```

现在我们运行服务器 `mix phoenix.server`, 在浏览器中访问
[http://localhost:4000/users/new](http://localhost:4000/users/new), 并用 photo 生成一个新用户。

Since we generated an HTML resource, we can now start our server with `mix phoenix.server`, visit
[http://localhost:4000/users/new](http://localhost:4000/users/new), and create a new user with a photo.

日志中会出现 `user_params` 的详细内容：

```elixir
%{"bio" => "Guitarist", "email" => "dweezil@example.com", "name" => "Dweezil Zappa", "number_of_pets" => "3",
"photo" => %Plug.Upload{content_type: "image/jpg", filename: "cute-kitty.jpg", path: "/var/folders/_6/xbsnn7tx6g9dblyx149nrvbw0000gn/T//plug-1434/multipart-558399-917557-1"}}
```
我们有了一个 "photo" 键 --- 其值被映射到了 `Plug.Upload` 结构体，来代表上传的文件信息。

为了方便，我们只关心这个结构体。

```elixir
%Plug.Upload{content_type: "image/jpg", filename: "cute-kitty.jpg", path: "/var/folders/_6/xbsnn7tx6g9dblyx149nrvbw0000gn/T//plug-1434/multipart-558399-917557-1"}
```

`Plug.Upload` 里的键有文件的 content 类型，可选的文件名，已经 Plug 为我们创建的零时文件。 在我们的例子中,`"/var/folders/_6/xbsnn7tx6g9dblyx149nrvbw0000gn/T//plug-1434/"`
就是Plug 为我们上传文件创建的零时文件。 它在请求的整个周期内都会存在。`"multipart-558399-917557-1"` 是 Plug 给
起的文件名。如果我们在多功能表单中上传了多个文件， Plug 会确保文件名唯一。

> 注意： 这个文件只是零时的，当请求完成后， Plug 会将其删除。 如果我们对文件做一些操作，我们必须赶在之前。

当我们能在控制器中放问到 `Plug.Upload` 后，我们就可以做任何操作了。比如我们可以使用 `File.exists?/1` 确定文件
是否存在；使用 `File.cp/2` 将文件拷贝到其他地方；使用第三方工具将文件发送到 S3；或者甚至使用 [Plug.Conn.send_file/5](http://hexdocs.pm/plug/Plug.Conn.html#send_file/5)
将文件发回到前端。

当我们没有上传文件时，请求参数里将不会有 "photo" 或者 "Plug.Upload" 键值：

```elixir
%{"bio" => "Guitarist", "email" => "dweezil@example.com", "name" => "Dweezil Zappa", "number_of_pets" => "3"}
```

## 配置上传限制

在文件上传过程中表单到`Plug.Upload`的数据转换实际上是通过 `Plug.Parsers` 实现的。我们可以在
`HelloPhoenix.Endpoint` 找到：

```elixir
plug Plug.Parsers,
  parsers: [:urlencoded, :multipart, :json],
  pass: ["*/*"],
  json_decoder: Poison
```

除了上面的选项, `Plug.Parsers` 还接受其他选项来控制上传行为：

  * `:length` - 限制文件最大字节数 默认是 `8_000_000` 字节。
  * `:read_length` - 设置每次到达的文件块大小，默认 `1_000_000` 字节。
  * `:read_timeout` - 设置每次文件块到达的超时时间默认`15_000` ms。

第一个选项顶一个了最大的上传值。其他的两个选项则配置了文件上传块的大小和频率 (The remaining ones configure how much data we expect to
read and its frequency.)。如果客户端上传文件太慢，则连接会被终止。Phoenix 设定的这些默认值是经过考虑的，当然你
也可以在个别情况下定制它，比如，如果我们在从慢速的客户端接收一个很大的文件。

需要指出的是，这里的设置还有安全方面的考虑。比如，如果我们不设置文件块限制的，攻击者可能会启动成千上万个连接，并在
每个连接中每两分钟上传一个字节，这会过多的占用服务器的资源。Phoenix 的这些默认设置让攻击者的门槛变高了。
