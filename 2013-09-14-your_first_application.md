---
layout: post
title: Luminus手册-第一个应用
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

# Guestbook应用

此文使用Luminus构建一个简单的guestbook应用。guestbook可以保存信息，展示信息。此应用将涉及到简单的HTML模板，数据库访问和项目结构.

# 安装Leiningen

首先你需要安装Leiningen才能使用Luminus。安装Leiningen非常的简单:

-   下载脚本
-   将其设置为可执行权限(chmod +x lein)
-   将脚本放到你的PATH下面
-   运行lein self-install ，然后等待安装结束


```sh
wget https://raw.github.com/technomancy/leiningen/stable/bin/lein
chmod +x lein
mv lein ~/bin
lein self-install
```

# 创建一个新应用

安装完Leiningen后，你就可以在命令行中输入如下的命令

```sh
lein new luminus guestbook +h2
cd guestbook
```

上面的命令将会创建一个使用了H2嵌入式数据库的模板项目。

<!-- more -->

# 剖析Luminus应用

新创建的项目的目录结构如下

```sh
guestbook
|____.gitignore
|____Procfile
|____project.clj
|____README.md
|
|____src
| |____guestbook
|   |____handler.clj
|   |____layout.clj
|   |____middleware.clj
|   |____session.clj
|   |____routes
|   | |____home.clj
|   |____db
|     |____core.clj
|
|____test
| |____guestbook
|   |____test
|     |____handler.clj
|
|____resources
| |____templates
| | |____about.html
| | |____base.html
| | |____home.html
| |____public
| | |____css
| | | |____screen.css
| | |____img
| | |____js
| |____docs
| | |____docs.md
| |____sql
|   |____queries.sql
|
|____migrations
  |____201501155317-add-users-table.down.sql
  |____201501155317-add-users-table.up.sql
```

我们先来看一看根目录下的文件的作用:

-   Procfile - 部署相关信息
-   README.md - 项目相关的文档
-   project.clj - 用于管理项目的配置以及Leiningen依赖关系
-   .gitignore - Git中不提交的文件记录在此文件中

# 源代码目录

所有的代码都在src目录下。我们的应用名称叫guestbook，这个名字同时也是源代码的根命名空间。让我们来看一下源代码目录下所有的命名空间。

## guestbook

-   handler.clj - 定义了应用最基本的路由。这是应用的入口。我们自定义的所有的页面都需要在这里添加各自的路由定义。
-   layout.clj - 页面布局帮助命名空间
-   middleware.clj - 中间件命名空间
-   session.clj - 创建内存session

## guestbook.db

db命名空间下定义的是应用所使用的model以及持久化相关操作.

-   core.clj - 包含一组可以和数据库交互的函数

## guestbook.routes

routes命名空间是存放路由以及controller的地方。当你要添加路由的时候，比如安全验证，特殊流程等等，你需要在这里创建他们。

-   home.clj - 一个定义了home和about页面的命名空间

# 测试目录

这里是放置测试代码的地方。该目录下已经有样例测试代码。

# 资源目录

这里是存放静态资源的地方。包括css,javascript,images和markdown.

## HTML模板

这个命名空间存放的是Selmer模板文件，用于应用页面的展示。

-   about.html - about页面
-   base.html - base页面
-   home.html - home页面

# SQL查询

sql查询语句在resources/sql目录中。

- queries.sql：定义sql查询以及关联的函数名称

# 数据迁移目录

Luminus使用Ragtime来管理数据迁移。数据通过使用up，down文件来进行迁移。文件通过日期来进行版本管理，并按顺序执行

- 201501155317-add-users-table.up.sql - 创建表
- 201501155317-add-users-table.down.sql - 删除表

# project文件

上面已经说过了，项目所有的依赖关系都是由project.clj来管理的。这个文件看起来像这样。

