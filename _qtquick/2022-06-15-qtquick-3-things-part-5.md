---
title: "Qt Quick Tutorial: 3 Things To Do List Part 5 - Adding a Database"
layout: post
date: 2022-06-15
published: true
cover: /assets/2022-06-15-add-database.png
excerpt: Add a C++ database.
---

Qt directly supports a local storage API which can be accessed via `QtQuick.LocalStorage` however we're going to construct our database functionality using C++ as that gives us more flexibility.

## Database design

Before we start writing any code we need to consider what information should be in the database and how best to present it.

For this app this will be a very simple structure, we have no dependencies and therefore only need a single table with the following columns:

- item name/details - VARCHAR
- today - BOOLEAN
- completed - BOOLEAN

## Add the files:

In order to switch our construction from the DataObject/DataModel construction we have to a SQL database and SQL Model we're going to need a need a way to construct the database (`dbhelper`) and a model derived from `QSqlTableModel` (`sqlmodel`).

Go ahead and create `dbhelper.h`, `dbhelper.cpp`, `sqlmodel.h` and `sqlmodel.cpp` then add them to `qt_add_executable` in `CMakeLists.txt` as you usually would:

```cmake
qt_add_executable(appThreeThings
    main.cpp
    dataobject.h
    dataobject.cpp
    datamodel.h
    datamodel.cpp
    filtermodel.h
    filtermodel.cpp
    dbhelper.h
    dbhelper.cpp
    sqlmodel.h
    sqlmodel.cpp
)
```

As we're adding a dependency to the project we're also going to need to add the Sql package to our project in `find_packages` and `target_link_libraries`:

```cmake
find_package(Qt6 6.2 COMPONENTS Quick Widgets Sql REQUIRED)

[...]

target_link_libraries(appThreeThings
    PRIVATE Qt6::Quick
    PRIVATE Qt6::Widgets
    PRIVATE Qt6::Sql)
```

## Database construction

Qt has a lot of built in support for databases meaning that you don't have to write a whole lot of code to perform every single function you need. Most of these are accessable using the [SQL Model Classes](https://doc.qt.io/qt-6/sql-model.html), we want to be able to read and write to our database and we don't have any Foreign Key Fields so we're going to derive our model from [QSqlTableModel](https://doc.qt.io/qt-6/qsqltablemodel.html). Before we can access the data though we need to create a database and connect to it. Because we don't need to write a whole lot of queries to make this work we're going to leverage `namespaces` for the construction/connection elements, initially we're going to pre-populate the database with data so we can check that our code is working properly.

Go ahead and open `dbhelper.h` and lets get our namespace set up:

```cpp
#include <QSqlDatabase>
#include <QSqlQuery>
#include <QSqlError>
#include <QDebug>
#include <QFile>

namespace dbhelper {
    QSqlDatabase connectDB(bool test = false);
    void removeDB();
}
```

As you can see this is a very short header which creates two namespace accessible functions, one to connect to the database and another to remove it. It would also be fine to create a class with static functions instead of using namespaces, however this is a more semantic way of creating helper functions.

In order to avoid having to edit our code again when we want to move over to using a real database we've given our `connectDB` function a `test` parameter which defaults to false.

Next in `dbhelper.cpp` we need to create two namespace declarations `namespace dbhelper` and within it an anonymous namespace, the latter will contain any functions we don't want to be accessible outside the namespace, similar to creating `private` methods in a class. We're also going to define two variables within the anonymous namespace `DATABASE_NAME` and `_db`.

The first is a constant containing our database name to avoid having to type it in repeatedly[^1]. This wants to be a string, however, if you define it as `QString` you will get a warning about it being a non-<abbr title="Plain Old Data">POD</abbr> global static. We can fix this by switching out our `QString` for an array of `char` which avoids the risk of it not being initialised correctly.


All `char` arrays in c++ terminate with a null character[^2] therefore our array needs to be initialised to string-length + 1.


Our second variable `_db` will have the same warning, however we can't avoid this one as easily. As we are definitely going to be using this variable, and we're not instanciating an entire libraries worth of static globals, we don't have to worry about compile time slowdowns. As it is reliant on `QSqlDatabase` there is a potential issue with it not getting initialised, however, it is a reasonably safe assumption that this will get constructed before it is used. In practice, this works just fine. So you can ignore the warning[^3].


```cpp
#include "dbhelper.h"

namespace dbhelper {
    namespace {
        const char DATABASE_NAME[15] = "threeThings.db";
        QSqlDatabase _db;
    }
}
```

