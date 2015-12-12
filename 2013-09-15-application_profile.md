---
layout: post
title: Luminus手册-应用配置
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---
Profiles
========

使用lein new luminus myapp命令会创建一个新的luminus应用，它使用的是默认的配置。如果你想使用其他的特性，你可以修改相应的配置参数。

下面就是各个配置：

-   +cljs - 添加ClojureScript支持
-   +cucumber - 提供通过clj-webdriver配置cucumber
-   +h2 - 添加models.db并提供H2数据库依赖
-   +postgres - 添加models.db并提供PostreSQL数据库依赖
-   +mysql - 添加models.db并提供mysql依赖
-   +mongodb - 添加db.core并提供MongoDB依赖
-   +site - 创建一个包含注册和验证框架的应用。使用bootstrap和h2
-   +dailycred -
    添加dailycred支持，当和+site参数配合使用时，它使用dailycred作为验证框架
-   +http-kit - 添加HTTP Kit支持

要添加配置，只需要简单的添加到你的应用的名字后面，比如:

```sh
lein new luminus myapp +cljs
```

可以多个参数一起使用:

```sh
lein new luminus myapp +site +postgres
```

如果两个参数会生成相同的文件，则后面参数所生成的文件会覆盖前面参数所生
成的文件。

<!-- more -->

HTTP Kit notes
==============

HTTP Kit是一个嵌入式的服务器。与Jetty不同的是，HTTP Kit并不被lein-ring
支持。所以你需要这样来运行HTTP Kit.

```sh
lein run
```

为了能热部署，你需要使用-dev选项，并且可以自定义端口号。默认端口号是8080
要打包一个可运行的HTTP Kit的jar不包，使用如下的命令:

```sh
lein uberjar
```
