* NAME
  Mojo::DOM - Minimalistic HTML/XML DOM parser with CSS selectors

* __语法__
  ```perl
  use Mojo::DOM;

  # 解析
  my $dom = Mojo::DOM->new('<div><p id="a">Test</p><p id="b">123</p></div>');

  # 查找
  say $dom->at('#b')->text;
  say $dom->find('p')->map('text')->join("\n");
  say $dom->find('[id]')->map(attr => 'id')->join("\n");

  # 迭代
  $dom->find('p[id]')->reverse->each(sub { say $_->{id} });

  # 循环
  for my $e ($dom->find('p[id]')->each) {
    say $e->{id}, ':', $e->text;
  }

  # 修改
  $dom->find('div p')->last->append('<p id="c">456</p>');
  $dom->at('#c')->prepend($dom->new_tag('p', id => 'd', '789'));
  $dom->find(':not(p)')->map('strip');

  # 渲染
  say "$dom";
  ```

* __概述__
  Mojo::DOM 是一个极简、易用、支持CSS选择器的HTML/XML DOM解析器。由于它甚至可以解析损坏的 HTML 和 XML，所以不适合用来校验一个HTML/XML是否语法正确。

* __节点和元素__
  当解析一个 HTML/XML 片段，将得到一个树状节点
  ```html
  <!DOCTYPE html>
  <html>
    <head><title>Hello</title></head>
    <body>World!</body>
  </html>
  ```

  现在有8个不同类型的节点，cdata, comment, doctype, pi, raw, root, tag 和 text。
  元素属于 tag 类型的节点。

  ```
  root
  |- doctype (html)
  +- tag (html)
     |- tag (head)
     |  +- tag (title)
     |     +- raw (Hello)
     +- tag (body)
        +- text (World!)
  ```

  目前所有节点类型使用 [Mojo::DOM](https://mojolicious.org/perldoc/Mojo/DOM) 对象的形式呈现，一些方法只对元素有效，如："[attr](https://mojolicious.org/perldoc/Mojo/DOM#attr)" 和 "[namespace](https://mojolicious.org/perldoc/Mojo/DOM#namespace)" 。

* __区分大小写__
  Mojo::DOM 默认采用 HTML 语义，这意味着标签、属性名称是小写的，选择器则必须是小写。
  ```perl
  # HTML 语义
  my $dom = Mojo::DOM->new('<P ID="greeting">Hi!</P>');
  say $dom->at('p[id]')->text;
  ```
  （这里示例中 HTML 标签是大写，Mojo会自动将其转为小写，所以一切都是小写的形式）

  如果找到一个 XML 声明，则解析器将自动切换到 XML 模式，进入大小写敏感模式。
  ```
  # XML 语义
  my $dom = Mojo::DOM->new('<?xml version="1.0"?><P ID="greeting">Hi!</P>');
  say $dom->at('P[ID]')->text;
  ```
 （XML中的标签可以大写也可以小写，Mojo将为其保留大小写状态）

  HTML 或 XML 语义也可以强制使用 "[xml](https://mojolicious.org/perldoc/Mojo/DOM#xml)" 方法处理。
  ```perl
  # 强制使用 HTML 语义
  my $dom = Mojo::DOM->new->xml(0)->parse('<P ID="greeting">Hi!</P>');
  say $dom->at('p[id]')->text;
  # 强制使用 XML 语义
  $dom = Mojo::DOM->new->xml(1)->parse('<P ID="greeting">Hi!</P>');
  say $dom->at('P[ID]')->text;
  ```
  （xml 方法 接受布尔值作为参数，0表示关闭xml模式，1表示开启xml模式 ）

* __方法__
  * all_text
    ```perl
    my $text = $dom->all_text;
    ```
    提取从元素所有子节点中的文本内容。
    ```perl
    # "foo\nbarbaz\n"
    $dom->parse("<div>foo\n<p>bar</p>baz\n</div>")->at('div')->all_text;
    ```

  * acestors（父级）
    ```perl
    my $collection = $dom->ancestors;
    my $collection = $dom->ancestors('div ~ p');
    ```
    寻找当前元素中匹配CSS选择器的所有上级元素，返回 [Mojo::Collection](https://mojolicious.org/perldoc/Mojo/Collection) 对象，这些对象包含这些元素（Mojo::DOM对象）。选择器用法参考：[Mojo::DOM::CSS SELECTORS](https://mojolicious.org/perldoc/Mojo/DOM/CSS#SELECTORS)。
    ```perl
    # List tag names of ancestor elements
    say $dom->ancestors->map('tag')->join("\n");
    ```
  * append
    ```perl
    $dom = $dom->append('<p>I ♥ Mojolicious!</p>');
    $dom = $dom->append(Mojo::DOM->new);
    ```
    追加 HTML/XML 片段到当前节点（对于root以外类型的节点）
    ```perl
    # "<div><h1>Test</h1><h2>123</h2></div>"
    $dom->parse('<div><h1>Test</h1></div>')
      ->at('h1')->append('<h2>123</h2>')->root;

    # "<p>Test 123</p>"
    $dom->parse('<p>Test</p>')->at('p')
      ->child_nodes->first->append(' 123')->root;
    ```

  + append_content

    ```perl
    $dom = $dom->append_content('<p>I ♥ Mojolicious!</p>');
    $dom = $dom->append_content(Mojo::DOM->new);
    ```

    添加 HTML/XML 片段 （对于root和tag节点）或raw content到当前节点的content。

    ```perl
    # "<div><h1>Test123</h1></div>"
    $dom->parse('<div><h1>Test</h1></div>')
      ->at('h1')->append_content('123')->root;

    # "<!-- Test 123 --><br>"
    $dom->parse('<!-- Test --><br>')
      ->child_nodes->first->append_content('123 ')->root;

    # "<p>Test<i>123</i></p>"
    $dom->parse('<p>Test</p>')->at('p')->append_content('<i>123</i>')->root;
    ```

  + at
    ```perl
    my $result = $dom->at('div ~ p');
    my $result = $dom->at('svg|line', svg => 'http://www.w3.org/2000/svg');
    ```

    寻找第一个与CSS选择器匹配的子元素，返回 Mojo::DOM 对象，如果未找到则返回 undef。 支持 [Mojo::DOM::CSS SELECTORS](https://mojolicious.org/perldoc/Mojo/DOM/CSS#SELECTORS) 中列出的所有选择器。

    ```perl
    # 寻找第一个 svg 命名的元素。 Find first element with "svg" namespace definition
    my $namespace = $dom->at('[xmlns\:svg]')->{'xmlns:svg'};
    ```

    可以追加键值对来声明 xml 的别名。  #1 有待确认
    Trailing key/value pairs can be used to declare xml namespace aliases.

    ```perl
    # "<rect />"
    $dom->parse('<svg xmlns="http://www.w3.org/2000/svg"><rect /></svg>')
      ->at('svg|rect', svg => 'http://www.w3.org/2000/svg');
    ```

  + attr
    ```perl
    my $hash = $dom->attr;
    my $foo  = $dom->attr('foo');
    $dom     = $dom->attr({foo => 'bar'});
    $dom     = $dom->attr(foo => 'bar');
    ```

    对应元素的属性
    ```perl
    # 移除一个属性
    delete $dom->attr->{id};
    # 不含值的属性
    $dom->attr(selected => undef);
    # 枚举 id 属性的值
    say $dom->find('*')->map(attr => 'id')->compact->join("\n");
    ```

  + child_nodes
    ```perl
    my $collection = $dom->child_nodes;
    ```
    返回 Mojo::Collection 对象，包含当前对象的所有子节点（这些子节点亦是 Mojo::DOM 对象 ）
    ```perl
    # "<p><b>123</b></p>"
    $dom->parse('<p>Test<b>123</b></p>')->at('p')->child_nodes->first->remove;
    # "<!DOCTYPE html>"
    $dom->parse('<!DOCTYPE html><b>123</b>')->child_nodes->first;
    # " Test "
    $dom->parse('<b>123</b><!-- Test -->')->child_nodes->last->content;
    ```

  + children
    ```perl
    my $collection = $dom->children;
    my $collection = $dom->children('div ~ p');
    ```
    寻找与CSS选择器匹配的所有子元素，返回包含这些元素（Mojo::DOM对象）的 Mojo::Collection 对象。

    ```perl
    # 随机显示子节点的标签名
    say $dom->children->shuffle->first->tag;
    ```

  + content
    ```perl
    my $str = $dom->content;
    $dom    = $dom->content('<p>I ♥ Mojolicious!</p>');
    $dom    = $dom->content(Mojo::DOM->new);
    ```
    返回当前节点的内容，或将其替换为 HTML/XML 片段（针对root或者指定标签节点）或者 raw content。
    ```perl
    # "<b>Test</b>"
    $dom->parse('<div><b>Test</b></div>')->at('div')->content;

    # "<div><h1>123</h1></div>"
    $dom->parse('<div><h1>Test</h1></div>')->at('h1')->content('123')->root;

    # "<p><i>123</i></p>"
    $dom->parse('<p>Test</p>')->at('p')->content('<i>123</i>')->root;

    # "<div><h1></h1></div>"
    $dom->parse('<div><h1>Test</h1></div>')->at('h1')->content('')->root;

    # " Test "
    $dom->parse('<!-- Test --><br>')->child_nodes->first->content;

    # "<div><!-- 123 -->456</div>"
    $dom->parse('<div><!-- Test -->456</div>')
      ->at('div')->child_nodes->first->content(' 123 ')->root;
    ```
  + 