```clojure
(defproject guestbook "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"

  :dependencies [[org.clojure/clojure "1.6.0"]
                 [ring-server "0.3.1"]
                 [selmer "0.8.0"]
                 [com.taoensso/timbre "3.3.1"]
                 [com.taoensso/tower "3.0.2"]
                 [markdown-clj "0.9.62"]
                 [environ "1.0.0"]
                 [im.chit/cronj "1.4.3"]
                 [ring/ring-defaults "0.1.3"]
                 [ring/ring-session-timeout "0.1.0"]
                 [ring-middleware-format "0.4.0"]
                 [noir-exception "0.2.3"]
                 [bouncer "0.3.2"]
                 [prone "0.8.0"]
                 [ragtime "0.3.8"]
                 [yesql "0.5.0-rc1"]
                 [com.h2database/h2 "1.4.182"]]

  :min-lein-version "2.0.0"
  :uberjar-name "guestbook.jar"
  :repl-options {:init-ns guestbook.handler}
  :jvm-opts ["-server"]

  :plugins [[lein-ring "0.9.0"]
            [lein-environ "1.0.0"]
            [lein-ancient "0.5.5"]
            [ragtime/ragtime.lein "0.3.8"]]

  :ring {:handler guestbook.handler/app
         :init    guestbook.handler/init
         :destroy guestbook.handler/destroy
         :uberwar-name "guestbook.war"}

  :ragtime
  {:migrations ragtime.sql.files/migrations
   :database "jdbc:h2:./site.db"}

  :profiles
  {:uberjar {:omit-source true
             :env {:production true}
             :aot :all}
   :production {:ring {:open-browser? false
                       :stacktraces?  false
                       :auto-reload?  false}}
   :dev {:dependencies [[ring-mock "0.1.5"]
                        [ring/ring-devel "1.3.2"]
                        [pjstadig/humane-test-output "0.6.0"]]
         :injections [(require 'pjstadig.humane-test-output)
                      (pjstadig.humane-test-output/activate!)]
         :env {:dev true}}})
```

project.clj就是个简单的Clojure的list，这个list中包含了键值对，描述了应用的方方面面。如果你要添加自定义依赖，只需要简单的将需要的依赖添加到:dependencies这个vector内。

:plugins后的vector可以用来添加额外的函数，比如说Ragtime这个数据迁移插件。

:profiles包含不同的项目配置，可以以开发模式还是部署模式来进行启动。

请到Leiningen官方文档中查看完整的project.clj文件。

# 创建数据库

首先，我们可以通过在数据迁移目录中新建<date>-add-users-table.up.sql文件来创建一个model。文件中包含了如下内容:

```sql
CREATE TABLE users
(id VARCHAR(20) PRIMARY KEY,
 first_name VARCHAR(30),
 last_name VARCHAR(30),
 email VARCHAR(30),
 admin BOOLEAN,
 last_login TIME,
 is_active BOOLEAN,
 pass VARCHAR(100));
```

我们将users表修改为适合我们应用的名称:

```sql
CREATE TABLE guestbook
(id INTEGER PRIMARY KEY AUTO_INCREMENT,
 name VARCHAR(30),
 message VARCHAR(200),
 timestamp TIMESTAMP);
```
guestbook表将保存所有的信息描述。我们保存上面的sql，并通过在项目根目录运行如下的命令来创建数据库:

```sh
lein ragtime migrate
```

# 访问数据库

接着我们来看看src/guestbook/db/core.clj文件。我们发现里面已经有了数据库链接的相关配置.

```clojure
(ns guestbook.db.core
  (:require
    [yesql.core :refer [defqueries]]
    [clojure.java.io :as io]))

(def db-store (str (.getName (io/file ".")) "/site.db"))

(def db-spec
  {:classname   "org.h2.Driver"
   :subprotocol "h2"
   :subname     db-store
   :make-pool?  true
   :naming      {:keys   clojure.string/lower-case
                 :fields clojure.string/upper-case}})

(defqueries "sql/queries.sql" {:connection db-spec})
```
这个数据库将数据库文件保存在相对应用目录的site.db文件中。

在执行defqueries这个函数时，会自动生成执行数据库查询的函数，这些函数是与查询相对应的。相关的查询在sql/queries.sql文件中。

```sql

--name: create-user!
-- creates a new user record
INSERT INTO users
(id, first_name, last_name, email, pass)
VALUES (:id, :first_name, :last_name, :email, :pass)

--name: update-user!
-- update an existing user record
UPDATE users
SET first_name = :first_name, last_name = :last_name, email = :email
WHERE id = :id

-- name: get-user
-- retrieve a used given the id.
SELECT * FROM users
WHERE id = :id
```

可以看到，所有的函数名称都定义在--name:后面。接下来的一行注释定义了函数说明，最后是sql。sql参数使用:加变量名的形式来传递。

下面我们来添加自定义sql

