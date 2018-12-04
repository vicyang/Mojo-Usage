Mojolicious::Guides::Tutorial - Get started with Mojolicious

* ### Tutorial
  [Mojolicious::Lite](https://metacpan.org/pod/Mojolicious::Lite) 简单教学示例。本章所涉及到的技巧同样适用于 [Mojolicious](https://metacpan.org/pod/Mojolicious) 应用

  这是 Mojolicious::Guides 中的入门部分，[growing]() 和 [Mojolicious::Lite]() 教学进一步指导如何构建完整的 Mojo 应用，建议读者在读完本章之后再阅读 [routing]()、 [rendering]() 等剩下的章节。

* ####　Hello World
  Mojo的 Hello World 应用非常简单： strict, warnings, utf8 以及 Perl 5.10 特性自动开启，当你看到  Mojolicious::Lite 的时候，就已经开启了完整的 Web Application 特性。
  
  ```perl
  #!/usr/bin/env perl
  use Mojolicious::Lite;
   
  get '/' => sub {
    my $c = shift;
    $c->render(text => 'Hello World!');
  };
   
  app->start;
  ```
  
  同时 Mojo 还提供 Mojolicious::Command::Author::generate::lite_app 命令用来帮助生成小型应用模板（实例）：

  `$ mojo generate lite_app myapp.pl`

* ### 指令
  许多不同的[指令(commands)](https://metacpan.org/pod/Mojolicious::Commands#COMMANDS)可以通过终端调用。CGI和PSGI环境下甚至可以自动检测，不需要提供指令即可运转。

  ```sh
  $ ./myapp.pl daemon
  Server available at http://127.0.0.1:3000
   
  $ ./myapp.pl daemon -l http://*:8080
  Server available at http://127.0.0.1:8080
   
  $ ./myapp.pl cgi
  ...CGI 内容输出...
   
  $ ./myapp.pl get /
  Hello World!
   
  $ ./myapp.pl
  ... 列出有效命令/参数 (或者自动检测环境执行对应命令) ...
  ```
  
  Mojolicious 的 start 方法（app->start）用于开启命令系统，该语句应放在Mojo应用的末尾，因为其返回值对Mojo应用来说非常重要（在Perl中，最后一条表达式的值作为返回值）
  （注：以下第二种方式，通过start参数传递指令，自动执行，而非通过 @ARGV）

  ```perl
  # 通过 @ARGV 获得命令参数
  app->start;

  # 开启 daemon 命令
  app->start('daemon', '-l', 'http://*:8080');
  ```
  
* ### Reloading
  如果使用 morbo web server 启动app，你的应用将能够自动重新加载，修改后也无需重启服务（动态更新）。

  ```
  $ morbo ./myapp.pl
  Server available at http://127.0.0.1:3000
  ```
  
  更多关于应用部署的资料，参考 ["DEPLOYMENT" in Mojolicious::Guides::Cookbook](https://metacpan.org/pod/distribution/Mojolicious/lib/Mojolicious/Guides/Cookbook.pod#DEPLOYMENT)

* ### Routes
  路由主要用来简化地址的路径处理以及引导执行对应的行为（当URL路径配对时）。第一个参数传递到 [Mojolicious::Controller](https://metacpan.org/pod/Mojolicious::Controller) 对象：$c，其中包含了HTTP请求和响应信息。

  ```perl
  use Mojolicious::Lite;
  # Route leading to an action that renders some text
  get '/foo' => sub {
    my $c = shift;
    $c->render(text => 'Hello World!');
  };
  app->start;
  ```

  响应表单（Response content）通常通过Mojolicious::Controller的 [render](https://metacpan.org/pod/Mojolicious::Controller#render) 动作生成，这个稍后会谈到。

* ### GET/POST 参数
  所有 GET 和 POST 的请求参数通过 Mojolicious::Controller 的 [param](https://metacpan.org/pod/Mojolicious::Controller#param) 方法传递
  
  ```perl
  use Mojolicious::Lite;
  # /foo?user=sri
  get '/foo' => sub {
    my $c    = shift;
    my $user = $c->param('user');
    $c->render(text => "Hello $user.");
  };
  app->start;
  ```
  
  Stash 和模板
  Mojolicious::Controller 的 [stash](https://metacpan.org/pod/Mojolicious::Controller#stash) 方法用来传递数据到模板，它可以插入到 `__DATA__` 中。一些stash值作为模板，保存文本和数据，通过 render 方法决定如何生成响应表单。

  ```perl
  use Mojolicious::Lite;
 
  # Route leading to an action that renders a template
  get '/foo' => sub {
    my $c = shift;
    $c->stash(one => 23);
    $c->render(template => 'magic', two => 24);
  };
   
  app->start;
  __DATA__
   
  @@ magic.html.ep
  The magic numbers are <%= $one %> and <%= $two %>.
  ```

  更多关于模板的信息，请参考 ["Embedded Perl" in Mojolicious::Guides::Rendering](https://metacpan.org/pod/distribution/Mojolicious/lib/Mojolicious/Guides/Rendering.pod#Embedded-Perl)

* ### HTTP
  Mojolicious::Controller 的 [req](https://metacpan.org/pod/Mojolicious::Controller#req) 和 [res](https://metacpan.org/pod/Mojolicious::Controller#res) 方法用来处理所有HTTP特性和信息。

  ```perl
  use Mojolicious::Lite;
   
  # Access request information
  get '/agent' => sub {
    my $c    = shift;
    my $host = $c->req->url->to_abs->host;
    my $ua   = $c->req->headers->user_agent;
    $c->render(text => "Request by $ua reached $host.");
  };
   
  # Echo the request body and send custom header with response
  post '/echo' => sub {
    my $c = shift;
    $c->res->headers->header('X-Bender' => 'Bite my shiny metal ass!');
    $c->render(data => $c->req->body);
  };
   
  app->start;
  ```

  你可以通过 [Mojolicious::Command::get](https://metacpan.org/pod/Mojolicious::Command::get) 命令做更高级的测试
  
  `$ ./myapp.pl get -v -M POST -c 'test' /echo`

  JSON
  JSON 是互联网最流行的数据转储格式（之一）。Mojo提供了纯Perl实现的内置模块：[Mojo::JSON](https://metacpan.org/pod/Mojo::JSON)，以及通过 Mojo::Message 的[json](https://metacpan.org/pod/Mojo::Message#json)方法处理json数据。

  ```perl
  use Mojolicious::Lite;
   
  # 修改接收到的 JSON 数据并返回
  put '/reverse' => sub {
    my $c    = shift;
    my $hash = $c->req->json;
    $hash->{message} = reverse $hash->{message};
    $c->render(json => $hash);
  };
   
  app->start;
  ```

  你可以通过执行 [Mojolicious::Command::get](https://metacpan.org/pod/Mojolicious::Command::get) 命令，向app传递 JSON 数据。

  `$ ./myapp.pl get -M PUT -c '{"message":"Hello Mojo!"}' /reverse`
  
* ### 内置异常 和 not_found 页面
  在开发时期只要你不小心出错就会遇到这些页面，其中包含很多有效信息来帮助你调试应用。
  
  ```perl
  use Mojolicious::Lite;
  # Not found (404)
  get '/missing' => sub { shift->render(template => 'does_not_exist') };
  # Exception (500)
  get '/dies' => sub { die 'Intentional error' };
  app->start;
  ```

  甚至可以通过向 [Mojolicious::Command::get](https://metacpan.org/pod/Mojolicious::Command::get) 命令传递 CSS selector 来选择感兴趣的信息。

  `$ ./myapp.pl get /dies '#error'`
  （实测如果去掉 '#error' 则会产生大量页面信息）
  
  不过不必太担心这些包含大量信息的页面，它们只在开发过程中出现，以在生产环境中将被自动替换为不含任何敏感信息的页面

* ### 路径名称
  所有路径

All routes can have a name associated with them, this allows automatic template detection and backreferencing with "url_for" in Mojolicious::Controller, on which many methods and helpers like "link_to" in Mojolicious::Plugin::TagHelpers rely.

use Mojolicious::Lite;
 
# Render the template "index.html.ep"
get '/' => sub {
  my $c = shift;
  $c->render;
} => 'index';
 
# Render the template "hello.html.ep"
get '/hello';
 
app->start;
__DATA__
 
@@ index.html.ep
<%= link_to Hello  => 'hello' %>.
<%= link_to Reload => 'index' %>.
 
@@ hello.html.ep
Hello World!

Nameless routes get an automatically generated one assigned that is simply equal to the route itself without non-word characters.
  


