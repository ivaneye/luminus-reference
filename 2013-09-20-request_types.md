---
layout: post
title: Luminus手册-请求
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Requests
========

默认情况下，请求的参数(比如一个form的POST请求)将会被自动绑定到request的:params键上。

但是，如果你在请求体内传递一些特殊类型的参数，则你需要使用适合的中间件来处理他们。Luminus使用ring-middleware-format来处理这些参数。

中间件可以通过在noir.util.middleware/app-handler上添加:formats键来开启:

```clojure
(def app (middleware/app-handler
          all-routes
          :formats [:json :edn]))
```

这样请求中的application/json和application/edn类型将会被中间件处理。相应的请求参数会在:params中。注意，这也会处理响应中的对应类型参数。具体信息请见Response Types章节。

下面是有效的格式化类型:

```clojure
:json - JSON with string keys in :params and :body-params
:json-kw - JSON with keywodized keys in :params and :body-params
:edn - native Clojure format.
:yaml - YAML format
:yaml-kw - YAML format with keywodized keys in :params and :body-params
:yaml-in-html - yaml in a html page
```
