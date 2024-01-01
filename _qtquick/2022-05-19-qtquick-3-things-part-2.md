---
title: "Qt Quick Tutorial: 3 Things To Do List Part 2 - Adding Data"
date: 2022-05-19
published: true
excerpt: "Add tasks"
cover: "/assets/2022-05-19-part-2-cover.png"
---

At the end of [Part 1](qtquick/qt-quick-3-things-part-1) we had a UI that statically produced a list of 3 identical to do's. In order to have multiple to do items we're going to switch out the `Repeater` we've been using to display our `ToDoItem`'s for a `ListView` which requires either a `ListModel` or a `QObjectList` based model to back it up. While using a `ListModel` is undoubtedly easier when hard coding the data, as we will be doing initially, we will eventually be communicating with an SQLite database via C++[^qml-sqlite] so rather than construct the data twice, we're going to jump straight in to creating a model that derives from `QAbstractListModel` and hooking that up. We're using `QAbstractListModel` rather than `QObjectList` as the former can notify QML when the data changes.

As you can see from the screen grab above initially the DataModel will output all of our to do's regardless of whether they're marked as to do today or not, in [Part 3](/qtquick/qtquick-3-things-part-3) we will add a filter to display only the items we want.

## The dataobject

Create `dataobject.h` and `dataobject.cpp` then add them to `CMakeLists`:

```cmake
qt_add_executable(appThreeThings
     main.cpp
     dataobject.h
     dataobject.cpp
 )
```

### Header file

Open up `dataobject.h` and set up your  class with the following properties:

- item (string)
- today (boolean)
- completed (boolean)


For each of these you will need a way to read and set the content as well as a signal that the content has been changed.

