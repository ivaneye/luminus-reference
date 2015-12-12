---
layout: post
title: Luminus手册-输入验证
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---
# Input Validation

验证相关助手方法在noir.validation命名空间中。如果我们要使用验证，我们需要先引入这个命名空间。

```clojure
(ns myapp.routes
  (:require ... [noir.validation :as v]))
```

验证函数在当验证不通过时，将相应的错误设置到对应的字段上。验证函数接受一个条件函数和一个包含了需要验证的字段和错误信息的Vector.当验证函数验证通过时需要返回true否则返回false.

下面是已经存在的验证函数:

```sh
has-value? args: [v] - returns true if v is truthy and not an empty string.
has-values? args: [coll] - returns true if all members of the collection has-value? This works on maps as well.
not-nil? args: [v] - returns true if v is not nil
min-length? args: [v len] - returns true if v is greater than or equal to the given len
max-length? args: [v len - returns true if v is less than or equal to the given len
matches-regex? args: [v regex] - returns true if the string matches the given regular expression
is-email? args: [v] - returns true if v is an email address
valid-file? args: [m] - returns true if a valid file was supplied
valid-number? args: [v] - returns true if the string can be parsed to a Long
greater-than? args: [v n] - returns true if the string represents a number > given
less-than? args: [v n] - returns true if the string represents a number < given
equal-to? args: [v n] - returns true if the string represents a number = given
```
例如：如果我们想检查id和pass字段不能为空，则可以创建如下验证规则：

````clojure
(v/rule (has-value? id)
        [:id "screen name is required"])

(v/rule (has-value? pass)
        [:pass "password is required"])
```

错误信息被保存在noir.validation/*errors*这个atom中，这个atom被绑定到了request上。每个error都包含一个key和一个对应的错误Vector.验证函数可以在同一字段上多次调用来设置多个错误。

一旦验证规则被执行了，我们可以通过errors?这个函数来检查是否有错误。如果没有传递参数，则此函数会检查noir.validation/*error*是否为空，如果传递了参数，次函数根据传递的值来查询是否有错误。

<!-- more -->

```clojure
;; returns true if any errors have been set
(v/errors?)
;; returns true if any errors have been set for the key :id
(v/errors? :id)
```

类似的，我们可以通过调用get-errors函数来获取当前的错误。当不传递参数时，返回所有的错误。

```clojure
;; returns a sequence of all errors that have been set
(v/get-errors)
;; returns all the errors set for the key :id
(v/get-errors :id)
```

我们还可以通过on-error来提供自定义的错误处理函数。这个函数需要一个字段相关联的error集合作为参数。函数结果由on-error返回。

```clojure
(v/on-error :id (fn [errors] (clojure.string/join ", " errors)))
```

最后，错误可以通过clear-errors!函数来清除。

```clojure
(v/clear-errors!)
```

下面是一个完整的登录相关验证。

```clojure
(defn set-errors [id pass]
  (v/rule (has-value? id)
        [:id "screen name is required"])

  (v/rule (has-value? pass)
        [:pass "password is required"])

  (if (v/errors? :id :pass)
    {:body {:status "error" :errors (get-errors)}}
    (do
    (session/put! :user id)
     {:body {:status "ok"}})))
```
