---
layout: post
title: Luminus手册-自定义中间件
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Adding custom middleware
========================

Luminus使用Ring来路由应用的处理程序，你也可以添加一些中间件。

中间件可以给处理程序添加一些额外的功能。中间件函数主要被用来扩展Ring的基本处理函数。

中间件其实就是个简单的函数，它接受现有的处理程序和一些可选参数作为函数参数并返回一个新的处理程序。

```clojure
(defn wrap-nocache [handler]
  (fn [request]
     (let [response (handler request)]
        (assoc-in response [:headers  "Pragma"] "no-cache"))))

(def app (wrap-nocache handler))
```

可以看到，上面的函数接受一个处理函数并返回一个新的函数，这个函数接受request作为参数。返回的函数只在handler所在的范围内有效。当调用这个函数时，它将会给响应map添加Pragma:no-cache键值对。

你可以添加自定义的中间件，只需要使用:middleware关键字就可以了:

```clojure
(def app (middleware/app-handler
          all-routes
          ;;put any custom middleware
          ;;in the middleware vector
          :middleware [wrap-nocache]))
```

具体内容可参见Ring官方文档。

<!-- more -->

Useful ring middleware
======================

-   ring-middleware-format - Ring middleware for handling different fromats such as JSON and EDN
-   ring-ratelimit - Rate limiting middleware
-   ring-etag-middleware - Calculates etags for ring responses and returns 304 responses when appropriate
-   ring-gzip-middleware - Gzips ring responses for user agents which can handle it
-   ring-upload-progress - Provide upload progress data in ring session
-   ring-anti-forgery - Ring middleware to prevent CSRF attacks

