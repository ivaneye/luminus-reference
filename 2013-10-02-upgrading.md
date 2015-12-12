---
layout: post
title: Luminus手册-升级
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Upgrading to the Latest Version
===============================

不幸的是，框架无法自动从老版本升级到新版本。主要原因是新版本的依赖可能会有接口方面的修改，导致老代码无法正常运行。

最好的方法是使用[lein-ancient](https://github.com/xsc/lein-ancient)插件来保持你的依赖包时刻为最新。这将会帮你很容易的发现哪些插件有了新的版本。

