---
layout: post
title: Luminus手册-HTML模板
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Templating Options
==================

Luminus包含了Hiccup的依赖.如果你熟悉Hiccup那么可以直接使用。
Hiccup使用clojure数据结构来描述模板。并且,Hiccup提供了丰富的API来生成
HTML元素。 Luminus也包含了Selmer，Selmer使用了普通的文本文件来描述模板。
你可以使用它们中的任何一个模板，或者混合使用.抑或你可以选择任何你喜欢
的模板框架，比如Enlive或者Stencil。

HTML Templating Using Selmer
============================

Selmer是一个类似Django模板的框架。如果你熟悉Django或者其他类似的模板语
言，那么你会感觉很熟悉.

Creating Templates
------------------

Selmer将展示逻辑和程序逻辑分开。实际上Selmer模板就是包含了动态元素的
HTML文件。来看下面的例子。

```html
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>My First Template</title>
    </head>
    <body>
        <h2>Hello { {name}}</h2>
    </body>
</html>
```

模板使用了一个包含了键值对的上下文环境。上下文环境中包含了我们
需要在运行时获得的变量的值。上面的代码中，我们从上下文环境中获取名字为
name的变量的值。
有两个函数可以渲染模板,render和render-file。render函数接收一个字符串来
渲染模板。而render-file接收一个字符串作为路径来渲染模板。
如果我们将上面的模板定义保存到index.html文件中。我们就可以这样来渲染:

```clojure
(ns example.routes.home
  (:use [selmer.parser :only [render-file]]))

(defn index [request]
  (render-file "example/views/templates/index.html"
               {:name "John"}))
```

<!-- more -->

render-file函数将src目录作为根路径，使用相对路径来查找模板。
上面，我们传递了一个字符串作为变量name的值。实际上，我们可以传递任何类
型的值。比如，我们传递一个集合，在模板里我们可以使用tag来遍历这个集合。

```html
    <ul>
        { % for item in items %}
        <li> { {item}} </li>
        { % endfor %}
    </ul>
```

```clojure
(render-file "/example/views/templates/items.html {:items (range 10)}")
```

如果传递的是个map，我们可以这样访问:

```clojure
(render "<p>Hello { {user.first}} { {user.last}}</p>"
        {:user {:first "John" :last "Doe"}})
```

如果没有指定的展示方式，那么默认调用toString方法。
默认情况下，Selmer会缓存编译后的模板。只有当文件发生变化时，才会触发
Selmer重新编译模板。你也可以选择Selmer是开启缓存还是关闭缓存。只需要调
用(selmer.parser/cache-on!)或(selmer.parser/cache-off!).


Filters
-------

Filter允许我们在渲染变量之前对其做一些处理。比如将其值变为大写，计算一
个hash或者是计算长度。Filter的使用方法很简单，只需要在变量名后面跟一个
"|"接着是Filter即可。比如：

```html
    { {name|upper}}
```

下面是内置的filter:

\*add\*

```clojure
(render "{ {add_me|add:2:3:4}}" {:add_me 2}) => 11
```

\*addslashes\* 为了保留字符串中的转义符""

```clojure
(render "{ {name|addslashes}}" {:name "\"Russian tea is best tea\""}) => "\"Russian tea is best tea\""
```

\*block.super\* 在代码块中插入上级代码块中的内容。


```clojure
        { % block foo %} { {block.super}} some content{ % endblock %}
```


\*capitalize\*

```clojure
(render "{ {name|capitalize}}" {:name "russian tea is best tea"}) => "Russian tea is best tea"
```

\*center\*

```clojure
(render "{ {name|center:20}}" {:name "bitemyapp"}) => " bitemyapp "
```

\*count\*

```clojure
(render "{ {name|count}}" {:name "Yogthos"}) => "7"

(render "{ {items|count}}" {:items [1 2 3 4]}) => "4"
```

\*date\*

```clojure
(render "{ {creation-time|date:\"yyyy-MM-dd_HH:mm:ss\"}}" {:created-at (java.util.Date.)}) => "2013-07-28_20:51:48"
```

\*default\*

```clojure
(render "{ {name|default:"I <3 ponies"}}" {:name "bitemyapp"}) => "bitemyapp"

(render "{ {name|default:"I <3 ponies"}}" {:name nil}) => "I <3 ponies"

(render "{ {name|default:"I <3 ponies"}}" {:name []}) => "[]"

(render "{ {name|default:"I <3 ponies"}}" {}) => "I <3 ponies"
```

