---
title: "QT Quick Tutorial: 3 Things To Do List Part 6 - Linking the Database"
date: 2022-06-15
published: true
excerpt: "Link the database to the app."
cover: "/assets/2022-06-15-todo-list.png"
---

In order to use the database we constructed in [Part 5](/qtquick-3-things-part-5/) we need to replace our `DataModel` with an `SqlModel` based on `QSqlTableModel` which implements most of the functionality we have in `DataModel`.

Because we're using a database, the header file for `SqlModel` will be shorter than that of `DataModel` as we don't require the ability to populate the model, nor do we need to implement `rowCount()` or have a private variable to hold the list itself.

It is important to remember that we have an `id` field in our database which we didn't have in our `DataObject`.

```cpp
#ifndef SQLMODEL_H
#define SQLMODEL_H

#include <QSqlTableModel>
#include <QSqlDatabase>

class SqlModel : public QSqlTableModel {
    Q_OBJECT

public:
    enum Roles {
        IdRole = Qt::UserRole + 1,
        ItemRole,
        TodayRole,
        CompletedRole,
        DeleteRole,
    };

    explicit SqlModel(QObject *parent = nullptr, QSqlDatabase db = QSqlDatabase());
    ~SqlModel();

    Q_INVOKABLE void add();

    bool setData(const QModelIndex &index, const QVariant &value, int role);
    QVariant data(const QModelIndex &index, int role) const;

protected:
    QHash<int, QByteArray> roleNames() const;

};

#endif
```

The constructor and destructor for `SqlModel` are very similar to the basic constructor and destructor for `DataModel`, and `roleNames()` simply needs an `IdRole` added to it. Note that I have changed the `ItemNameRole` to `ItemRole`, if you choose to do the same don't forget to update your `ToDoItem.qml` as well.

```cpp
#include "sqlmodel.h"

SqlModel::SqlModel(QObject *parent, QSqlDatabase db) :
    QSqlTableModel(parent, db) {
//    empty
}

SqlModel::~SqlModel() {
//    empty
}

QHash<int, QByteArray> SqlModel::roleNames() const {
    QHash<int, QByteArray> roles;
    roles[IdRole] = "id";
    roles[ItemRole] = "item";
    roles[TodayRole] = "today";
    roles[CompletedRole] = "completed";
    roles[DeleteRole] = "remove";
    return roles;
}
```

In order to populate our view we first need to construct `data`.

As before we are checking to see that we have a valid `index` before attempting to access the data. We are then going to retrieve the relevant data using `QSqlDataModel::data()` to invoke the parent method.

The parent `data()` method requires a valid index and role to be passed to it, to get the index we need to combine row from `index.row()` as well as reversing the role enumeration in order to get the index of the field we are accessing. Since our roles are enumerated starting from `Qt::UserRole + 1` we can retrieve the original value by taking `Qt::UserRole + 1` away from our role value.

In order to return the data in a form that the filter model can cope with we need to check the `role` and return a `bool` for `today` and `completed` (OR we could change the filter model to check for `1`/`0` instead of `true`/`false`). The `switch` statement here leverages fall through[^1] to have both `today` and `completed` return `d.toBool()`. As we're only using this to change the values of today and completed to boolean this`switch` could be replaced with

```cpp
if (role == Today role || role == CompletedRole) {
    return d.toBool();
} else {
    return d;
}
```

I have chosen to use a `switch` as it allows for cleaner code should it become necessary to add more statements.

```cpp
QVariant SqlModel::data (const QModelIndex &index, int role) const {
    if ( !index.isValid() || index.row() > QSqlTableModel::rowCount() ) {
        return QVariant();
    } else if ( role <= Qt::UserRole ) {
        return QSqlTableModel::data(index, role);
    }

    QVariant d = QSqlTableModel::data(this->index(index.row(), role - Qt::UserRole - 1), Qt::DisplayRole);

    switch (role) {
        case TodayRole:
        case CompletedRole:
            return d.toBool();
        default:
            return d;
    }
}
```

Now if we update `main.cpp` to construct a database and use the model we've just created we will be able to see the data again.

In order to create the database we need to access the `connectDB()` function from within the `dbhelper` namespace, then we need to connect that to an instance of `SqlModel`.

Once we have our model instantiated we need to tell it which table we want to use with it and set the edit strategy before retrieving the data from the table using `select()`.

Now replace `dataModel` in `filterModel.setSourceModel()` with your new `SqlModel` instance and build the app.

```cpp
#include "filtermodel.h"
#include "sqlmodel.h"
#include "dbhelper.h"


int main(int argc, char *argv[]) {

    [...]

    QSqlDatabase db = dbhelper::connectDB(true);

    SqlModel* data(new SqlModel(&app, db));
    data->setTable("todoTable");
    data->setEditStrategy(QSqlTableModel::OnManualSubmit);
    data->select();

    FilterModel filterModel;
    filterModel.setSourceModel(data);
    filterModel.setShowTodayOnly(true);

    [...]
}
```

Once you've checked that everything is rendering correctly it's time to create the rest of the functionality in `sqlmodel.cpp`

This is where we really start to see the advantages of subclassing `QSqlTableModel` in our code as much of the heavy lifting is done for us. Particularly when it comes to adding or removing a record.

Don't forget that nothing is saved until you call `submitAll()`.

```cpp
void SqlModel::add() {
    this->insertRows(this->rowCount(), 1);
    this->submitAll();
}

bool SqlModel::setData(const QModelIndex &index, const QVariant &value, int role) {
    bool updated = false;

    if ( index.isValid() && roleNames().contains(role)) {
        if ( role == DeleteRole ) {
            this->removeRows(index.row(), 1);
            this->submitAll();
        } else if (value != this->data(index, role)) {
            QSqlTableModel::setData(this->index(index.row(), role - Qt::UserRole - 1), value);
            this->submitAll();
            updated = true;
        }
    }

    if (updated) emit dataChanged(index, index, { role });
    return updated;
}
```

Build again and everything should be back to working as it was before.

[^1]: Switch statements will continue to execute statements from the point the condition evaluates `true`until a break point is reached. This can be utilised effectively to have a switch statement run the same code for more than one condition. See <a href="https://en.cppreference.com/w/cpp/language/switch">CPP Reference: Switch Statement</a>.