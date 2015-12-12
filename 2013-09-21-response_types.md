---
layout: post
title: Luminus手册-响应
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Responses
=========

noir.response提供了很多辅助函数来处理响应.

Setting headers
===============

可以使用set-header来设置响应头，使用一个map作为参数。这里需要注意的是,map的key必须是字符串。

```clojure
(set-headers {"x-csrf" csrf}
    (common/layout [:p "hi there"]))
```

Setting content type
====================

你可以使用content-type函数来设置响应的类型:

```clojure
(GET "/project" []
       (noir.response/content-type
       "application/pdf"
       (clojure.java.io/input-stream "report.pdf")))
```

下面是有效的响应类型设置:

-   XML - Wraps the response with the content type for xml and sets the body to the content.
-   JSON- Wraps the response in the json content type and generates JSON from the content
-   JSONP - Generates JSON for the given content and creates a javascript response for calling func-name with it.
-   edn - Wraps the response in the application/edn content-type and calls pr-str on the Clojure data stuctures passed in.

```clojure
(GET "/xml" [] (xml "<foo></foo>"))
(GET "/json" [] (json {:response "ok"}))
(GET "/jsonp" [] (jsonp  "showUsers" [{:name "John"} {:name "Jane"}]))
(GET "/edn" [] (edn {:foo 1 :bar 2}))
```

为了设置响应类型，你还需要设置noir.util.middleware/app-handler中间件。和上一节的请求一样，你只需要配置:formats键就可以了

```clojure
(app-handler routes :formats [:json])
```
<!-- more -->

可用的格式化类型:

-   :json - JSON with string keys in :params and :body-params
-   :json-kw - JSON with keywodized keys in :params and :body-params
-   :edn - native Clojure format.
-   :yaml - YAML format
-   :yaml-kw - YAML format with keywodized keys in :params and :body-params
-   :yaml-in-html - yaml in a html page
-   Setting custom status

可以使用status函数来设置一个自定义的状态

```clojure
(GET "/missing-page" [] (status 404 "your page could not be found"))
```

Redirects
=========

noir.response/redirect可以进行重定向。他可以向重定向函数传递参数。他支持如下类型:
-   :permanent
-   :found
-   :see-other
-   :not-modified
-   :proxy
-   :temporary

如果不传递参数的话，那么默认为:found

```clojure
(require 'noir.response)

(redirect "/foo")
(redirect "/bar" :see-other)
```