\*default-if-empty\*

```clojure
(render "{ {name|default-if-empty:"I <3 ponies"}}" {:name "bitemyapp"}) => "bitemyapp"

(render "{ {name|default-if-empty:"I <3 ponies"}}" {:name nil}) => "I <3 ponies"

(render "{ {name|default-if-empty:"I <3 ponies"}}" {:name []}) => "I <3 ponies"

(render "{ {name|default-if-empty:"I <3 ponies"}}" {}) => "I <3 ponies"
```

\*double-format\*

```clojure
(render "{ {tis-a-number|double-format:2}}" {:tis-a-number 10.00001}) => 10.00

(render "{ {tis-a-number|double-format}}" {:tis-a-number 10.00001}) => 10.0
```

\*first\*

```clojure
(render "{ {seq-of-some-sort|first}}" {:seq-of-some-sort [:dog :cat :bird :bird :bird :is :the :word]}) => :dog
```

\*get-digit\* 从右边返回相应位数的数据。

```clojure
(render "{ {tis-a-number|get-digit:1}}" {:tis-a-number 12.34567}) => 7
```

\*hash\* 有效的hash方式: md5, sha, sha256, sha384, sha512

```clojure
(render "{ {domain|hash:\"md5\"}}" {:domain "example.org"}) => "1bdf72e04d6b50c82a48c7e4dd38cc69"
```

\*join\*

```clojure
(render "{ {sequence|join}}" {:sequence [1 2 3 4]}) => "1234"
```

\*json\* 生成json格式，默认情况下特殊字符会被编码.

```clojure
(render "{ {data|json}}" {:data [1 2 {:foo 27 :dan "awesome"}]})
               => "[1,2,{&quot;foo&quot;:27,&quot;dan&quot;:&quot;awesome&quot;}]"
```

如果你不希望被编码则使用safe过滤器

```clojure
(render "{ {f|json|safe}}" {:f {:foo 27 :dan "awesome"}})
```

\*last\*

```clojure
(render "{ {sequence|last}}" {:sequence 12.34567}) => 7

(render "{ {sequence|last}}" {:sequence [1 2 3 4]}) => 4
```

\*length\*

```clojure
(render "{ {sequence|length}}" {:sequence [1 2 3 4]}) => 4
```

\*length-is\*

```clojure
(render "{ {sequence|length-is:4}}" {:sequence [1 2 3 4]}) => true
```

\*linebreaks\* 普通换行替换为html换行,结果在\<p\>标签内

```clojure
(render "{ {foo|linebreaks|safe}}" {:foo "\nbar\nbaz"}) => "<p><br />bar<br />baz</p>"
```

\*linebreaks-br\* 和linebreaks功能相同，但是没有\<p\>标签

```clojure
(render "{ {foo|linebreaks-br|safe}}" {:foo "\nbar\nbaz"}) => "bar<br />baz"
```

\*linenumbers\* 显示行号

```clojure
(render "{ {foo|linenumbers" {:foo "foo\n\bar\nbaz"}) => "1. foo\n2. \bar\n3. baz"
```

\*lower\*

```clojure
(render "{ {foo|lower}}" {:foo "FOOBaR"}) => "foobar"
```

\*pluralize\* 返回单词的复数

```clojure
(render "{ {items|count}} item{ {items|pluralize}}" {:items []}) => "0 items"

(render "{ {items|count}} item{ {items|pluralize}}" {:items [1]}) => "1 item"

(render "{ {items|count}} item{ {items|pluralize}}" {:items [1 2]}) => "2 items"

(render "{ {fruit|count}} tomato{ {fruit|pluralize:\"es\"}}" {:fruit []}) => "0 tomatoes"

(render "{ {people|count}} lad{ {people|pluralize:\"y\":\"ies\"}}" {:people [1]}) => "1 lady"

(render "{ {people|count}} lad{ {people|pluralize:\"y\":\"ies\"}}" {:people [1 2]}) => "2 ladies"
```

\*rand-nth\* 从集合中返回rand-nths值

```clojure
(render "{ {foo|rand-nth}}" {:foo [1 2 3]}) => "2"
```

\*remove\* 从字符串中去除特殊字符

