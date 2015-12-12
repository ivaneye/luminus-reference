---
layout: post
title: Luminus手册-访问数据库
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

Configuring the Database
========================

当你使用数据库参数来创建Luminus工程时，比如+postgress，那么Luminus会使用Korma来处理数据库操作.

Korma是一个基于Clojure的数据库操作DSL。Korma提供了一套简单的接口，方便你来处理复杂的数据。

给一个现有的项目添加数据库支持，非常的简单。首先，你需要在project.clj文件中添加Korma依赖。

```clojure
[korma "0.4.0"]
```

另外还需要相应数据库的驱动，所以你还需要在project.clj中添加相关的数据库驱动。比如说，你要连接PostreSQL数据库，你需要添加如下的依赖.

```clojure
[postgresql/postgresql "9.3-1102-jdbc41"]
```

添加完依赖后，你可以创建一个新的namespace来管理你的model,这个namespace建议叫db.core。你需要在里面引入korma.db。

```clojure
(ns myapp.db.core
  (:use korma.core
        [korma.db :only (defdb)]))
```

Setting up the database connection
----------------------------------

配置完依赖后，我们要做的事情就是定义数据库连接。我们只需要提供一个包含连接信息的map。

```clojure
(def db-spec {:subprotocol "postgresql"
         :subname "//localhost/my_website"
         :user "admin"
         :password "admin"})
```

或者也可以配置服务器提供的JNDI名称来创建连接。

```clojure
(def db-spec {:name "jdbc/myDatasource"})
```

<!-- more -->

这种方式适合当你有多个连接的时候。比如说，你有个dev/staging/production
服务器，你可以将JNDI连接指向它所配置的数据库。应用程序将会从环境中来加
载数据库连接信息。这意味着，你可以随意的改变你的数据库连接，而不需要修
改你的代码。 最后，你能自己配置JDBC连接:

```clojure
(def db-spec
  {:datasource
    (doto (new PGPoolingDataSource)
     (.setServerName   "localhost")
     (.setDatabaseName "my_website")
     (.setUser         "admin")
     (.setPassword     "admin")
     (.setMaxConnections 10))})
```

Creating tables
---------------

Korma依赖clojure.java.jdbc.这个库提供操作表的功能。

你可以使用create-table函数来从应用中创建数据表。

```clojure
(defn create-users-table []
  (sql/db-do-commands db-spec
    (sql/create-table-ddl
      :users
      [:id "varchar(32)"]
      [:pass "varchar(100)"])))
```

create-table-ddl函数需要包含在db-do-commands内，db-do-commands保证了数据库连接的关闭。

Accessing the Database
======================

当使用Korma时，你需要先用defdb来包裹db-spec。

```clojure
(defdb db schema/db-spec)
```

这将会使用c3p0来创建一个连接池。需要注意的是，最后创建的连接池会被设为默认的连接池。

Korma使用entities来表示sql表。这个entities构成了你查询的核心。

entities使用defentity宏来创建:

```clojure
(defentity users)
```

我们可以这样来创建user:

```clojure
(defn create-user [user]
  (insert users
          (values user)))
```

而如果我们想查询user，我们可以这样写:

```clojure
(defn get-user [id]
  (first (select users
                 (where {:id id})
                 (limit 1))))
```

详细文档可参考[Korma官网](http://sqlkorma.com/docs)

# Yesql

Korma提供了操作SQL的DSL，而Yesql可以直接操作SQL.

如果要使用Yesql你需要添加依赖：

```clojure
[yesql "0.4.0"]
```

当你在classpath下创建一个包含查询的SQL文件，比如resources/queries.sql.文件格式如下:

```sql
    -- name: find-users
    -- Find the users with the given ID(s).
    SELECT *
    FROM user
    WHERE user_id IN (:id)
    AND age > :min_age

    -- name: user-count
    -- Counts all the users.
    SELECT count(*) AS count
    FROM user
```

创建问上述文件后，你需要在需要的地方引入yesql.core/defqueries。

```clojure
(ns myapp.db.core
  (:require [yesql.core :refer [defqueries]]))

(defqueries "resources/queries.sql")
```

每个查询可以像函数一样通过名字来调用。

```clojure
(find-users db-spec [1001 1003 1005] 18)
```
