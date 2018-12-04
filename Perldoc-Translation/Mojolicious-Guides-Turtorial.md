Mojolicious::Guides::Tutorial - Get started with Mojolicious

* ### Tutorial
  [Mojolicious::Lite](https://metacpan.org/pod/Mojolicious::Lite) 教学示例。本章所涉及到的技巧同样适用于 [Mojolicious](https://metacpan.org/pod/Mojolicious) 应用

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
  （注：以下第二种方式，通过start参数传递指令，自动执行，而非通过命令行指令传参）

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

* ### 路由（Routes）
  路由主要用来简化处理不同的路径占位符，以及引导执行对应的动作（当URL路径配对时）。第一个传递到$c的参数是 [Mojolicious::Controller](https://metacpan.org/pod/Mojolicious::Controller)  对象，其中包含了HTTP请求和响应信息。

  ```perl
  use Mojolicious::Lite;
  # 通过路由引导 -> 执行文本渲染操作
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
  
  （ 测试方法：`app->start("get", "/foo?user=test");` 或者 `myapp.pl get /foo?user=terminal` ）

  Stash 和 Template 模板
  Mojolicious::Controller 的 [stash](https://metacpan.org/pod/Mojolicious::Controller#stash) 方法用来传递数据到模板，可用来将数据插入到 `__DATA__` 节点。stash的行为类似render方法中的模板参数，保存数据，然后通过render生成响应表单

  ```perl
  use Mojolicious::Lite;
  # 通过路由引导至模板渲染操作
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
  [Mojolicious::Controller]() 的 [req](https://metacpan.org/pod/Mojolicious::Controller#req) 和 [res](https://metacpan.org/pod/Mojolicious::Controller#res) 方法用来处理HTTP特性和信息。

  ```perl
  use Mojolicious::Lite;
   
  # 处理请求信息
  get '/agent' => sub {
    my $c    = shift;
    my $host = $c->req->url->to_abs->host;
    my $ua   = $c->req->headers->user_agent;
    $c->render(text => "Request by $ua reached $host.");
  };
   
  # 显示请求主体，发送客户端 header 作为响应。
  post '/echo' => sub {
    my $c = shift;
    $c->res->headers->header('X-Bender' => 'Bite my shiny metal ass!');
    $c->render(data => $c->req->body);
  };
   
  app->start;
  ```

  你可以通过 [Mojolicious::Command::get](https://metacpan.org/pod/Mojolicious::Command::get) 命令做更高级的测试
  
  `$ ./myapp.pl get -v -M POST -c 'test' /echo`

  * 输出：
    ```shell
    >myapp.pl get -v -M POST -c 'test' /echo
    [2018-12-04 10:33:03.99260] [6120] [debug] POST "/echo" (1f49dcd3)
    [2018-12-04 10:33:03.99339] [6120] [debug] Routing to a callback
    [2018-12-04 10:33:03.99380] [6120] [debug] 200 OK (0.001197s, 835.422/s)
    POST /echo HTTP/1.1
    Host: 127.0.0.1:51781
    Content-Length: 6
    Accept-Encoding: gzip
    User-Agent: Mojolicious (Perl)

    HTTP/1.1 200 OK
    Server: Mojolicious (Perl)
    X-Bender: Bite my shiny metal ass!
    Content-Type: text/html;charset=UTF-8
    Date: Tue, 04 Dec 2018 02:33:03 GMT
    Content-Length: 6

    'test'
    ```
  * 参数说明
    -v, --verbose              Print request and response headers to STDERR
    -M, --method <method>      HTTP method to use, defaults to "GET"
    -c, --content <content>    Content to send with request

  换成 cURL 就是
  `curl 127.0.0.1:3000/echo -d test -v`

  JSON
  JSON 是互联网最流行的数据交换格式（之一）。Mojo提供了纯Perl实现的内置模块支持：[Mojo::JSON](https://metacpan.org/pod/Mojo::JSON)，以及通过 Mojo::Message 的[json](https://metacpan.org/pod/Mojo::Message#json)方法处理json数据。

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
  注意在windows中应该改为
  `myapp.pl get -M PUT -c "{\"message\":\"Hello Mojo!\"}" /reverse`
  JSON中的双引号不能改为单引号。

  * LWP::UserAgent 测试
    ```perl
    use LWP::UserAgent;
    my $ua = LWP::UserAgent->new();
    my $res = $ua->put( "http://127.0.0.1:3000/reverse",
                        'Content' => "{\"message\":\"abcdef\"}"
        );
    print $res->content();
    ```
  
* ### 异常处理 和 page not_found
  在开发时期只要你不小心出错就会遇到这些页面，其中包含很多有效信息来帮助你调试应用。
  
  ```perl
  use Mojolicious::Lite;
  # Not found (404)
  get '/missing' => sub { shift->render(template => 'does_not_exist') };
  # Exception (500)
  get '/dies' => sub { die 'Intentional error' };
  app->start;
  ```

  甚至可以通过向 [Mojolicious::Command::get](https://metacpan.org/pod/Mojolicious::Command::get) 命令传递 CSS selector 来筛选感兴趣的信息。

  `$ ./myapp.pl get /dies '#error'`
  （'#error' 用于筛选）
  
  不过不必太担心这些包含大量信息的页面，它们只在开发过程中出现，并且在生产环境中将自动替换为不含任何敏感信息的页面

* ### 路径名称
  任何路径都可以有一个名字与之关联，这样就能允许自动模板检测、Mojolicious::Controller的[url_for](https://metacpan.org/pod/Mojolicious::Controller#url_for)方法进行反向引用，Mojolicious::Plugin::TagHelpers的[link_to](https://metacpan.org/pod/Mojolicious::Plugin::TagHelpers#link_to)和许多其他方法也都依赖于此。

  ```perl
  use Mojolicious::Lite;
  # 渲染 "index.html.ep" 模板
  get '/' => sub {
    my $c = shift;
    $c->render;
  } => 'index';

  # 渲染 "hello.html.ep" 模板
  get '/hello';
   
  app->start;
  __DATA__
  @@ index.html.ep
  <%= link_to Hello  => 'hello' %>.
  <%= link_to Reload => 'index' %>.
   
  @@ hello.html.ep
  Hello World!
  ```

  匿名路由会生成一个自动分配的路由，这个路由简单地等于路由本身，没有非单词字符。
  Nameless routes get an automatically generated one assigned that is simply equal to the route itself without non-word characters.

* ### 布局
  模板支持布局，通过选择 [Mojolicious::Plugin::DefaultHelpers layout](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers#layout) 中的其中一个布局和放置
  [Mojolicious::Plugin::DefaultHelpers content](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers#content) 返回的模板实现。
  ```perl
  use Mojolicious::Lite;
   
  get '/with_layout';
   
  app->start;
  __DATA__
   
  @@ with_layout.html.ep
  % title 'Green';
  % layout 'green';
  Hello World!
   
  @@ layouts/green.html.ep
  <!DOCTYPE html>
  <html>
    <head><title><%= title %></title></head>
    <body><%= content %></body>
  </html>
  ```

  stash 方法和 Mojolicious::Plugin::DefaultHelpers 的 [title helpers](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers#title)，允许传递附加信息到布局中。


* ### 块
  模板块可以像普通Perl函数一样使用，并且总是以 begin 开始，以 end 关键字结束，这是许多 helpers 的基础。

  ```perl
  use Mojolicious::Lite;
   
  get '/with_block' => 'block';
   
  app->start;
  __DATA__
   
  @@ block.html.ep
  % my $link = begin
    % my ($url, $name) = @_;
    Try <%= link_to $url => begin %><%= $name %><% end %>.
  % end
  <!DOCTYPE html>
  <html>
    <head><title>Sebastians frameworks</title></head>
    <body>
      %= $link->('http://mojolicious.org', 'Mojolicious')
      %= $link->('http://catalystframework.org', 'Catalyst')
    </body>
  </html>
  ```

* ###　Helpers
  辅助器是一些通过 Mojolicious::Lite 的 helper 关键字创建的函数，并且从actions到模板，贯彻/复用于整个应用。
  ```perl
  use Mojolicious::Lite;
   
  # 一个鉴别访客身份的辅助器
  helper whois => sub {
    my $c     = shift;
    my $agent = $c->req->headers->user_agent || 'Anonymous';
    my $ip    = $c->tx->remote_address;
    return "$agent ($ip)";
  };
   
  # 在action动作和模板中使用辅助器
  get '/secret' => sub {
    my $c    = shift;
    my $user = $c->whois;
    $c->app->log->debug("Request from $user");
  };
   
  app->start;
  __DATA__
   
  @@ secret.html.ep
  We know who you are <%= whois %>.
  ```

  完整的内建名单可以在 [Mojolicious::Plugin::DefaultHelpers](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers) 和 [Mojolicious::Plugin::TagHelpers](https://metacpan.org/pod/Mojolicious::Plugin::TagHelpers) 中查看

  Placeholders

Route placeholders allow capturing parts of a request path until a / or . separator occurs, similar to the regular expression ([^/.]+). Results are accessible via "stash" in Mojolicious::Controller and "param" in Mojolicious::Controller.

use Mojolicious::Lite;
 
# /foo/test
# /foo/test123
get '/foo/:bar' => sub {
  my $c   = shift;
  my $bar = $c->stash('bar');
  $c->render(text => "Our :bar placeholder matched $bar");
};
 
# /testsomething/foo
# /test123something/foo
get '/<:bar>something/foo' => sub {
  my $c   = shift;
  my $bar = $c->param('bar');
  $c->render(text => "Our :bar placeholder matched $bar");
};
 
app->start;

To separate them from the surrounding text, you can surround your placeholders with < and >, which also makes the colon prefix optional.