```sql
--name:save-message!
-- creates a new message
INSERT INTO guestbook
(name, message, timestamp)
VALUES (:name, :message, :timestamp)

--name:get-messages
-- selects all available messages
SELECT * from guestbook
```

# 运行项目

```sh
lein ring server
guestbook started successfully...
2013-03-01 19:05:30.389:INFO:oejs.Server:jetty-7.6.1.v20120215
Started server on port 3000
2013-03-01 19:05:30.459:INFO:oejs.AbstractConnector:Started SelectChannelConnector@0.0.0.0:3000
```

浏览器会自动打开，你将能看到运行的应用。如果你不想浏览器自动打开，你可以用下面的命令来启动项目.

```sh
lein ring server-headless
```

你也可以自定义端口号，命令如下:

```sh
lein ring server-headless 8000
```

# 创建表单页面

应用的路由是定义在guestbook.routes.home命名空间下的。我们来打开它，并添加一些逻辑。首先，需要添加db命名空间以及Bouncer验证和ring.util.response.

```clojure
(ns guestbook.routes.home
  (:require
    ...
    [guestbook.db.core :as db]
    [bouncer.core :as b]
    [bouncer.validators :as v]
    [ring.util.response :refer [redirect]))
```

然后创建一个函数来验证form参数

```clojure
(defn validate-message [params]
  (first
    (b/validate
      params
      :name v/required
      :message [v/required [v/min-count 10]])))
```

上面的函数使用Bouncer根据我们编写的逻辑来验证:name和:message。在这里name为必填，message必须最少包含10个字符。可以通过嵌套vector来对一个字段构建多个验证。

我们添加一个函数来验证和保存信息：

```clojure
(defn save-message! [{:keys [params]}]
  (if-let [errors (validate-message params)]
    (-> (redirect "/")
        (assoc :flash (assoc params :errors errors)))
    (do
      (db/save-message! (assoc params :timestamp (java.util.Date.)))
      (redirect "/"))))
```

上面的函数通过:params从request中获取表单参数。当validate-message返回错误信息时将重定向到"/"，并将错误信息通过:flash来返回。否则保存信息。

接着，我们会修改home-page这个controller:

```clojure
(defn home-page [{:keys [flash]}]
  (layout/render
   "home.html"
   (merge {:messages (db/get-messages)}
          (select-keys flash [:name :message :errors]))
   :name (:name flash)))
```

我们所做的就是多传递了几个参数给模板，其中一个是从数据库中查询到的信息.

最后呢，在home-routes定义里面添加这个controller的路由定义。

```clojure
(defroutes home-routes
  (GET "/" request (home-page request))
  (POST "/" request (save-message! request))
  (GET "/about" [] (about-page)))
```

别忘了引入POST

```clojure
(ns guestbook.routes.home
  (:require ...
            [compojure.core :refer [defroutes GET POST]]
            ...))
```

controller已经编写OK。我们打开处于resources/templates下的home.html模板,目前只是简单的显示内容：

```html
{ % extends "templates/base.html" %}
{ % block content %}
 <div class="jumbotron">
    <h1>Welcome to guestbook</h1>
    <p>Time to start building your site!</p>
    <p><a class="btn btn-primary btn-lg" href="http://luminusweb.net">Learn more &raquo;</a></p>
 </div>

 <div class="row-fluid">
    <div class="span8">
    { {content|safe}}
    </div>
 </div>
{ % endblock %}
```

我们改为如下代码：

```html
{ % extends "templates/base.html" %}
{ % block content %}
 <div class="jumbotron">
    <h1>Welcome to guestbook</h1>
 </div>

 <div class="row-fluid">
    <div class="span8">
      <ul>
      { % for item in messages %}
        <li>
          <time>{ {item.timestamp|date:"yyyy-MM-dd HH:mm"}}</time>
          <p>{ {item.message}}</p>
          <p> - { {item.name}}</p>
        </li>
      { % endfor %}
      </ul>
    </div>
 </div>
{ % endblock %}

```

我们使用了迭代器来遍历信息。而每个迭代结果都是一个包含了信息的map。我们能通过名字来访问它们。同时，我们使用了一个日期过滤器来生成一个适于人类阅读的时间. 接着我们来添加错误信息的展示.

```html
{ % if error %}
    <p>{ {error}}</p>
{ % endif %}

```

