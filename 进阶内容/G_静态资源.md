# 静态资源

Phoenix 没有内建的资源打包工具， 而是使用了 [Brunch](http://brunch.io), 速度很快并且易于使用。Phoenix 默认包含
了对 Brunch 的配置，它默认支持 js 以及 css ， 并且添加诸如  CoffeeScript, TypeScript 或 Less 之类的支持也非常
简单。

Brunch 有个 [很棒的教程](https://github.com/brunch/brunch-guide), 足够我们熟练的使用其管理我们的 Phoenix 资源。

#### 安装

Brunch 是一个 [Node.js](https://nodejs.org/) 的应用。Phoenix 默认生成 `package.json` 中的依赖也使用 node 的包
管理器 [npm](https://www.npmjs.com/) 来安装。 在运行 `mix phoenix.new` 生成项目时，Phoenix 会询问我们是否同时
安装 `package.json` 中的依赖，如果你想稍后安装或者改动了 `package.json` 文件，你可以手动的运行安装：

```
npm install
```

这一步会将 Brunch 和其插件安装到 `node_modules` 目录去。

#### 默认的配置和工作流

我们的主角是 `brunch-config.js` -- Brunch 自己的配置文件，让我们看看它是怎样管理 Phoenix 的资源的。

根据配置， Brunch 会将 `web/static` 目录下的文件视为源文件。`web/static/assets` 目录下的文件和目录会被原封不动
的拷贝到目标路径。

Brunch 会将除 `web/static/assets` 之外的 `web/static` 目录下的 `css` 和 `js` 文件分组，压缩处理合并后的 js 文件会
被放置在 `priv/static/js/app.js`, 样式文件则会被放置在 `priv/static/css/app.css` 。

当 Phoenix 在运行时，Brunch 会监视并自动处理资源文件的改动，你也可以手动执行：

```
node node_modules/brunch/bin/brunch build
```
或者我们可以全局安装 Brunch:

```
npm install -g brunch
```

然后像这样编译资源。

```
brunch build
```

除了 `web/static` 目录下的 js 文件，下列文件也会被加入到最终的  `priv/static/js/app.js` 。

- Brunch 自己的初始代码 ("bootstrapper" code ), 包含了模块管理以及 `require()` 逻辑。
- Phoenix Channels 的 js 客户端文件 (`deps/phoenix/web/static/js/phoenix.js`)。
- 一些在 Phoenix.HTML (`deps/phoenix_html/web/static/js/phoenix_html.js`) 中的代码。


#### 模块

默认情况下每一个 js 文件都会被包含在一个模块中， 想要执行这个代码需要在其他模块中引入并执行 (for this code to be executed it needs
to be required and executed from another module.)。 Brunch 使用不包含文件后缀名的文件路径作为模块的名字，让我
们看看它是如何工作的：

在 Phoenix 中 Brunch 默认使用 ES6 语法。
打开  `web/static/js/app.js` 添加如下代码：

```javascript
export var App = {
  run: function(){
    console.log("Hello!")
  }
}
```

如果 ES6 语法对你有些陌生，下面的等价的 js 代码。

```javascript
var App = {
  run: function run() {
    console.log("Hello!");
  }
};

module.exports = {
  App: App
};
```

打开默认的应用布局文件 (application layout) `web/templates/layout/app.html.eex`, 找到：

```html
<script src="<%= static_path(@conn, "/js/app.js") %>"></script>
```

在下面加上：

```html
<script>require("web/static/js/app").App.run()</script>
```

当我们刷新页面的时候就可以在浏览器控制台上看到 'hello!'。

注意那个 `"web/static/js/app"`。这并不是一个文件名，这是被包含在 `"web/static/js/app.js"` 中的模块的名字。

让我们再来添加一个文件： `web/static/js/greeter.js`:

```javascript
export var Greet = {
  greet: function(){
    console.log("Hello!")
  }
}

export var Bye = {
  greet: function(){
    console.log("Bye!")
  }
}
```

然后改动 `web/static/js/app.js` 引入新模块。

```javascript
import { Greet } from "web/static/js/greeter";

export var App = {
  run: function(){
    Greet.greet()
  }
}
```

然后刷新页面，可以在控制台上看到 `hello!`。

注意 `Bye` 并没有被引入到 `"web/static/js/app"` 。

注意我们在 HTML 页面和 js 文件中引入模块的方式是不同的，因为 HTML 文件不会经过预处理，所以 ES6 的语法是不起作
用的。

#### 老旧的非模块化代码 (Legacy Non-modularized Code)

如果我们有一些老旧的 js 代码和现有的模块系统不太兼容或者我们需要使用其中定义的全局变量，我们可以将它们放到
`web/static/vendor` 目录下。 Brunch 就不会将这些代码包装成模块了。

我们来测试一下。创建目录 `web/static/vendor` , 并创建文件 `web/static/vendor/meaning_of_life.js` 添加一行代码：

```
meaning_of_life = 42;
```

刷新页面，打开控制台并输入 `meaning_of_life`, 会返回 `42`, 这个变量是全局的。

注意： 默认的配置下 `web/static/vendor` 目录下的 js 文件是不支持 ES6 语法的，如果你需要，可以看看
`brunch-config.js` 文件中 `plugins: { babel: { ignore:` 部分。

#### 外部 JavaScript 库

我们的项目可能会使用外部的第三方库比如 jQuery 或者 underscore 等等，像上面提到的类似，我们可以将其放置在
`web/static/vendor` 目录下。但是使用 `npm` 去安装他们是更好的选择，以 jQuery 为例，只需要在 package.json 中加
入 `"jquery": ">= 2.1"` 然后使用 `npm install --save` 安装即可。 如果我们在 `brunch-config.js` 的 `npm` 部分
有 `whitelist`属性，我们也需要将 `jQuery` 添加到那里。 现在我们就可以在 `app.js` 中使用 `import $ from "jquery"` 来使用了。

如果我们现有的代码中是将 jQuery 作为全局变量使用的，我们需要重构一下代码（从长远维护角度），或者就将 jQuery 作
为全局变量来使用（算是一个 hack）。

将其暴露为全局变量，我们需要在配置中使用 `globals` 字段。比如，如果我们想要 jQuery 作为 $ 在全局使用，我们需要
配置如下：

```javascript
  npm: {globals: {
    $: 'jquery',
    jQuery: 'jquery'
  }},
```
更进一步的，有些外部的库会自带样式，为了让 Brunch 将这些样式加入到编译流程，我们同样需要配置。比如，如果我们安
装了 Picaday 包并且想引入它的样式，我们可以配置如下：

```javascript
  npm: {styles: {
    bootstrap: ['dist/css/bootstrap.min.css']
  }},
```
#### Brunch Plugin Pipeline

Brunch 中所有的转换功能都是通过插件机制实现的， Brunch 会使用 `package.json` 中安装的工具来实现其功能，主要部
分如下：

##### Javascript

- [`babel-brunch`](https://github.com/babel/babel-brunch) 使用 [Babel](https://github.com/babel/babel) 转换
  ES6 语法。
- [`javascript-brunch`](https://github.com/brunch/javascript-brunch) 主要的 js 处理插件，如果没有他，js 文件将
  被忽略。
- [`uglify-js-brunch`](https://github.com/brunch/uglify-js-brunch) 压缩 Javascript 代码。

##### CSS

- [`clean-css-brunch`](https://github.com/brunch/clean-css-brunch)  CSS 的一个压缩工具。
- [`css-brunch`](https://github.com/brunch/css-brunch) 主要的 CSS 处理插件, 如果没有。is the main CSS processing plugin. Without it  CSS files will be ignored.

添加插件很容易，我们可以在 [Brunch 官网](http://brunch.io/plugins.html) 和
[npm 的 brunch 标签](https://www.npmjs.com/search?q=brunch) 下找到他们。

插件执行的顺序就是在 `package.json` 中出现的顺序。

比如添加一个 CoffeeScript 插件，只需在 `package.json` 的 `javascript-brunch` **之前** 加入下面语句即可:

```json
  "coffee-script-brunch": ">= 1.8",
```

然后安装：

```
npm install
```

然后我们将 `greeter.js` 改成 `greeter.coffee` 并改动其中的内容：

```coffeescript
Greet =
  greet: -> console.log("Hello!")

Bye =
  greet: -> console.log("Bye!")

module.exports =
  Greet: Greet
  Bye: Bye
```

一旦 Brunch 完成编译，刷新页面，我们可以看到结果和之前一致。

添加其他的预处理器和这个一样简单，一些插件的配置就在 `brunch-config.js` 中，但是得等插件安装后才能起效。

#### Other Things Possible With Brunch

使用 Brunch 还能做许多有趣的事情，这里只列出一小部分：

- 可以使用
  [多目标编译](https://github.com/brunch/brunch-guide/blob/master/content/en/chapter04-starting-from-scratch.md#split-targets)
  将资源编译成多个文件，比如 `app.js` 为我们的应用逻辑，`vendor.js` 为我们的第三方依赖。
- 可以控制编译文件的生成顺序，尤其对于 `vendor` 目录下的第三方库（它们可能有相互依赖），这个特性是很重要的。
- 除了直接拷贝第三方文件到`web/static/vender`, 我们还可以使用[Bower 来安装他们](https://github.com/brunch/brunch-guide/blob/master/content/en/chapter05-using-third-party-registries.md).

更多的细节请查看 [Brunch 文档](http://brunch.io/docs/getting-started).

#### 不使用 Brunch

如果你不想使用 Brunch， 可以在创建 Phoenix 应用时使用 `--no-brunch` 选项。

```
mix phoenix.new --no-brunch my_project
```

####  使用其他的打包编译工具

在 Phoenix 中使用其他的打包工具，我们需要：

- 配置该工具使其将编译后的资源放置到 `priv/static/`.
- 让 Phoenix 知道怎样根据每次文件的变动来编译目标文件。

第一条很简单，但怎样让 Phoenix 知道编译资源的命令呢？ 很简单，打开 `config/dev.exs` 我们会发现 `:watchers` 字
段。 `:watchers` 是一个包含可执行程序及其参数的元祖，比如：

```elixir
  watchers: [node: ["node_modules/brunch/bin/brunch", "watch"]]
```

在开发模式下会展开成：

```
node node_modules/brunch/bin/brunch  watch
```

再让我们看看如何在 Phoenix 中集成 [Webpack](http://webpack.github.io), 首先，我们使用 `--no-brunch` 新建工程。

```
mix phoenix.new --no-brunch my_project
```

或者手动移除 brunch:

```
rm brunch-config.js && rm -rf node_modules/*
```

修改 `package.json`:

```json
{
  "devDependencies": {
    "webpack": "^1.11.0"
  }
}
```

安装 webpack

```
npm install
```

创建 webpack 的配置文件 `webpack.config.js` ：

```javascript
module.exports = {
  entry: "./web/static/js/app.js",
  output: {
    path: "./priv/static/js",
    filename: "app.js"
  }
};
```

然后让 Phoenix 知道怎样运行 webpack 。将 `:watchers` 中的内容替换成如下内容：

```elixir
watchers: [node: ["node_modules/webpack/bin/webpack.js", "--watch", "--color"]]
```

当我们重启 Phoenix, webpack 会监视文件改动并编译它们。

注意这里 webpack 的配置都是最基本的, 只是演示在 Phoenix 中怎样集成不同的打包编译工具， 更多的细节请查看
[Webpack 文档](http://webpack.github.io/docs/)。