```clojure
(render "{ {foo|remove:\"aeiou\"}}" {:foo "abcdefghijklmnop"}) => "bcdfghjklmnp"
```

\*remove-tags\* 去除html标签

```clojure
(render "{ { value|remove-tags:b:span }}" {:value "<b><span>foobar</span></b>"}) => "foobar"
```

\*safe\* 默认情况下Selmer会编码所有的内容，此过滤器时Selmer不去编码

```clojure
(render "{ {data}}" {:data "<foo>"}) => "&lt;foo&gt;"

(render "{ {data|safe}}" {:data "<foo>"}) => "<foo>"
```

\*sort\*

```clojure
(render "{ { value|sort }}" {:value [1 4 2 3 5]}) => "(1 2 3 4 5)"
```

\*sort-by\*

```clojure
(render "{ { value|sort-by:name }}" {:value [{:name "John"} {:name "Jane"}]})
                     => "({:name &quot;Jane&quot;} {:name &quot;John&quot;})"
```

\*sort-reversed\* 反序排列

\*sort-by-reversed\* 和sort-by功能相同，反序

upper

```clojure
(render "{ {shout|upper}}" {:shout "hello"}) => "HELLO"
```

Defining Custom Filters
-----------------------

你可以使用selmer.filters/add-filter!函数来方便的添加自定义的Filter.此
函数以元素作为参数，并返回替换后的值.

```clojure
(use 'selmer.filters)

(add-filter! :embiginate #(.toUpperCase %))
 (render "{ {shout|embiginate}}" {:shout "hello"})

(add-filter! :count count)
(render "{ {foo|count}}" {:foo (range 3)})
```

Filters可以链式使用

```clojure
(add-filter! :empty? empty?)
(render "{ {foo|upper|empty?}}" {:foo "Hello"})
```

Tags
----

Selmer提供了两种类型的Tag。一种是类似extends和include这样的内联tag。这
种tag不需要结束tag。另一种形式的tag是块tag。这种tag有开始和结束tag，以
及代码块。比如if ... endif块。 让我们先来看看默认的tag:

\*include\* 将引用的模板内容替换其自身

```html
{ % include "path/to/comments.html" %}
```

你可以提供默认的参数，如果tag有匹配上上的参数，则会获取它的值。

```clojure
{ % include "templates/inheritance/child.html" with name="Jane Doe" greeting="Hello!" %}
```

\*block\*
定义一个块，可以使用模板继承的方式来覆盖，类似Java中的父类方法.

```html
    { % block foo %}This text can be overridden later{ % endblock %}
```

\*cycle\* 循环提供的参数

```clojure
(render "{ % for i in items %}<li class={ % cycle \"blue\" \"white\"%}>{ {i}}</li>{ % endfor %}" {:items (range 5)})
                        => "<li class=\"blue\">0</li><li class=\"white\">1</li><li class=\"blue\">2</li>
                            <li class=\"white\">3</li><li class=\"blue\">4</li>"
```

\*extends\* 引用父模板。父模板中的块会被子模板中相应的块覆盖。

-   Note: 子模板只能包含块。任何块以外的tag或文本都被忽略

例如有一个叫base.html的父模板和一个叫child.html的子模板

```html
<html>
  <body>
    { % block foo %}This text can be overridden later{ % endblock %}
  </body>
</html>
```

```html
    { % extends "base.html" %}
    { % block foo %}
      <p>This text will override the text in the parent</p>
    { % endblock %}
```

\*if\*

```html
{ % if condition %}yes!{ % endif %}

{ % if condition %}yes!{ % else %}no!{ % endif %}
```

配合filter使用

```clojure
(add-filter! :empty? empty?)
(render "{ % if files|empty? %}no files{ % else %}files{ % endif %}"
  {:files []})
```

\*ifequal\* 之用当两个参数相等时才执行

```html
{ % ifequal foo bar %}yes!{ % endifequal %}

{ % ifequal foo bar %}yes!{ % else %}no!{ % endifequal %}

{ % ifequal foo "this also works" %}yes!{ % endifequal %}
```

\*for\* 遍历集合中的元素。同时可以使用如下参数。
-   forloop.first
-   forloop.last
-   forloop.counter
-   forloop.counter0
-   forloop.revcounter
-   forloop.revcounter0
-   forloop.length