我们只是简单的检查了一下是否有错误信息，如果有就展示。最后我们创建一个form来接受用户提交留言.

```html
<form action="/" method="POST">
    <p>
       Name:
       <input type="text" name="name" value={ {name}}>
    </p>
    <p>
       Message:
       <textarea rows="4" cols="50" name="message">
           { {message}}
       </textarea>
    </p>
    <input type="submit" value="comment">
</form>
```

最后，home.html看起来像这样

```html
{ % extends "guestbook/views/templates/base.html" %}

{ % block content %}
    <ul>
    { % for item in messages %}
      <li>
          <time>{ {item.timestamp|date:"yyyy-MM-dd HH:mm"}}</time>
          <p>{ {item.message}}</p>
          <p> - { {item.name}}</p>
      </li>
    { % endfor %}
    </ul>

{ % if error %}
    <p>{ {error}}</p>
{ % endif %}

<form action="/" method="POST">
    <p>
       Name:
       <input type="text" name="name" value={ {name}}>
    </p>
    <p>
       Message:
       <textarea rows="4" cols="50" name="message">
           { {message}}
       </textarea>
    </p>
    <input type="submit" value="comment">
</form>
{ % endblock %}

```

我们可以修改位于resources/public/css目录下的screen.css来使得页面更好看一些.

```css
body {
	height: 100%;
	padding-top: 70px;
	font: 14px 'Helvetica Neue', Helvetica, Arial, sans-serif;
	line-height: 1.4em;
	background: #eaeaea;
	color: #4d4d4d;
	width: 550px;
	margin: 0 auto;
	-webkit-font-smoothing: antialiased;
	-moz-font-smoothing: antialiased;
	-ms-font-smoothing: antialiased;
	-o-font-smoothing: antialiased;
	font-smoothing: antialiased;
}

input[type=submit] {
	margin: 0;
	padding: 0;
	border: 0;
  line-height: 1.4em;
	background: none;
	vertical-align: baseline;
}

input[type=submit], textarea {
	font-size: 24px;
	font-family: inherit;
	border: 0;
	padding: 6px;
	border: 1px solid #999;
	box-shadow: inset 0 -1px 5px 0 rgba(0, 0, 0, 0.2);
	-moz-box-sizing: border-box;
	-ms-box-sizing: border-box;
	-o-box-sizing: border-box;
	box-sizing: border-box;
}

input[type=submit]:hover {
	background: rgba(0, 0, 0, 0.15);
	box-shadow: 0 -1px 0 0 rgba(0, 0, 0, 0.3);
}

textarea {
	position: relative;
	line-height: 1em;
	width: 100%;
}

.error {
  font-weight: bold;
	color: red;
}

.jumbotron {
	position: relative;
	background: white;
	z-index: 2;
	border-top: 1px dotted #adadad;
}

h1 {
	width: 100%;
	font-size: 70px;
	font-weight: bold;
	text-align: center;
}

ul {
	margin: 0;
	padding: 0;
	list-style: none;
}

li {
	position: relative;
	font-size: 16px;
	padding: 5px;
	border-bottom: 1px dotted #ccc;
  box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.2),
	            0 25px 50px 0 rgba(0, 0, 0, 0.15);
}

li:last-child {
	border-bottom: none;
}

li time {
	font-size: 12px;
	padding-bottom: 20px;
}

form:before, .error:before {
	content: '';
	position: absolute;
	top: 0;
	right: 0;
	left: 0;
	height: 15px;
	border-bottom: 1px solid #6c615c;
	background: #8d7d77;
}

form, .error {
	width: 520px;
	padding: 30px;
	margin-bottom: 50px;
	background: #fff;
	border: 1px solid #ccc;
	position: relative;
	box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.2),
	            0 25px 50px 0 rgba(0, 0, 0, 0.15);
}

form input {
	width: 50%;
	clear: both;
}
```

现在刷新页面就可以看到我们修改的内容了。试试留个言！

# 打包应用

要打包程序，可输入

```sh
lein ring uberjar
```

这将会创建一个可运行的jar。通过下面的命令来运行

```sh
java -jar target/guestbook-0.1.0-SNAPSHOT-standalone.jar
```

如果我们想把应用部署到tomcat这样的服务器上，你可以运行

```sh
lein ring uberwar
```

这将会打包一个war包。

完整的源代码可以到[这里](https://github.com/yogthos/guestbook)下载。

