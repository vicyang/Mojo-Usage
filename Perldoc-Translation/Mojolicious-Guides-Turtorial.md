翻译：vicyang
[Perldoc-Translation](https://github.com/vicyang/Mojo-Usage)

Mojolicious::Guides::Tutorial - Get started with Mojolicious

* ### Tutorial
  [Mojolicious::Lite](https://metacpan.org/pod/Mojolicious::Lite) 教学示例。本章所涉及到的技巧同样适用于 [Mojolicious](https://metacpan.org/pod/Mojolicious) 应用

  这是 Mojolicious::Guides 中的起始章节，[growing]() 和 [Mojolicious::Lite]() 教学进一步指导如何构建完整的 Mojo 应用，建议读者在读完本章之后再阅读 [routing]()、 [rendering]() 等剩下的章节。

* ###　Hello World
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

* __路径名称__
  任何路径都可以有一个名称与之关联，这样就能实现自动模板检测，允许 Mojolicious::Controller的[url_for](https://metacpan.org/pod/Mojolicious::Controller#url_for)方法进行反向引用，Mojolicious::Plugin::TagHelpers的许多方法和的辅助器如 [link_to](https://metacpan.org/pod/Mojolicious::Plugin::TagHelpers#link_to) 都依赖于此。

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

  匿名路由会自动分配一个等同于自身路径（去除非单词字符）的名称
  （指的就是 `get '/hello';` ，等同于 `get '/hello' => sub { ... } => 'hello';` ）
  （实测 `get '/test' => 'hello';` 也是可以的 ）

* __布局__
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

  * 效果
    ```html
    <!DOCTYPE html>
    <html>
      <head><title>Green</title></head>
      <body>Hello World!
     
    </body>
    </html>
    ```

* __块__
  块模板可以像普通Perl函数一样使用，并且总是以 begin 开始，以 end 关键字结束，这是许多 helpers 的基础。

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

  * 输出结果
    ```html
    <!DOCTYPE html>
    <html>
      <head><title>Sebastians frameworks</title></head>
      <body>
          Try <a href="http://mojolicious.org">Mojolicious</a>.
          Try <a href="http://catalystframework.org">Catalyst</a>.
      </body>
    </html>
    ```
  link_to 的用法参考前面的 "路径名称" 一节

* __Helpers__
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
   
  # 在动作和模板中使用辅助器
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

  完整的内建辅助器名单可以在 [Mojolicious::Plugin::DefaultHelpers](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers) 和 [Mojolicious::Plugin::TagHelpers](https://metacpan.org/pod/Mojolicious::Plugin::TagHelpers) 中查看

* __占位符（Placeholders）__
  路径占位符捕获请求路径中的一部分，直到遇到 / 或者 . 分隔符。类似于正则表达式中的 ([^/.]+)。其结果通过Mojolicious::Controller的 [stash](https://metacpan.org/pod/Mojolicious::Controller#stash) 和 [param](https://metacpan.org/pod/Mojolicious::Controller#param) 方法获取。

  ```perl
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
  ```

  要将占位符从围绕的文本之中区别出来，可以用 < 和 > ，此时冒号前缀是可选的（可以直接`<bar>`）

* __宽松占位符__
  宽松占位符允许匹配任意符号直到遇到/分隔符，类似正则表达式 ([^/]+)
  ```perl
  use Mojolicious::Lite;
   
  # /hello/test
  # /hello/test.html
  get '/hello/#you' => 'groovy';
   
  app->start;
  __DATA__
   
  @@ groovy.html.ep
  Your name is <%= $you %>.
  ```
  （并且宽松占位符可以直接作为 $you 使用，不需要通过 stash 和 param 获取）

* __通配符（Wildcard placeholders）__
  通配符（占位符）允许匹配任意符号，包括 / 和 . ，类似正则表达式 (.+)。

  ```perl
  use Mojolicious::Lite;
   
  # /hello/test
  # /hello/test123
  # /hello/test.123/test/123
  get '/hello/*you' => 'groovy';
   
  app->start;
  __DATA__
   
  @@ groovy.html.ep
  Your name is <%= $you %>.
  ```

* __HTTP方法__
  路由可以被约束（绑定）到特定的多个请求方法，通过 Mojolicious::Lite 的 [any](https://metacpan.org/pod/Mojolicious::Lite#any) 关键字实现。

  ```perl
  use Mojolicious::Lite;
  # GET /hello
  get '/hello' => sub {
    my $c = shift;
    $c->render(text => 'Hello World!');
  };

  # PUT /hello
  put '/hello' => sub {
    my $c    = shift;
    my $size = length $c->req->body;
    $c->render(text => "You uploaded $size bytes to /hello.");
  };
   
  # GET|POST|PATCH /bye
  any ['GET', 'POST', 'PATCH'] => '/bye' => sub {
    my $c = shift;
    $c->render(text => 'Bye World!');
  };
   
  # * /whatever
  any '/whatever' => sub {
    my $c      = shift;
    my $method = $c->req->method;
    $c->render(text => "You called /whatever with $method.");
  };
   
  app->start;
  ```

  （在示例中，/whatever 路径支持任意请求方式，/bye路径同时支持 GET, POST 和 PATCH ）

* __可选占位符__
  任何占位符都会尝试进行捕获，但通过初始化赋值可以让捕获作为可选项。
  ```perl
  use Mojolicious::Lite;
   
  # /hello
  # /hello/Sara
  get '/hello/:name' => {name => 'Sebastian', day => 'Monday'} => sub {
    my $c = shift;
    $c->render(template => 'groovy', format => 'txt');
  };
   
  app->start;
  __DATA__
   
  @@ groovy.txt.ep
  My name is <%= $name %> and it is <%= $day %>.
  ```

  那些和占位符无关的默认值（这里指 day => 'Monday'），总是通过 stash 合并到参数（变量）列表中

* __限定占位符__
  通过提供名单，可以进一步限定占位符的可选项。（比如只有 "test" 和 "123" 可以匹配该路由）
  ```perl
  use Mojolicious::Lite;
  # /test
  # /123
  any '/:foo' => [foo => ['test', '123']] => sub {
    my $c   = shift;
    my $foo = $c->param('foo');
    $c->render(text => "Our :foo placeholder matched $foo");
  };
   
  app->start;
  ```
  所有占位符在内部被编译为正则表达式，这个过程是可定制的。只是确保不要使用 ^ 和 $，以及组合符号 (...)，虽然非组合捕获符号 (?:...) 是允许的。（不太明白这里的 ?:... 用法，待确认）
  ```perl
  use Mojolicious::Lite;
  # /1
  # /123
  any '/:bar' => [bar => qr/\d+/] => sub {
    my $c   = shift;
    my $bar = $c->param('bar');
    $c->render(text => "Our :bar placeholder matched $bar");
  };
  app->start;
  ```

  可以通过 [Mojolicious::Command::routes](https://metacpan.org/pod/Mojolicious::Command::routes) 命令进一步查看所有（路径）的正则表达式。
  `$ ./myapp.pl routes -v`
  （实际列出的只有占位符名称？）

* __Under__
  通过 Mojolicious::Lite 的 [under](https://metacpan.org/pod/Mojolicious::Lite#under) 方法生成路由，可以轻松实现在多个路由之间鉴权和代码共享。under以下所有的路由仅在指定回调函数返回值为真时执行。

  ```perl
  use Mojolicious::Lite; 
  # 基于name参数的鉴权/认证
  under sub {
    my $c = shift;
    # 通过认证
    my $name = $c->param('name') || '';
    return 1 if $name eq 'Bender';

    # 未通过验证
    $c->render(template => 'denied');
    return undef;
  };
   
  # 尽在鉴权通过时抵达这里
  get '/' => 'index';

  app->start;
  __DATA__
   
  @@ denied.html.ep
  You are not Bender, permission denied.
   
  @@ index.html.ep
  Hi Bender.
  ```

  另一个用法是将其作为多个路由的前缀。
  ```perl
  use Mojolicious::Lite;
   
  # /foo
  under '/foo';
   
  # /foo/bar
  get '/bar' => {text => 'foo bar'};
   
  # /foo/baz
  get '/baz' => {text => 'foo baz'};
   
  # / (reset)
  under '/' => {msg => 'whatever'};
   
  # /bar
  get '/bar' => {inline => '<%= $msg %> works'};
   
  app->start;
  ```

  也可以通过 Mojolicious::Lite 的 [group](https://metacpan.org/pod/Mojolicious::Lite#group) 方法组织路由关系，并且可以和 under 方法生成的路由镶嵌使用。

  ```perl
  use Mojolicious::Lite;
  # 全局共享的路由逻辑
  under sub {
    my $c = shift;
    return 1 if $c->req->headers->header('X-Bender');
    $c->render(text => "You're not Bender.");
    return undef;
  };
   
  # Admin section
  group {
    # 组内区域共享的路由逻辑
    under '/admin' => sub {
      my $c = shift;
      return 1 if $c->req->headers->header('X-Awesome');
      $c->render(text => "You're not awesome enough.");
      return undef;
    };
   
    # GET /admin/dashboard
    get '/dashboard' => {text => 'Nothing to see here yet.'};
  };
   
  # GET /welcome
  get '/welcome' => {text => 'Hi Bender.'};
   
  app->start;
  ```

  * 测试
    `$ua->get( "http://127.0.0.1:3000/welcome" );`
    You're not Bender.
    
    `$ua->get( "http://127.0.0.1:3000/welcome", 'X-Bender' => 'yep' );`
    Hi Bender.

    `$ua->get( "http://127.0.0.1:3000/admin/dashboard" );`
    You're not Bender.
    
    `$ua->get( "http://127.0.0.1:3000/admin/dashboard", 'X-Bender' => 'yep' );` 
    You're not awesome enough.

    `$ua->get( "http://127.0.0.1:3000/admin/dashboard", 'X-Bender'=>'yep', "X-Awesome"=>"yep")`
    Nothing to see here yet.

* 格式
  类似 .html 扩展名的文件格式将被自动探测，通常会寻找正确的模板和生成正确的 Content-Type 信息头。 #1
  ```perl
  use Mojolicious::Lite;
  # /detection
  # /detection.html
  # /detection.txt
  get '/detection' => sub {
    my $c = shift;
    $c->render(template => 'detected');
  };
   
  app->start;
  __DATA__
   
  @@ detected.html.ep
  <!DOCTYPE html>
  <html>
    <head><title>Detected</title></head>
    <body>HTML was detected.</body>
  </html>
 
  @@ detected.txt.ep
  TXT was detected.
  ```

  默认的格式是 html，可以用限定占位符来约束格式类型
  ```perl
  use Mojolicious::Lite;
  # /hello.json
  # /hello.txt
  get '/hello' => [format => ['json', 'txt']] => sub {
    my $c = shift;
    return $c->render(json => {hello => 'world'})
      if $c->stash('format') eq 'json';
    $c->render(text => 'hello world');
  };
   
  app->start;
  ```

  或者你可以通过一个特殊类型的限定占位符（[format => 0]）来禁用格式检测
  ```perl
  use Mojolicious::Lite;
  # /hello
  get '/hello' => [format => 0] => {text => 'No format detection.'};

  # 禁用格式探测，并且允许之后的路由在需要时重新开启格式探测
  under [format => 0];

  # /foo
  get '/foo' => {text => 'No format detection again.'};
  # /bar.txt
  get '/bar' => [format => 'txt'] => {text => ' Just one format.'};   
  app->start;
  ```

  （ get '/bar' 就是重新通过设置限定占位符来启用格式探测 ）

* __表单切换__
  为了资源的不同呈现和真正静默的表单切换，你可以使用 Mojolicious::Plugin::DefaultHelpers 的 [respond_to](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers#respond_to) 方法。

  ```perl
  use Mojolicious::Lite;
  # /hello (Accept: application/json)
  # /hello (Accept: application/xml)
  # /hello.json
  # /hello.xml
  # /hello?format=json
  # /hello?format=xml
  get '/hello' => sub {
    my $c = shift;
    $c->respond_to(
      json => {json => {hello => 'world'}},
      xml  => {text => '<hello>world</hello>'},
      any  => {data => '', status => 204}
    );
  };
  app->start;
  ```

  在Mojolicious中，可以轻松地扩展或修改 MIME 类型映射 #1

  ```perl
  app->types->type(rdf => 'application/rdf+xml');
  ```

* __静态文件__
  类似于模板，但只有固定的文件扩展名，只支持Base64编码，静态文件可以内嵌到Perl的 `__DATA__` 节点，并且自动生效。
  ```
  use Mojolicious::Lite;
  app->start;
  __DATA__
   
  @@ something.js
  alert('hello!');
   
  @@ test.txt (base64)
  dGVzdCAxMjMKbGFsYWxh
  ```

  外部静态文件则不受到单文件扩展名的限制，当其存放于 public 目录时自动生效。.

  $ mkdir public
  $ mv something.js public/something.js
  $ mv mojolicious.tar.gz public/mojolicious.tar.gz

  GET 和 HEAD 请求都拥有比路由更高的优先级别。[Mojolicious::Command::get](https://metacpan.org/pod/Mojolicious::Command::get)命令可以轻松实现 Headers 的 "If-None-Match" 和 "If-Modified-Since" 测试，以及针对 Content 的 Range 操作。 （参考 [HTTP Header 详解](https://www.cnblogs.com/yudapeng/p/6288646.html) ）

  `$ ./myapp.pl get /something.js -v -H 'Range: bytes=2-4'`
  （在Widnows命令行应使用双引号）
   -H, --header <name:value>  One or more additional HTTP headers
   Range: 指定了字节截取范围


* __外部模板__
  render方法会在templates目录中寻找外部模板。

  `$ mkdir -p templates/foo`
  `$ echo 'Hello World!' > templates/foo/bar.html.ep`

  他们拥有比Perl `__DATA__` 节点中的模板拥有更高的优先权

  ```perl
  use Mojolicious::Lite;
  # Render template "templates/foo/bar.html.ep"
  any '/external' => sub {
    my $c = shift;
    $c->render(template => 'foo/bar');
  };
  app->start;
  ```

* __Home__
  你可以使用 Mojolicious 的 [home](https://metacpan.org/pod/Mojolicious#home) 方法来指定应用的home目录。这将作为 public 和 templates 的父目录，你还可以可以用它来存储所有应用相关的数据。

  `$ mkdir cache`
  `$ echo 'Hello World!' > cache/hello.txt`

  Mojo::Home 有很多实用的方法继承自 Mojo::File，例如 Mojo::File 的 child 和 slurp 方法，这些方法可以帮助实现程序的跨平台特性。

  ```perl
  use Mojolicious::Lite;
  # 将信息读入内存
  my $hello = app->home->child('cache', 'hello.txt')->slurp;

  # 显示信息
  get '/' => sub {
    my $c = shift;
    $c->render(text => $hello);
  };
  ```

  你也可以通过在命令行中执行 [Mojolicious::Command::eval](https://metacpan.org/pod/Mojolicious::Command::eval) 命令来反向操作（配置）应用。

  `$ ./myapp.pl eval -v 'app->home'`

* __条件限定__
  通过 [Mojolicious::Plugin::HeaderCondition](https://metacpan.org/pod/Mojolicious::Plugin::HeaderCondition) 的 agent 和 host conditions 可以用来构建更强大的路由。

  ```perl
  use Mojolicious::Lite;
   
  # Firefox
  get '/foo' => (agent => qr/Firefox/) => sub {
    my $c = shift;
    $c->render(text => 'Congratulations, you are using a cool browser.');
  };
   
  # Internet Explorer
  get '/foo' => (agent => qr/Internet Explorer/) => sub {
    my $c = shift;
    $c->render(text => 'Dude, you really need to upgrade to Firefox.');
  };
   
  # http://mojolicious.org/bar
  get '/bar' => (host => 'mojolicious.org') => sub {
    my $c = shift;
    $c->render(text => 'Hello Mojolicious.');
  };
   
  app->start;
  ```

  测试
  `$ua->get( "http://127.0.0.1:3000/foo", 'user-agent'=>'firefox');`
  `$ua->get( "http://127.0.0.1:3000/foo", 'user-agent'=>'Internet Explorer');`
  `$ua->get( "http://127.0.0.1:3000/bar", 'host' => 'mojolicious.org');`

* __会话__
  基于 Cookie 的会话是开箱即用的，在你使用 Mojolicious::Plugin::DefaultHelpers 的 [session](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers#session) 辅助器时就启动了。只是要留意所有的会话都Mojo::JSON格式存储于客户端，通过签名加密的方式来防止篡改。

  ```perl
  use Mojolicious::Lite;
  # 在动作(action)和模板中处理会话
  get '/counter' => sub {
    my $c = shift;
    $c->session->{counter}++;
  };
  app->start;
  __DATA__
   
  @@ counter.html.ep
  Counter: <%= session 'counter' %>
  ```

  * 测试
    ```perl
    $ua = LWP::UserAgent->new( cookie_jar => HTTP::Cookies->new( file => "cookies_lwp.txt" ) );
    $res = $ua->get( "http://127.0.0.1:3000/counter");
    print $res->content();
    $res = $ua->get( "http://127.0.0.1:3000/counter");
    print $res->content();
    ```
    输出：
    Counter: 1
    Counter: 2

  注意应该使用自定的 [Mojolicious secrets](https://metacpan.org/pod/Mojolicious#secrets) 来防止cookies签名被篡改。

  `app->secrets(['My secret passphrase here']);`

* __文件上传__
  通过 multipart/form-data 请求上传的文件，由Mojolicious::Controller的 [param](https://metacpan.org/pod/Mojolicious::Controller#param)方法自动转为[Mojo::Upload](https://metacpan.org/pod/Mojo::Upload)对象。无需担心内存占用问题，因为所有超过250KiB的文件将被自动以数据流的形式存入临时文件。为了更高效地构建HTML表单，你可以使用 tag 辅助器如：Mojolicious::Plugin::TagHelpers 的 [form_for](https://metacpan.org/pod/Mojolicious::Plugin::TagHelpers#form_for) 辅助实现。

  * 关于 multipart/form-data
    [w3.org - Multipart form data](https://www.w3.org/TR/html5/sec-forms.html#multipart-form-data)
    [Returning Values from Forms: multipart/form-data](https://tools.ietf.org/html/rfc7578)
    [什么是multipart/form-data请求](http://www.cnblogs.com/tylerdonet/p/5722858.html) ）

  ```perl
  use Mojolicious::Lite;
   
  # 从Perl __DATA__ 节点上传
  get '/' => 'form';
   
  # Multipart upload handler
  post '/upload' => sub {
    my $c = shift;
   
    # 检查文件大小
    return $c->render(text => 'File is too big.', status => 200)
      if $c->req->is_limit_exceeded;
   
    # 处理上传文件
    return $c->redirect_to('form') unless my $example = $c->param('example');
    my $size = $example->size;
    my $name = $example->filename;
    $c->render(text => "Thanks for uploading $size byte file $name.");
  };
   
  app->start;
  __DATA__
   
  @@ form.html.ep
  <!DOCTYPE html>
  <html>
    <head><title>Upload</title></head>
    <body>
      %= form_for upload => (enctype => 'multipart/form-data') => begin
        %= file_field 'example'
        %= submit_button 'Upload'
      % end
    </body>
  </html>
  ```

  为了避免被过大的文件占用资源，默认限制 16MiB 大小，可以通过 [max_request_size](https://metacpan.org/pod/Mojolicious#max_request_size) 属性修改这项限制。

  ```
  # Increase limit to 1GiB
  app->max_request_size(1073741824);
  ```

* __User agent__
  通过 [Mojolicious::Plugin::DefaultHelpers ua](https://metacpan.org/pod/Mojolicious::Plugin::DefaultHelpers#ua) 辅助器使用 Mojo::UserAgent 功能，这是一个内建的支持完整HTTP特性和网络套接字的模块。和 [Mojo::JSON](https://metacpan.org/pod/Mojo::JSON) 和 [Mojo::DOM](https://metacpan.org/pod/Mojo::DOM) 模块配合使用更为强大。

  ```perl
  use Mojolicious::Lite;
   
  # Blocking
  get '/headers' => sub {
    my $c   = shift;
    my $url = $c->param('url') || 'https://mojolicious.org';
    my $dom = $c->ua->get($url)->result->dom;
    $c->render(json => $dom->find('h1, h2, h3')->map('text')->to_array);
  };
   
  # Non-blocking
  get '/title' => sub {
    my $c = shift;
    $c->ua->get('mojolicious.org' => sub {
      my ($ua, $tx) = @_;
      $c->render(data => $tx->result->dom->at('title')->text);
    });
  };
   
  # Concurrent non-blocking
  get '/titles' => sub {
    my $c  = shift;
    my $mojo = $c->ua->get_p('https://mojolicious.org');
    my $cpan = $c->ua->get_p('https://metacpan.org');
    Mojo::Promise->all($mojo, $cpan)->then(sub {
      my ($mojo, $cpan) = @_;
      $c->render(json => {
        mojo => $mojo->[0]->result->dom->at('title')->text,
        cpan => $cpan->[0]->result->dom->at('title')->text
      });
    })->wait;
  };
   
  app->start;
  ```

  更多关于 user agent 的信息请参考 [Mojolicious::Guides::Cookbook "USER AGENT"](https://metacpan.org/pod/distribution/Mojolicious/lib/Mojolicious/Guides/Cookbook.pod#USER-AGENT)

* __WebSockets__
  Socket 应用从未如此简单，通过预订事件如：[Mojo::Transaction::WebSocket json](https://metacpan.org/pod/Mojo::Transaction::WebSocket#json) 、 [Mojolicious::Controller on](https://metacpan.org/pod/Mojolicious::Controller#on) 、 [Mojolicious::Controller send](https://metacpan.org/pod/Mojolicious::Controller#send) 来获得消息反馈。 #1

  ```perl
  use Mojolicious::Lite;
   
  websocket '/echo' => sub {
    my $c = shift;
    $c->on(json => sub {
      my ($c, $hash) = @_;
      $hash->{msg} = "echo: $hash->{msg}";
      $c->send({json => $hash});
    });
  };
   
  get '/' => 'index';
   
  app->start;
  __DATA__
   
  @@ index.html.ep
  <!DOCTYPE html>
  <html>
    <head>
      <title>Echo</title>
      <script>
        var ws = new WebSocket('<%= url_for('echo')->to_abs %>');
        ws.onmessage = function (event) {
          document.body.innerHTML += JSON.parse(event.data).msg;
        };
        ws.onopen = function (event) {
          ws.send(JSON.stringify({msg: 'I ♥ Mojolicious!'}));
        };
      </script>
    </head>
  </html>
  ```

  更多关于实时网络特性的资料，参考 [Mojolicious::Guides::Cookbook - REAL-TIME WEB](https://metacpan.org/pod/distribution/Mojolicious/lib/Mojolicious/Guides/Cookbook.pod#REAL-TIME-WEB)

  可以通过 Mojolicious 的 [log](https://metacpan.org/pod/Mojolicious#log) 方法操作 [Mojo::Log](https://metacpan.org/pod/Mojo::Log) 对象，打印调试信息。在生产环境中可以使用 Mojolicious 的 [mode](https://metacpan.org/pod/Mojolicious#mode) 方法切换 Mojolicious 模式，关闭调试信息。

  ```perl
  use Mojolicious::Lite;
   
  # 在启动时显示指定模式的相关信息
  my $msg = app->mode eq 'development' ? 'Development!' : 'Something else!';
   
  get '/' => sub {
    my $c = shift;
    $c->app->log->debug('Rendering mode specific message');
    $c->render(text => $msg);
  };
   
  app->log->debug('Starting application');
  app->start;
  ```

  默认操作模式为开发模式，并且可以通过命令行环境变量 MOJO_MODE 和 PLACK_ENV 来切换。其他模式下，日志级将会从 debug 调至 info。

  `$ ./myapp.pl daemon -m production`

  所有消息将被写入 STDERR 或者 log/$mode.log 文件（如果存在 log 目录）。

  `$ mkdir log`

  模式切换同样会影响框架的其他方面，例如内建的异常处理和 not_found 页面信息。一旦你从开发模式切换到生产模式，这些页面将不再显示敏感信息。

* __Testing__
  测试应用和创建目录写入Perl测试文件（t/basic.t）一样简单，这些都要归功于 Test::Mojo 模块。
  ```perl
  use Test::More;
  use Mojo::File 'path';
  use Test::Mojo;
   
  # Portably point to "../myapp.pl"
  my $script = path(__FILE__)->dirname->sibling('myapp.pl');
   
  my $t = Test::Mojo->new($script);
  $t->get_ok('/')->status_is(200)->content_like(qr/Funky/);
   
  done_testing();
  ```

  通过 prove 命令进行测试
  `$ prove -l -v`
  `$ prove -l -v t/basic.t`

* __更多__
  你可以继续浏览 Mojolicious::Guides 或者看看 [Mojolicious wiki](http://github.com/mojolicious/mojo/wiki)，其中包含更多文档和来自不同作者的示例

  如果你有任何关于文档的问题，不要犹豫直接去邮件列表提问或者在官方IRC频道：irc.freenode.net #mojo 咨询
