```html
{ % for x in some-list %}element: { {x}} first? { {forloop.first}} last? { {forloop.last}}{ % endfor %}
```

可以直接访问结构化的数据

```html
{ % for item in items %} <tr><td>{ {item.name}}</td><td>{ {item.age}}</td></tr> { % endfor %}
```

\*now\* 当前时间

```html
(render (str "{ % now \"" date-format "\"%}") {}) => "\"01 08 2013\""
```

\*comment\* 注释

```html
(render "foo bar { % comment %} baz test { {x}} { % endcomment %} blah" {}) => "foo bar baz test blah"
```

\*firstof\* 取得第一个不是false的值

```html
(render "{ % firstof var1 var2 var3 %}" {:var2 "x" :var3 "not me"}) => "x"
```

\*script\*
生成一个html的script标签，并以servlet-context键提供的值类作为文件的前
置目录.

```html
(render "{ % script \"/js/site.js\" %}" {:servlet-context "/myapp"}) =>
"<script src=\"/myapp/js/site.js\" type=\"text/javascript\"></script>"
```

\*style\*

生成一个html的style标签，并以servlet-context键提供的值类作为文件的前置目录.

```clojure
(render "{ % style \"/css/screen.css\" %}" {:servlet-context "/myapp"}) =>
"<link href=\"/myapp/css/screen.css\" rel=\"stylesheet\" type=\"text/css\" />"
```

\*verbatim\* 阻止内容中的parseeither解析

```clojure
(render "{ % verbatim %}{ {if dying}}Still alive.{ {/if}}{ % endverbatim %}" {}) => "{ {if dying}}Still alive.{ {/if}}"
```

\*with\* 向上下文map中新增一个值

```clojure
(render "{ % with total=business.employees|count %}{ { total }}{ % endwith %}"
         {:business {:employees (range 5)}})
=> "5 employees"
```

Defining Custom Tags
--------------------

除了上面已经提供的tag。你也可以自定义tag。使用add-tag!宏就可以了。

```clojure
(use 'selmer.parser)

(add-tag! :foo
  (fn [args context-map]
    (str "foo " (first args))))

(render "{ % foo quux %} { % foo baz %}" {})

(add-tag! :bar
  (fn [args context-map content]
    (str content))
  :baz :endbar)

(render "{ % bar %} some text { % baz %} some more text { % endbar %}" {})
```

可以看出，我们定义的tag提供了一个关键字作为tag的名称后面是tag体,结尾的
tag不是必须的。
当没有结尾的tag时,tag不包含任何内容。这种tag获取参数类进行处理。
当有结尾tag时，每个块内的内容都会被作为开头那个tag所对应的map的键.开头
tag对应的map包含:args和:content键，content的内容就对应到了:content.

Template inheritance
--------------------

Selmer模板可以使用块tag来引用其他的模板。有两种方法可以引用模板，一个
是extends一个是include.

### Extending Templates

当我们使用extends这个tag时，当前的模板将会将引用的模板作为父模板。
任何在base模板中的块，会被子模板中相同名字的块给覆盖.
子模板中的内容需要在块tag中。不在块中的内容将会被忽略。
我们来看个例子,我们创建一个叫base.html的父模板.

```html
<!DOCTYPE html>
<head>
    <link rel="stylesheet" href="style.css" />
        <title>{ % block title %}My amazing site{ % endblock %}</title>
</head>

<body>
    <div id="content">
        { % block content %}{ % endblock %}
    </div>
</body>
</html>
```

然后我们创建一个home.html模板来继承base.html。

```html
{ % extends "base.html" %}

    { % block content %}
        { % for entry in entries %}
            <h2>{ { entry.title }}</h2>
            <p>{ { entry.body }}</p>
        { % endfor %}
    { % endblock %}
```

当渲染home.html时entries的内容将会被打印出来。而在home.html中，我们没
有定义title块，在渲染时会获取base.html中的title块的定义。

-   注意，你可以多层级的继承模板。引擎会渲染最后的那个块.

### Including Templates

include这个tag允许你包含一个模板。
我们还是来看一个例子，我们有一个base.html这个模板，它包含了一个叫做
register.html的模板和一个home.html的模板：