Note that you cannot set these directly as the properties are stored in the private variables `_item`, `_today` and `_completed`. Although I favour `_var` for cleanliness `m_var` is common in C++, use whichever you like (or whichever the codebase you're working with uses).

```cpp
#ifndef DATAOBJECT_H
#define DATAOBJECT_H

#include <QObject>

class DataObject : public QObject {
    Q_OBJECT

    Q_PROPERTY(QString item READ item WRITE setItem NOTIFY itemChanged)
    Q_PROPERTY(bool today READ today WRITE setToday NOTIFY todayChanged)
    Q_PROPERTY(bool completed READ completed WRITE setCompleted NOTIFY completedChanged)

public:
    DataObject(QObject *parent=0);
    DataObject(const QString &item, const bool &today, const bool &completed, QObject *parent=0);

    QString item() const;
    void setItem(const QString &item);

    bool today() const;
    void setToday(const bool &today);

    bool completed() const;
    void setCompleted(const bool &today);

signals:
    void itemChanged();
    void todayChanged();
    void completedChanged();

private:
    QString _item;
    bool _today;
    bool _completed;
};
#endif
```

### CPP file

Now open up `dataobject.cpp` and add the following hopefully self explanatory code.

```cpp
#include <QDebug>
#include "dataobject.h"

DataObject::DataObject(QObject *parent)
    : QObject(parent) {
    //    empty
}

DataObject::DataObject(const QString &item, const bool &today, const bool &completed, QObject *parent)
    : QObject(parent), _item(item), _today(today), _completed(completed) {
    //  empty
}

QString DataObject::item() const {
    return _item;
}

void DataObject::setItem(const QString &item) {
    if ( item != _item ) {
        _item = item;
        emit itemChanged();
    }
}

bool DataObject::today() const {
    return _today;
}

void DataObject::setToday(const bool &today) {
    if (today != _today) {
        _today = today;
        emit todayChanged();
    }
}

bool DataObject::completed() const {
    return _completed;
}

void DataObject::setCompleted(const bool &completed) {
    if (completed != _completed) {
        _completed = completed;
        emit completedChanged();
    }
}
```

## The data model

Create `datamodel.h` and `datamodel.cpp` then add them to `CMakeLists` as before.

### Header file

Open up `datamodel.h` and add the following making sure you're referencing pointers for the `DataObject`'s throughout as they are a subclass of `QObject` and therefore have no copy constructors[^no-copy-constructors], attempts to pass the values directly will fail.

[^no-copy-constructors]: Because we created DataObject as a subclass of QObject we don't have a copy constructor therefore we need to pass our DataObjects around by pointer

```cpp
#ifndef DATAMODEL_H
#define DATAMODEL_H

#include <QAbstractListModel>
#include "dataobject.h"

class DataModel : public QAbstractListModel {
    Q_OBJECT

public:
    enum DataObjectRoles {
        ItemNameRole = Qt::UserRole + 1,
        TodayRole,
        CompletedRole,
        DeleteRole
    };

    DataModel(QObject *parent = 0);
    DataModel(DataObject* dataObject, QObject *parent = 0);
    DataModel(QList<DataObject*> dataObjects, QObject *parent = 0)
    ~DataModel();

    void addData(DataObject* dataObject);
    void addData(QList <DataObject*> &dataObjects);

    bool removeData(const QModelIndex &index);

    bool setData(const QModelIndex &index, const QVariant &value, int role);

    int rowCount(const QModelIndex &parent = QModelIndex()) const;

    QVariant data(const QModelIndex &index, int role = Qt::DisplayRole) const;

protected:
    QHash<int, QByteArray> roleNames() const;

private:
    QList<DataObject*> _dataObjects;
};

#endif
```

#### Walkthrough:

In order to be able to access `DataObject`'s properties we need to set up some custom roles for them and reimplement `roleNames`:

```cpp
public:
    enum DataObjectRoles {
        ItemNameRole = Qt::UserRole + 1,
        TodayRole,
        CompletedRole,
        DeleteRole,
    };

// ...

protected:
    QHash<int, QByteArray> roleNames() const;
```

We have created three constructors for our `DataModel` in order to provide a variety of ways of instantiating it: empty, with a single `DataObject` and with a list of `DataObject`s.

```cpp
DataModel(QObject *parent = 0);
DataModel(DataObject* dataObject, QObject *parent = 0);
DataModel(QList<DataObject*> dataObjects, QObject *parent = 0)
```

Similarly we have two `addDataObject`methods to allow adding individually or as a list:

```cpp
void addData(DataObject* dataObject);
void addData(QList <DataObject*> &dataObjects);
```

When you derive from `QAbstractListModel` you **have** to provide reimplimtations of `rowCount` and `data` and should provide a `headerData` implimentation which we have skipped. See <a href="https://doc.qt.io/qt-6.2/qabstractlistmodel.html">the docs</a> for more details.

### CPP

Hopefully the bodies of these functions should be self explanatory, our `addData` functions need to call `beginInsertRows` and `endInsertRows` in order to ensure all connected views are aware of the changes. Equally our `removeData` function needs to call `beginRemoveRows` and `endRemoveRows` and `setData` should `emit dataChanged`.

```cpp
#include "datamodel.h"

DataModel::DataModel(QObject *parent) : QAbstractListModel(parent) {
    // empty
}

DataModel::DataModel(DataObject* dataObject, QObject *parent)
    : QAbstractListModel(parent), _dataObjects({dataObject}) {
    // empty
}

DataModel::DataModel(QList<DataObject*> dataObjects, QObject *parent)
    : QAbstractListModel(parent), _dataObjects(dataObjects) {
    // empty
}

DataModel::~DataModel() {
    // empty
}

void DataModel::addData( DataObject* dataObject ) {
    beginInsertRows(QModelIndex(), rowCount(), rowCount());
    _dataObjects << dataObject;
    endInsertRows();
}

void DataModel::addData(QList <DataObject*> dataObjects ) {
    int endAt = rowCount() + dataObjects.count() - 1;

    beginInsertRows(QModelIndex(), rowCount(), endAt);
    for (int i = rowCount(); i < endAt; i++ ) {
        _dataObjects << dataObjects[i];
    }
    endInsertRows();
}

bool DataModel::removeData(const QModelIndex &index) {
    if (index.isValid()) {
        beginRemoveRows(QModelIndex(), index.row(), index.row());
        delete _dataObjects.takeAt(index.row());
        endRemoveRows();
        return true;
    }
    return false;
}

bool DataModel::setData(const QModelIndex &index, const QVariant &value, int role) {
    bool updated = false;

    if (index.isValid() && roleNames().contains(role)) {
        DataObject* d = _dataObjects[index.row()];
        if ( role == ItemNameRole && d->item() != value.toString()) {
            d->setitem(value.toString());
            updated = true;
        } else if ( role == TodayRole && d->today() != value.toBool()) {
            d->setToday(value.toBool());
            updated = true;
        } else if ( role == CompletedRole && d->completed() != value.toBool()) {
            d->setCompleted(value.toBool());
            updated = true;
        } else if ( role == DeletedRole) {
            updated = removeData(index);
        }
    }

    if (updated) {emit dataChanged(index, index, { role });}
    return updated;
}

int DataModel::rowCount(const QModelIndex & parent) const {
    Q_UNUSED(parent);
    return _dataObjects.count();
}

QVariant DataModel::data(const QModelIndex & index, int role) const {
    if (index.row() < 0 || index.row() >= _dataObjects.count()) {
        return QVariant();
    }

    DataObject* dataObject = _dataObjects[index.row()];
    if (role == ItemNameRole) {
        return dataObject->itemName();
    } else if (role == TodayRole) {
        return dataObject->today();
    } else if (role == CompletedRole) {
        return dataObject->completed();
    }
    return QVariant();
}

QHash<int, QByteArray> DataModel::roleNames() const {
    QHash<int, QByteArray> roles;
    roles[ItemNameRole] = "itemName";
    roles[TodayRole] = "today";
    roles[CompletedRole] = "completed";
    return roles;
}
```

### Add the dummy data to your to do list app:

Open `main.cpp` and add the following inside `main`:

```cpp
DataModel* dataList = new DataModel({
     new DataObject("Do 30 minutes of exercise", true, false),
     new DataObject("Write blog post", true, false),
     new DataObject("Bake a cake", true, false),
     new DataObject("Get the car cleaned", false, false),
     new DataObject("Learn how to knit", false, false),
 });
```

After `QQmlApplicationEngine engine;` add

```cpp
engine.setInitialProperties({ { "dataModel", QVariant::fromValue(dataList) } })
```

As we are using Qt Quick and therefore have a window object we cannot simply asign our dataList to `ListView`'s `model` parameter as is done in the [Qt Declarative Object List Model Example](https://doc.qt.io/qt-6.2/qtquick-models-objectlistmodel-example.html) so in order to access this we need to add a `property alias` to our `Window`. Note that I have not called the property `model`, this is to avoid potential namespace clashes, however it will need to be aliased to our toDoView's model property.

In `main.qml` add the following at the top of `Window`:

```qml
 property alias dataModel: toDoView.model
```

Now we're going to replace our `toDoColumnContainer` and everything inside it with a `ListView`. Delete this:

```qml
Item {
     id: toDoColumnContainer
     Layout.fillHeight: true
     Layout.minimumHeight: toDoColumn.implicitHeight
     Layout.fillWidth: true

     ColumnLayout {
         id: toDoColumn
         width: parent.width
         Repeater {
             id: toDoRepeater
             model: 3
             TodoItem {}
         }
     }
 }
```

Replace with this:

```qml
ListView {
     id: toDoView
     Layout.fillHeight: true
     Layout.fillWidth: true
     spacing: 8
     clip: true

     required model

     delegate: TodoItem {
                    width: toDoView.width - 16
                }
 }
```

This is a bit neater as the `ListView` is doing a lot of the heavy lifting for us. The `clip` property needs to be set to `true` here in order to stop the to do items from invading the space taken up by the `buttonArea`.

[^qml-sqlite]: It is eminently possible in QT to avoid writing C++ at all even to interact with a database: <https://www.qt.io/product/qt6/qml-book/ch13-storage-local-storage>


## References:

- [Using C++ Models with Qt Quick Views \| Qt Quick 6.2.4](https://doc.qt.io/qt-6.2/qtquick-modelviewsdata-cppmodels.html#qobjectlist-based-model)
- [QAbstractListModel Class \| Qt Core 6.2.4](https://doc.qt.io/qt-6.2/qabstractlistmodel.html)
- [Dynamic Views \| The Qt 6 Book](https://www.qt.io/product/qt6/qml-book/ch07-modelview-dynamic-views)