In order to keep our code <abbr title="don't repeat yourself">DRY</abbr> we're going to create three functions within our anonymous namespace:

- openDatabase
- restoreDatabase
- insertData

Before we can do anything with our database we need to establish a connection using `QSqlDatabase`, in order to do this we first need to specify the driver we want to use by calling `::addDatabase`.

Once we've done that we can set our connection parameters. As we're using a local Sqlite database with no host, username or password most of these are unnecessary however we will need to set the database name.

Finally we need to call `open` in order to establish the connection.


```cpp
bool openDatabase(QString dbname) {
    _db = QSqlDatabase::addDatabase("QSQLITE");
    _db.setDatabaseName(dbname);

    return _db.open();
}
```

In the event that we don't have an existing database file (or that we're constructing an in memory database for testing purposes) we will need to create the table


```cpp
bool restoreDatabase() {
    QSqlQuery query;
    query.prepare("CREATE TABLE todoTable ("
            "id INTEGER PRIMARY KEY AUTOINCREMENT, "
            "item VARCHAR(255) NOT NULL, "
            "today BOOL DEFAULT FALSE, "
            "completed BOOL DEFAULT FALSE "
        "); ");

        if ( !query.exec() ) {
            qDebug() << "Database Error: failed to create table";
            qDebug() << query.lastError().text();
            return false;
        }
    return true;
}
```

For in memory test database construction we also want to insert some data

```cpp
void insertData() {
    QSqlQuery query;
    query.exec("insert into todoTable ( item, today, completed ) "
               "values (\"Do 30 minutes of exercise\", true, false)");
    query.exec("insert into todoTable ( item, today, completed ) "
               "values (\"Write blog post\", true, false)");
    query.exec("insert into todoTable ( item, today, completed ) "
               "values (\"Bake a cake\", true, false)");
    query.exec("insert into todoTable ( item, today, completed ) "
               "values (\"Get the car cleaned\", false, false)");
    query.exec("insert into todoTable ( item, today, completed ) "
               "values (\"Learn how to knit\", false, false)");
}
```

`connectDb()` and `removeDB()` will be defined within the outer namespace so that they are accessible from outside the file itself.

Setting the database name to `":memory:"` creates the database in memory rather than on the disc meaning that it will be destroyed when you close the application. This is great for testing but not so useful for data persistance.

```cpp
QSqlDatabase connectDB(bool test) {
    if (test) {
        if (!openDatabase(":memory:")) {
            qDebug() << "Database Error: failed to open test database";
        } else {
            restoreDatabase();
            insertData();
        }

    } else {
        if ( !QFile("./" + QString(DATABASE_NAME)).exists()) {
            if ( openDatabase(DATABASE_NAME) ) {
                restoreDatabase();
            }
        } else {
            openDatabase(DATABASE_NAME);
        }
    }

    return _db;
}
```

Removing the database is simply a case of ensuring the connection is closed before removing the file.

```cpp
void removeDB() {
    _db.close();
    if (!QFile("./" + QString(DATABASE_NAME)).remove()) {
        qDebug() << "Database Error: cannot remove database file";
    }
}
```


[^1]: It is also possible to use `#define` here, however in the main this should be avoided as constants created this way are untyped and therefore less safe. For more information about macros in C++ and `#define` directive gotchas see this [Tutorial on the C Preprocessor](https://www.cprogramming.com/tutorial/cpreprocessor.html)

[^2]: For more information on char arrays see [Dev C++ Tutorials: Character Array in C++](https://cpphinditutorials.com/dev-cpp/character-array-in-cpp/)

[^3]: If you want to dig into this warning further there are interesting discussions of [non-POD theory and practice](https://stackoverflow.com/questions/1538137/c-static-global-non-pod-theory-and-practice) as well as [when static c class members are initialised](https://stackoverflow.com/questions/1421671/when-are-static-c-class-members-initialized) on Stack Overflow

## Bibliography

1. [C++ namespaces with private members](https://www.internalpointers.com/post/c-namespaces-private-members)
2. [GitHub - hardcodes/QSqlTableModel_with_QML_readonly](https://github.com/hardcodes/QSqlTableModel_with_QML_readonly)
3. [Table Model Example \| Qt SQL 6.3.2](https://doc.qt.io/qt-6/qtsql-tablemodel-example.html)
4. [Using the SQL Model Classes \| Qt SQL 6.3.2](https://doc.qt.io/qt-6/sql-model.html#the-sql-table-model)
5. [Using C++ Models with Qt Quick Views \| Qt Quick 6.3.1](https://doc.qt.io/qt-6/qtquick-modelviewsdata-cppmodels.html#sql-models)