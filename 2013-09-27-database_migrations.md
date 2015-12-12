---
layout: post
title: Luminus手册-数据迁移
categories: luminus
tags: [clojure,luminus]
avatarimg: "/img/head.jpg"
author: Ivan

---

# Migrations

默认情况下Luminus使用Ragtime来进行数据迁移。当你在创建Luminus项目时添加了+mysql或+postgres选项时，Ragtime会自动被添加进来。

# Migrations with Ragtime

Ragtime通过Leiningen插件来执行。此插件需要配置project.clj的:plugins中。

```clojure
:plugins [... [ragtime/ragtime.lein "0.3.4"]]
```

实际的迁移动作是由ragtime.sql.files这个适配器来执行。此适配器需要配置在依赖列表中，同时配置相关数据库配置。

```clojure
:dependencies [... [ragtime/ragtime.sql.files "0.3.4"]]
:ragtime {:migrations ragtime.sql.files/migrations
          :database "jdbc:mysql://localhost:3306/example_db?user=root"}
```

ragtime.sql.files适配器会从项目根目录下的指定目录查找sql脚本。我们只需要将脚本添加到对应目录下即可。

脚本按字母顺序排序。我们来创建两个脚本。

```sql
migrations/2014-13-57-30-create-tables.up.sql

CREATE TABLE users (id INT, name VARCHAR(25));

migrations/2014-13-57-30-create-tables.down.sql

DROP TABLE users;
```

通过如下命令执行：

```sh
lein ragtime migrate
```

通过如下命令回滚:

```sh
lein ragtime rollback
```

<!-- more -->

# Migrations with Lobos

我们来看看如何在Luminus中集成Lobos.Lobos提供了功能强大的DSL来编
写迁移代码以及方便的对Korma进行扩展.

首先，我们来创建一个Luminus项目。

```sh
lein new luminus ltest +site
```

然后添加Lobos依赖。

```clojure
[lobos "1.0.0-beta1"]
```

接着我们需要在src目录下创建一个lobos目录，并在该目录下创建两个文件
config.clj和migrations.clj，其中包含如下内容:

```clojure
(ns lobos.config
  (:use lobos.connectivity)
  (:require [ltest.models.schema :as schema]))

(open-global schema/db-spec)
```

```clojure
(ns lobos.migrations
  (:refer-clojure
   :exclude [alter drop bigint boolean char double float time])
  (:use (lobos [migration :only [defmigration]] core schema config)))
```

我们来创建我们的第一个迁移，使用lobos来管理create-users-table函数.
你只需要将create-users-table函数从schema.clj移动到migations.clj中即可。

```clojure
(defmigration add-users-table
  (up [] (create
          (table :users
                 (varchar :id 20 :primary-key)
                 (varchar :first_name 30)
                 (varchar :last_name 30)
                 (varchar :email 30)
                 (boolean :admin)
                 (time    :last_login)
                 (boolean :is_active)
                 (varchar :pass 100))))
  (down [] (drop (table :users))))
```

如果想添加其他的迁移，只需要再添加一个类似的definition即可。
现在我们定义了迁移，下面我们来看看schema命名空间，我们来更新并使用它。
我们将添加lobos.migration依赖:

```clojure
(ns ltest.models.schema
  (:use [lobos.core :only (defcommand migrate)])
  (:require [noir.io :as io]
            [lobos.migration :as lm]))
```

接着定义一个command，返回给我们一个待迁移列表.

```clojure
(defcommand pending-migrations []
  (lm/pending-migrations db-spec sname))
```

根据这个信息我们可以知道数据库的状态。我们来重命名initialised?和
actualized?函数，当待迁移列表没有数据时返回true

```clojure
(defn actualized?
  "checks if there are no pending migrations"
  []
  (empty? (pending-migrations)))
```

我们还需要将create-tables函数替换为actualize函数，actualize函数将会触
发迁移。

```clojure
(def actualize migrate)
```

实际上actualize只是一个对migrate的引用，migrate是lobos.core中的函数.
最后我们需要做的就是更新handler.clj文件，来使用我们的新函数:

```clojure
(defn init
  "runs when the application starts and checks if the database
   schema exists, calls schema/create-tables if not."
  []
  (if-not (schema/actualized?)
    (schema/actualize)))
```

现在运行你的服务器，你的应用将会自动运行所有的待执行迁移。
希望这个教程对你在Luminus中使用数据迁移有帮助。相关代码请见[这里](https://github.com/edtsech/ltest)

# Popular Migrations Alternatives

下面是几个比较流行的数据库迁移工具:

-   Drift - Drift是一个使用Clojure编写的数据库迁移工具。Drift工作方式和Rails迁移方式类似-使用一个目录来存放你所有的迁移文件。Drift将会决定去运行哪个合适的文件。
-   Migratus - 一个通用的迁移框架，有一个数据库迁移的实现
-   Ragtime - Ragtime是一个迁移结构化数据的Clojure库。它定义了一组接口来处理迁移，就像Ring定义了一组接口来处理web应用一样。
-   Lobos - Lobos是一个使用Clojure编写的SQL数据库操作和迁移库.目前它提供对H2,MySQL,PostgreSQL,SQLite和SQL Server的支持.