```html
<!DOCTYPE html>
<head>
    <link rel="stylesheet" href="style.css" />
        <title>{ % block title %}My amazing site{ % endblock %}</title>
</head>

<body>
    <div id="content">
        { % if user %}
        { % include "templates_path/home.html" %}
        { % else %}
        { % include "templates_path/register.html" %}
        { % endif %}
    </div>
</body>
</html>
```

现在我们能定义register.html

```html
    { % block register %}
      <form action="/register" method="POST">
          <label for="id">user id</label>
          <input id="id" name="id" type="text"></input>
          <input pass="pass" name="pass" type="text"></input>
          <input type="submit" value="register">
      </form>
    { % endblock %}
```

和home.html:

```html
    { % block home %}
    <h1>Hello { {user}}</h1>
    { % endblock %}
```

当渲染base.html时，将会引入register.html和home.html。
更多信息请见[官方文档](https://github.com/yogthos/Selmer)

HTML Templating Using Hiccup
============================

[Hiccup](https://github.com/weavejester/hiccup)
是另一个Clojure的HTML模板引擎。它的优势是我们可以使用纯正的
Clojure代码来生成页面。也就是说我们不需要学习第三方的语法了。
Hiccup主要使用Clojure的vector来渲染页面。

```clojure
[:tag-name {:attribute-key "attribute value"} tag-body]
```

例如，如果我们想创建一个包含内容的div，我们可以这样写：

```clojure
[:div {:id "hello", :class "content"} [:p "Hello world!"]]
```

这将生成如下的html代码

```html
        <div id="hello" class="content"><p>Hello world!</p></div>
```

Hiccup提供了简化设置id和class的方法，简写如下

```clojure
[:div#hello.content [:p "Hello world!"]]
```

Hiccup还提供了一些帮助类函数来创建元素，比如: forms, links, images, 等
等.所有这些函数只是简单的返回类似上面格式的vector。也就是说，如果一个
函数无法达到你的要求，你可以手写相应代码或者获取函数的返回值进行修改。
每个函数可以使用一个map作为其第一个参数，来描绘属性:

```clojure
(image {:align "left"} "foo.png")
```

生成html代码如下

```clojure
<img align=\"left\" src=\"foo.png\">
```

但是最佳实践还是将其保存到独立的CSS文件中。

Forms and Input
---------------

Hiccup还提供了form助手函数，看例子

```clojure
(form-to [:post "/login"]
  (text-field {:placeholder "screen name"} "id")
  (password-field {:placeholder "password"} "pass")
  (submit-button "login"))
```

这个函数第一个参数是一个vector，以HTTP请求类型为关键字，紧接着是请求
url。其余的参数必须是HTML元素。生成的HTML代码如下:

```html
<form action="/login" method="POST">
  <input id="id" name="id" placeholder="screen name" type="text" />
  <input id="pass" name="pass" placeholder="password" type="password" />
  <input type="submit" value="login" />
</form>
```

最后LUminus模板在你的应用下面提供了一辅助函数。叫做md-\>html,这个函数能
渲染载resources/public/目录下的markdown文件并返回一个HTML字符串。这个
可以和Hiccup函数一起使用:

```clojure
(:require [<yourapp>.util :as util])
...
(html [:div.contenr [:p (util/md->html "/md/paragraph.md")]])
```

markdown生成由markdown-clj来处理，具体信息请见其[Github](https://github.com/yogthos/markdown-clj)
页。
Content caching
===============

lib-noir通过cache!宏提供了基本的内存式缓存，相应的宏在noir.util.cache
下。如果想缓存一个页面，你是需要：

```clojure
(require '[noir.util.cache :as cache])

(defn slow-loading-page []
  (cache/cache!
   :slow-page
   (common/layout
    [:div "I load slowly"]
     (parse-lots-of-files))))
```

通过调用invalidate!来取消缓存,需要提供需要取消的缓存的key。

```clojure
(cache/invalidate! :slow-page)
```

使用clear!来清除当前所有的缓存

```clojure
(cache/clear!)
```

使用set-timeout!来设置缓存时间(单位：秒)，超时后会重新加载。

```clojure
(cache/set-timeout! 10)
```

最后你可以通过set-size!来设置缓存的大小。当缓存超出了限定的大小，最先
使用的数据将被被替换。

```clojure
(cache/set-size! 10)
```

-   注意，缓存会在操作成功后才重新加载。也就是说，比如程序去获取一个远程
    文件，但是失败了，那么缓存是不会被重新加载的。
