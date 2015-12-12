---
layout: post
title: Luminus手册-路由定义
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Defining routes
===============

Luminus使用Compojure来定义应用的路由.路由由HTTP请求方法和接受它的URI，
参数和以及相应的处理程序组成。Compojure定义了所有标准HTTP请求的路由，
比如ANY,DELETE,GET,HEAD,OPTIONS,PATCH,POST和PUT。

例如：如果我们想定义一个应用，它指向/，/返回"Hello World!"字符串。我们
可以这样写:

```clojure
(defroutes app-routes
  (GET "/" [] "Hello World!"))
```

如果你想构建一个路由来响应POST请求，我们可以这样写：

```clojure
(POST "/hello" [id] (str "Welcome " id))
```

有些路由需要访问请求的map，我们只需要在路由上定义第二个参数就可以了。

```clojure
(GET "/foo" request (interpose ", " (keys request)))
```

上面的路由读出请求map中所有的key并展示出来。结果如下:

```sh
:ssl-client-cert, :remote-addr, :scheme, :query-params, :session, :form-params,
:multipart-params, :request-method, :query-string, :route-params, :content-type,
:cookies, :uri, :server-name, :params, :headers, :content-length, :server-port,
:character-encoding, :body, :flash
```

Compojure还提供了一些有用的函数来处理请求map和表单参数。例如，在
guestbook应用的例子中，我们看到了如下的路由定义:

```clojure
(POST "/"  [name message] (save-message name message))
```

<!-- more -->

这个路由从参数中获取到name和message的值，并将其绑定到相同名字的变量上。
我们能像使用其他定义的变量那样来使用他们。我们也可以直接在路由中使用Clojure的解构。

```clojure
(GET "/:foo" { {foo "foo"} :params}
  (str "Foo = " foo))
```

实际上，Compojure还提供了解构部分参数，并将其他参数构建为一个map的功能。

```clojure
[x y & z]
x -> "foo"
y -> "bar"
z -> {:v "baz", :w "qux"}
```

上面的代码中,参数x和y被绑定到了变量上，而v和w还在叫z的map中。
最后，如果我们需要获取完整的请求，我们可以这样做：

```clojure
(GET "/" [x y :as r] (str x y r))
```

这里我们将表单参数绑定到了x和y上，而完整的请求则绑定到了变量r上。

Organizing application routes
=============================

对路由管理的最佳实践就是对其统一管理。Compojure提供了defroutes宏来方便
我们作这样的处理。

```clojure
(defroutes auth-routes
  (POST "/login" [id pass] (login id pass))
  (POST "/logout" [] (logout)))

(defroutes app-routes
  (GET "/" [] (home))
  (route/resources "/")
  (route/not-found "Not Found"))
```

对于路径相似的路由，可以使用context来简化管理。
例如，你的路径都以/user/:id为根路径:

```clojure
(defroutes user-routes
      (GET "/user/:id/profile" [id] ...)
      (GET "/user/:id/settings" [id] ...)
      (GET "/user/:id/change-password" [id] ...))
```

你可以这样简写:

```clojure
(def user-routes
      (context "/user/:id" [id]
        (GET "/profile" [] ...)
        (GET "/settings" [] ...)
        (GET "/change-password" [] ...)))
```

当你定义好了所有的路由，你可以将他们添加到
noir.util.middleware/app-handler下的vector中，它在handler目录下。
你应该注意到了，生成的项目中已经定义了一个叫app的变量，你只需要添加新的路由就可以了。

```clojure
(def app (middleware/app-handler
           ;;add your application routes here
           [home-routes app-routes]
           ;;add custom middleware here
           :middleware []
           ;;add access rules here
           ;;each rule should be a vector
           :access-rules []))
```

详细的文档请键Compojure官方wiki.

Restricting access
==================

有些页面只能在特定条件下才可以访问。
例如，你可能想定义一个只能给管理员查看的页面或者一个用户管理页面，只给
当前用户看。
使用lib-noir中的noir.util.route可以达到这个目的，我们可以定义访问特殊
页面的逻辑

Marking Routes as Restricted
============================

noir.util.route/restricted宏被用来定义访问路由的方式:

```clojure
(GET "/private/:id" [id] (restricted "private!"))
```

