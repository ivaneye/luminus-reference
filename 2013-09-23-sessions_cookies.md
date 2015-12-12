---
layout: post
title: Luminus手册-Session与Cookie
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---
Sessions
========

Session管理相关功能由noir.session提供。默认提供的noir.util.middleware/app-handler函数默认将Session保存在内存里。

当然你可以修改它，只需要给函数传递第二个参数，告诉它将Session保存在哪里就可
以了。 下面的例子创建了一个保存在内存里的session.

```clojure
(def app (middleware/app-handler [home-routes app-routes]))
```

下面，我们来重新定义session的保存位置。使用:session-options来替换掉默认的值就可以了。

```clojure
(def app
  (middleware/app-handler
    [home-routes app-routes]
    :session-options {:cookie-name "example-app-session"
                      :store (cookie-store)}))
```

Accessing the session
=====================

你可以在任意范围内的任何函数里访问并操作session。

```clojure
(require '[noir.session :as session])

(defn set-user [id]
  (session/put! :user id)
  (session/get :user))

(defn remove-user []
  (session/remove! :user)
  (session/get :user))

(defn set-user-if-nil [id]
  (session/get :user id))

(defn clear-session []
  (session/clear!))

(defroutes app-routes
  (GET "/login/:id" [id] (set-user id))
  (GET "/remove" [] (remove-user))
  (GET "/set-if-nil/:id" [id] (set-user-if-nil id))
  (GET "/logout" [] (clear-session)))
```

你还可以通过noir.session/flash-put!和noir.session/flash-get来创建flash变量.

```clojure
(session/flash-put! :message "User added!")
(session/flash-get :message)
```

<!-- more -->

Cookies
=======

Cookie处理函数由noir.cookies提供。设置或获取Cookie和Session很类似。

```clojure
(require '[noir.cookies :as cookies])

(defn set-user-cookie [id]
  (cookies/put! :user id)
  (cookies/get :user))

(defn set-user-if-nil [id]
  (cookies/get :user id))

(cookies/put! :track
              {:value (str (java.util.UUID/randomUUID))
              :path "/"
              :expires 1})
```
