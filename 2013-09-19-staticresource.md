---
layout: post
title: Luminus手册-静态资源
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Static resources
================

在noir.io下有多个访问静态资源的帮助方法。
你可以通过调用resource-path来获取public目录的路径。

Handling file uploads
=====================

上传文件通过noir.io空间下的upload-file方法来实现，upload-file接受一个路径和文件的map。
例如我们有一个upload.html文件:

```html
    <h2>Upload a file</h2>
    <form action="/upload" enctype="multipart/form-data" method="POST">
        <input id="file" name="file" type="file" />
        <input type="submit" value="upload" />
    </form>
```

我们可以这样来处理文件上传。

```clojure
(ns myapp.upload
  ...
  (:require [upload-test.views.layout :as layout]
            [noir.response :as response]
            [noir.io :as io]))

(defn upload-page []
  (layout/render "upload.html"))

(defn handle-upload [file]
  (io/upload-file "/" file)
  (response/redirect
    (str "/" (:filename file))))

(defroutes upload-routes
  (GET "/upload" [] (upload-page))
  (POST "/upload" [file] (handle-upload file)))
```

<!-- more -->

Serving static resources
========================

你可以使用get-resource通过相对路径来加载public目录中的资源，例如:

```clojure
(get-resource "/md/outline.md")
```

上面的代码将返回在包含public/md/outline.md下资源的clojure.java.io/resource。
最后，我们可以使用slurp-resource来读取文件的内容:

```clojure
(slurp-resource "/md/outline.md")
```