如果我们想定义多个路由的访问权限，可以使用def-restricted-routes宏.

```clojure
(def-restricted-routes private-pages
  (GET "/profile" [] (show-profile))
  (GET "/my-secret-page" [] (show-secret-page))
  (GET "/another-secret-page" [] (another-secret-page)))
```

上面的代码等同于

```clojure
(defroutes private-pages
  (GET "/profile" [] (restricted (show-profile)))
  (GET "/secret-page1" [] (restricted (show-secret-page)))
  (GET "/secret-page2" [] (restricted (another-secret-page))))
```

默认情况下，受限访问的路由会检查请求是否符合所有的条件。

Specifying Access Rules
=======================

我们来看看如何创建一个路由，它只有在session中存在:user所对应的值时，
才能访问。 首先，我们需要引用noir.util.route和noir.session.

```clojure
(ns myapp.handler
  (:require ...
            [noir.util.route :refer [restricted]]
            [noir.session :as session]))
```

接着，我们将编写函数来实现上面我们所说的访问规则。函数必须接受request
作为其参数并且返回true或者false来确认页面是否可以访问。
下面就是该函数的实现，它检查当前session中是否存在user。如果没有则重定
向。默认重定向到根路径。

```clojure
(defn user-access [request]
  (session/get :user))

(def app
 (middleware/app-handler
   [app-routes]
   :access-rules [user-access]))
```

现在，所有的受限的处理程序在session中不存在:user时都会重定向到根目录。

Access Rule Groups
==================

当你以map的方式来提供访问规则，则可以进行进一步的处理。

-   :uri - 符合访问规则的URI
-   :uris - 和:uri类似，匹配多个URI
-   :redirect - 重定向地址
-   :on-fail - 重定向可选项，可以指定一个请求错误时的处理函数，函数必须
    以request为参数
-   :rule - 一个规则
-   :rules - 多个规则

:rules可以用在下面集中情形下。

```clojure
:rules [rule1 rule2]
:rules {:any [rule1 rule2]}
:rules {:every [rule1 rule2] :any [rule3 rule4]}
```

默认情况下，每个规则都必须通过才能访问成功，:any则表示只要符合任何一个
规则即可访问。下面是一些例子:

```clojure
(defn admin-access [req]
 (session/get :admin))

:access-rules [{:redirect "/access-denied"
                :rule user-access}]

:access-rules [{:uris ["/user/*" "/private*"]
                :rule user-access}]

:access-rules [{:uri "/admin/*" :rule admin-access}
               {:uri "/user/*"
                :rules {:any [user-access admin-access]}}]

:access-rules [{:on-fail (fn [req] "access restricted")
                :rule user-access}]
```

Cross Site Request Forgery Protection
=====================================

CSRF攻击指的是第三方通过一个验证过的用户来提交动作。当你的网站包含了一
个恶意链接或按钮或js的时候，很容易发生。
我们可以使用Ring-Anti-Forgery来放置csrf攻击. 首先添加[ring-anti-forgery
"0.2.1"]依赖。接着引入相应的包。

```clojure
(ns myapp.handler
  (:require
    ...
    [selmer.parser :refer [add-tag!]]
    [ring.util.anti-forgery :refer [anti-forgery-field]]
    [ring.middleware.anti-forgery :refer [wrap-anti-forgery]]))
```

接着，我们将添加wrap-anti-forgery中间件:

```clojure
(def app (middleware/app-handler
           ;;add your application routes here
           [home-routes app-routes]
           ;;add custom middleware here
           :middleware [wrap-anti-forgery]
           ;;add access rules here
           ;;each rule should be a vector
           :access-rules []))
```

添加完后一个随机字符串将会被赋给anti-forgery-token变量。所有的POST请求
将会包含一个叫\_~anti~-forgery-token的token.
接着我们在init函数中添加csrf标签:

```clojure
(defn init
  ...
  (add-tag! :csrf-token (fn [_ _] (anti-forgery-field)))
  ...)
```

然后在模板中使用:

```clojure
    <form name="input" action="/login" method="POST">
      { % csrf-token %}
      Username: <input type="text" name="user">
      Password: <input type="password" name="pass">
    <input type="submit" value="Submit">
    </form>
```

所有的POST请求如果不包含这个token，则会被拒绝。服务器返回403错误。

