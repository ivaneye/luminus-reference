---
layout: post
title: Luminus手册-国际化
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Internationalization
====================

Luminius模板包含了[Tower](https://github.com/ptaoussanis/tower)依赖，Tower提供了函数式的国际化和翻译。

首先，我们需要创建一个字典map：

```clojure
(def tconfig
  {:fallback-locale :en-US
   :dictionary
   {:en-US   {:page {:title "Here is a title"
                     :content "Time to start building your site."}
              :missing  "<Missing translation: [%1$s %2$s %3$s]>"}
    :fr-FR {:page {:title "Voici un titre"
                   :content "Il est temps de commencer votre site."}}
    }})
```

然后添加Tower中间件来包装我们的配置:

```clojure
(def app
 (app-handler
   [routes]
   :middleware
   [#(taoensso.tower.ring/wrap-tower-middleware % {:tconfig tconfig})]))
```

中间件将会通过accept-language头来确定相应的语言。中间件将会添加两个key到请求中。地一个key是:t,它的值是翻译函数。第二个key是:locale，它的值是中间件提供的语言类型。

你可以提供一个自定义的locale-selector函数给中间件。

```clojure
(defn my-selector [req]
  (when (= (:remote-addr req) "127.0.0.1") :en))

(def app
  (app-handler
    [routes]
    :middleware
    [#(wrap-tower-middleware % {:tconfig tconfig
                                :locale-selector my-selector})]))
```

<!-- more -->

中间件将按照如下顺序查找，会使用第一个有效的语言来进行处理:

```clojure
(:locale request)
(when-let [ls locale-selector] (ls request))
(:locale session)
(:locale params)
(locale-from-headers headers)
fallback-locale
```

当中间件设置完成后，我们可以通过下面的代码来实现国际化:

```clojure
(ns mysite.routes.home
  (:use compojure.core)
  (:require [i18ntest.layout :as layout]
            [i18ntest.util :as util]
            [taoensso.tower :refer [t]]))

(defn home-page [{:keys [locale tconfig]}]
  (layout/render
    "home.html" {:title   (t locale tconfig :page/title)
                 :content (t locale tconfig :page/content)}))

(defn about-page []
  (layout/render "about.html"))

(defroutes home-routes
  (GET "/" req (home-page req))
  (GET "/about" [] (about-page)))
```
