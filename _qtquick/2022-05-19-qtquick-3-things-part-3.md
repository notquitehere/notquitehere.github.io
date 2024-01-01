---
title: "Qt Quick Tutorial: 3 Things To Do List Part 3 - Filtering"
date: 2022-05-19
published: true
excerpt: Limit the "Today" view to only the three things we want to see.
cover: /assets/2022-05-19-part-3-cover.png
---

At the end of [Part 2](/qtquick/qtquick-3-things-part-2) we had a todo list that output our dummy data regardless of the value set in `today`. To fix that we need to write a filter model. Create `filtermodel.h` and `filtermodel.cpp` and add them to `CMakeLists` as before. In order to implement the filter we're going to derive from [`QSortFilterProxyModel`](https://doc.qt.io/qt-6/qsortfilterproxymodel.html#details).

## Header

For the moment this is only going to be a very small implementation as we are just using it to filter out the three tasks we have set for today from all the others and we can rely on the underlying functionality to do the bulk of the work for us. Any functionality we want to be accessible from QML needs to be declared `Q_INVOKABLE`.

```cpp
#ifndef FILTERMODEL_H
#define FILTERMODEL_H

#include <QSortFilterProxyModel>

class FilterModel : public QSortFilterProxyModel {
    Q_OBJECT

public:
    FilterModel(QObject *parent = nullptr);
    bool showTodayOnly();
    Q_INVOKABLE void setShowTodayOnly(const bool &today);

private:
    bool _showTodayOnly;
};

#endif
```

## CPP

In `setShowTodayOnly` we're using `setFilterRole` to isolate the data we want to filter on and using `setFilterFixedString` to ensure the whole string is matched, setting the string to `""` effectively clears it

```cpp
#include "datamodel.h"
#include "filtermodel.h"

FilterModel::FilterModel(QObject *parent) : QSortFilterProxyModel(parent) {
    // empty
}

bool FilterModel::showTodayOnly() {
    return _showTodayOnly;
}

void FilterModel::setShowTodayOnly(const bool &today) {
    _showTodayOnly = today;
    if (today) {
        this->setFilterRole(DataModel::TodayRole);
        this->setFilterFixedString("true");
    } else {
        this->setFilterFixedString("");
    }
}
```

## Main

### CPP

Back in `main.cpp` we need to instanciate our `FilterModel`, set the source model and tell it to `showTodayOnly` as the default state:

```cpp
FilterModel filterModel;
filterModel.setSourceModel(dataList);
filterModel.setShowTodayOnly(true);
```

Then we need to remove

```cpp
engine.setInitialProperties({ { "dataModel", QVariant::fromValue(dataModel) } });
```

and instead load our `filterModel` into the root context so that all our views can access it:

```cpp
QQmlContext *context = engine.rootContext();
context->setContextProperty("dataModel", &filterModel);
```

### QML

As we now have our model loaded into the root context we no longer need to alias `dataModel` to `toDoView.model` so we can remove this line:

```qml
property alias dataModel: toDoView.model
```

Then change `require model` in `toDoView` to `model: dataModel`.

Now when you run the app you will only get the items marked `true` for today showing:

![](/assets/2022-05-19-part-3-to-do-today.png)

That's great but we need a way to see all of our todo's so lets make that See all button at the bottom work. To do that we need to add a state to our `toDoContainer`, both so we know what state we're currently in and so we can change other things within the view. Here we're changing the `titleText` and the `changeView` text. The default state is referred to by an empty string `""` and isn't required to be explicitly included here.

```qml
states: [
    State {
        name: ""
    },

    State {
        name: "viewAll"
        PropertyChanges {
            target: titleText
            text: "All the To Do Items"
        }

        PropertyChanges {
            target: txt
            text: "See Today"
        }
    }
]
```

Now we need a way to invoke our `viewAll` state, to do that we're going to add a `MouseArea` to our `CustomButton`and have it trigger the state change and filter when we click on it by setting up a `buttonClicked` signal that we can use to pass the function to the `onClicked` signal.

```qml
Rectangle {
    property alias txt: txt
    signal buttonClicked()

    ...

    MouseArea {
        anchors.fill: parent
        onClicked: buttonClicked()
    }

}
```

Back in `main.qml` we need to link that `buttonClicked` signal to the action we want to take:

```qml
CustomButton {
    id: changeView
    anchors.centerIn: parent
    color: "#aa5c2b80"
    txt.text: "See all"
    txt.color: "#fafafa"
    txt.padding: 16
    onButtonClicked: {
        if (root.state === "viewAll") {
            root.state = ""
            dataModel.setShowTodayOnly(true);
        } else {
            root.state = "viewAll"
            dataModel.setShowTodayOnly(false);
        }
    }
}
```

Now if you run the app you can switch between the two views using the purple button.

![Screen shot of the app showing today and all screens](/assets/2022-05-19-part-3-cover.png)

## References

- [Custom Sort/Filter Model Example \| Qt Widgets 6.3.0](https://doc.qt.io/qt-6/qtwidgets-itemviews-customsortfiltermodel-example.html)
- [QSortFilterProxyModel Class \| Qt Core 6.3.0](https://doc.qt.io/qt-6/qsortfilterproxymodel.html)
- [QML List View Sort and Filter](https://arunpkqt.wordpress.com/2016/12/08/qml-list-view-sort-and-filter/)
- [[Solved] Custom Component [propery alias VS ..]](https://forum.qt.io/topic/1728/solved-custom-component-propery-alias-vs)