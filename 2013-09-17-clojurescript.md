---
layout: post
title: Luminus手册-ClojureScript
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Summary
=======

ClojureScript可以作为JavaScript在客户端的替代品。使用ClojureScript可以
有如下有点：

-   在客户端和服务端使用相同的语言
-   在前后端共用相同的代码
-   简介流畅的语言
-   使用Leiningen管理依赖关系
-   不可变数据
-   功能强大的库
-   Adding ClojureScript Support

最简单的添加ClojureScript支持的方法就是在创建新项目的时候添加+cljs选项
。不过在现有的项目中添加ClojureScript也同样的简单。你只需要在
project.clj文件中添加如下内容即可。

```clojure
:plugins [...
          [lein-cljsbuild "0.3.0"]]

:hooks [... leiningen.cljsbuild]

:cljsbuild
{:builds [{:source-paths ["src-cljs"]
           :compiler {:output-to "resources/public/js/site.js"
                      :optimizations :advanced}}]}
```

上面的代码将在项目中添加cljsbuild插件和一个钩子。
所有的命名空间需要在项目根目录下的src-cljs目录内。注意，ClojureScript
文件需要以.cljs结尾。如果文件以.clj结尾，它也会被编译，不过它就无法访
问js命名空间。

Using Libraries
===============

使用ClojureScript的一个好处就是你可以使用Leiningen来管理客户端的依赖。
ClojureScript库和其他库一样都配置在project.clj文件中。

<!-- more -->

Running the Compiler
====================

开发ClojureScript应用最简单的方法就是自动运行编译器。这样，你的任何修
改都会立即生效。要开启自动编译，只需要输入如下命令:

```sh
lein cljsbuild auto
```

也可以使用一次编译的方式。这将会将所有的js编译到一个js文件中。

```sh
lein cljsbuild once
```

Advanced Compilation and Exports
================================

使用高级编译的话变量名会被编译器压缩。如果要确保函数被编译为js时函数名
不变，那么我们需要保证他们的名字是受保护的。而这只需要添加\^:export注解：

```clojure
(ns main)

(defn ^:export init []
  (js/alert "hello world"))
```

现在我们可以像这样来调用:

```clojure
    <script>
    main.init();
    </script>
```

如果我们在代码里使用js库，那么我们必须要保护我们所调用的函数名。例如，
如果我们想使用AlbumColors库，如果我们这样写:

```clojure
(defn ^:export init []
  (.getColors (js/AlbumColors. "/img/foo.jpg")
    (fn [[background]]
     (.log js/console background))))
```

当使用高级编译时AlbumColors和getColors将会被改写。为了不让编译器改写他
们，我们需要创建一个js文件，里面包含了我们需要保护的函数，并在代码里引
用它。

```js
var AlbumColors = {};
AlbumColors.getColors = function() {};
```

我们把上面的代码放到resource目录下一个叫externs.js文件里面。并在
cljsbuild丽引用它，像这样:

```clojure
{:source-paths ["src-cljs"]
     :compiler
     {:pretty-print false
      :output-to "resources/public/js/site.js"
      ;;specify the externs file to protect function names
      :externs ["resources/externs.js"]
      :optimizations :advanced}}
```

Interacting with JavaScript
===========================

所有的js函数和变量都是在js这个Namespace的。

\*方法调用\*

```clojure
(.method object params)

(.log js/console "hello world!")
```

\*访问属性\*

```clojure
(.-property object)

(.-style div)
```

\*设置属性\*

```clojure
(set! (.-property object))

(set! (.-color (.-style div) "#234567"))
```

更多的例子请查看[Himera文档](http://himera.herokuapp.com/synonym.html)

Working With the DOM
====================

有不少的库可以访问和修改DOM元素。一般用得比较多的是[Domina](https://github.com/levand/domina)和[Dommy](https://github.com/Prismatic/dommy)。
Domina对选择和操作DOM元素做了轻量级的封装。而Dommy是个类似hiccup的模板。

Ajax
====

Luminus使用cljs-ajax来处理Ajax请求。这个库提供的ajax-request，GET和
POST函数能很方便的发送ajax请求。

\*ajax-request\* ajax-request是最基本的请求函数，有如下参数:

-   uri - 请求URI
-   method - 一个表示http请求类型的字符串,比如："PUT", "DELETE"
-   format - 一个表示响应类型的关键字,比如： :json 或者 :edn,默认是 :edn
-   handler - 请求成功的回调函数，以响应为参数
-   error-handler - 请求失败的回调函数,以一个map为参数，map包含了错误信
    息，相应的key为: :status 和 :status-text
-   params - 一个包含参数的map，发送给服务器

\*GET/POST helpers\* GET和POST助手接收一个URL和选项，来简化请求：

-   :handler - 请求成功回调函数
-   :error-handler - 请求失败函数
-   :format - 相应格式化
-   :params - 发送给服务器的参数

和ajax-request参数功能相同。

```clojure
(ns foo
  (:require [ajax.core :refer [GET POST]]))

(defn handler [response]
  (.log js/console (str response)))

(defn error-handler [{:keys [status status-text]}]
  (.log js/console
    (str "something bad happened: " status " " status-text)))

(GET "/hello")

(GET "/hello" {:handler handler})

(POST "/hello")

(POST "/send-message"
        {:params {:message "Hello World"
                  :user    "Bob"}
         :handler handler
         :error-handler error-handler})
```
