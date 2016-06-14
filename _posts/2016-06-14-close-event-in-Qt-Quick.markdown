---
layout: post
title:  "Catchy close event in Qt Quick"
date:   2016-06-14
category: cpp
tags: [qt, qt-quick]
permalink: close-event-qtquick/
---

It's pretty common that the application may need to **clean/save some data before closing**. In Qt4 we've done this by the overriding `closeEvent` method, like this:
{% highlight cpp %}
class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    void closeEvent(QCloseEvent *event)
    {
        cleanUp();
        event->accept();
    }

};
{% endhighlight %}

In **Qt Quick** you can do it from QML file by putting desired actions into `ApplicationWindow.onClosing` method. Obviously, this method should be invoked when the application is closing: i.e. user clicked on close button on the title bar, or has chosen the **Exit** option from menu bar. And this **Exit** option appeared to be catchy for me today.

Every time I create a new Qt Quick project, my Qt Creator provides me an example implementation of the window with a menu bar. There is **File** menu that offers **Exit** action, which closes the application with `Qt.quit` call. My sad reflection was that the `ApplicationWindow.onClosing` handler is not invoked when the `Qt.quit` method is used! The window just closes without cleanup then. But everything is fine when the app is closed from the button from the title bar.

Here is simplified version of the code I used:

{% highlight cpp %}
ApplicationWindow {
    id: root
    onClosing: {
        cleanUp();
        console.log("Cleanup done, can close!");
    }

    menuBar: MenuBar {
        Menu {
            title: qsTr("&File")
            MenuItem {
                text: qsTr("E&xit")
                onTriggered: Qt.quit(); // no cleanUp!
            }
        }
    }
}
{% endhighlight %}

The solution is to replace the `Qt.quit` call with the `root.close` (or whatever is the window's id). Still it's interesting why the ApplicationWindow doesn't emit the `closing` signal when it's closed by the previous method. It may be a bug (there are similar bugs reported i.e. [this](https://bugreports.qt.io/browse/QTBUG-35390), but they apply to `close` method instead of `Qt.quit`), or it may be a feature. If you know what it is, feel free to let me know.

