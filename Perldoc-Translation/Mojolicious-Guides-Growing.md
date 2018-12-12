翻译：vicyang
[Perldoc-Translation](https://github.com/vicyang/Mojo-Usage)

* __NAME__
  Mojolicious::Guides::Growing - Growing Mojolicious applications

* 摘要
  这份文档解释 Mojolicious::Lite 的初始原型，并将其扩展壮大至完整的 [Mojolicious](https://metacpan.org/pod/Mojolicious) 应用

* __概念__
  任何 [Mojolicious](https://metacpan.org/pod/Mojolicious) 开发者都应该知道的基础

* __Model View Controller（模型、视图、控制台）__
  MVC是一种软件架构模式，起源于 SmallTalk-80 的GUI编程，用于划分应用逻辑、呈现和输入。
  ```
           +------------+    +-------+    +------+
  Input -> | Controller | -> | Model | -> | View | -> Output
           +------------+    +-------+    +------+
  ```

  将应用逻辑转移到控制台的改版模型成为每个优秀网站架构的基础，[Mojolicious](https://metacpan.org/pod/Mojolicious) 就是其一。
  ```
              +----------------+     +-------+
  Request  -> |                | <-> | Model |
              |                |     +-------+
              |   Controller   |
              |                |     +-------+
  Response <- |                | <-> | View  |
              +----------------+     +-------+
  ```

  控制台接受来自用户的请求，数据传入到 模型 并从中还原数据，然后通过 视图 返还一个实际响应。注意这个框架（图）只是用来引导实现更多可维护和清晰的代码，并不是在所有情况下都这样实现。

* __REpresentational State Transfer__
  REST 是一个用来构建网络超媒体系统的架构风格。虽然它可以应用到多种协议，但目前最常见的是配合HTTP使用。在REST规范下，当你在浏览器打开地址为 http://mojolicious.org/foo 的URL时，实际是在向网络服务器请求 http://mojolicious.org/foo 资源的HTML呈现。

  ```
  +--------+                                  +--------+
  |        | -> http://mojolicious.org/foo -> |        |
  | Client |                                  | Server |
  |        | <-  <html>Mojo rocks!</html>  <- |        |
  +--------+                                  +--------+
  ```

  


  


