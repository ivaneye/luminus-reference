---
layout: post
title: Luminus手册-部署
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Running Standalone
==================

要创建一个可独立运行的包，只需要运行如下的命令:

```sh
lein ring uberjar
```

打包完成的jar会出现在target目录下。可以通过下面的命令运行:

```clojure
java -jar myapp-0.1.0-SNAPSHOT-standalone.jar
```

独立可运行的程序可以使用Jetty来运行。如果要设置端口号，你需要设置\$PORT
环境变量:

```sh
export PORT=8080
java -jar target/myapp1-0.1.0-SNAPSHOT-standalone.jar
```

Delpoying on Immutant
=====================

如果想部署应用到Immutant，请执行如下命令:

```sh
lein immutant deploy
```

更多信息请访问[官网](http://immutant.org/tutorials/deploying/index.html)

Deploying to Tomcat
===================

如果想打包应用为war包:

```sh
lein ring uberwar
```

然后只需要将打好的包拷贝到tomcat的webapps目录下即可:

```sh
cp target/myapp-0.1.0-SNAPSHOT-standalone.war ~/tomcat/webapps/myapp.war
```

<!-- more -->

Heroku Deployment
=================

首先确保你有git和Heroku，然后按照下面的步骤做就可以了。
要测试你的应用是否通过foreman在本地运行，只需要执行下面的命令:

```sh
foreman start
```

现在你能初始化你的git仓库，并提交你的应用

```sh
git init
git add .
git commit -m "init"
```

在Heroku创建你的应用

```sh
heroku create
```

你还可以创建数据库

```sh
heroku addons:add heroku-postgresql
```

连接信息可以在你的Heroku页面看到. 部署应用.

```sh
git push heroku master
```

现在你的应用就被部署到了Heroku上.
具体信息请看[官方文档](https://devcenter.heroku.com/articles/clojure)

