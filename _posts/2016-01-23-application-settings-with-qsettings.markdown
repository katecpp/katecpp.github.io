---
layout: post
title:  "Application settings with QSettings"
permalink: qsettings
date:   2016-01-23
category: cpp
tags: [qt, how-to]
---

Some of the application settings should be persistent between shutdowns and starts of the application. <a href="http://doc.qt.io/qt-4.8/qsettings.html" target="_blank">QSettings</a> class from Qt library is very easy and convenient way to achieve it with just a few lines of code.

The following example shows how to do it.
<ol>
<li><b>Set the application name, the organization  name and domain for your application.</b><br>
These names will be used to create unique application settings file. Apply the names to the QApplication object, like in the example below.

{% highlight cpp %}
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    a.setOrganizationName("MyOrganization");
    a.setOrganizationDomain("MyDomain");
    a.setApplicationName("MyAppName");

    MainWindow window;
    window.show();
    return a.exec();
}
{% endhighlight %}

This is not a necessary step, but if you skip it you will need to pass application name and organization name every time you call the QSettings constructor. With this step, you are able to call QSettings constructor with no parameters from everywhere inside your application code.
</li><br>
    
 <li><b>Define which parameters should be persistent.</b><br>
For example, in a minesweeper game, it's nice to store the user configuration of mine-board: the width and height and the mines count. These params can be encapsulated in a structure:

{% highlight cpp %}
struct SPreferences
{
    int32_t width;
    int32_t height;
    int32_t mines;
};
{% endhighlight %}

This structure can be a private member of MainWindow of application. When the game is (re)started, the values from that structure are used to create the new board.</li><br>

<li><b>Create loadSettings and saveSettings functions.</b><br>
Create a function loadSettings to run at the application startup and saveSettings to run before the application is closed.

{% highlight cpp %}
class MainWindow : public QMainWindow
{
    Q_OBJECT
    /* ... */
private:

    void loadSettings();
    void saveSettings();

    SPreferences m_prefs;
};
{% endhighlight %}

LoadSettings function loads saved settings or uses default values if none settings were stored. The settings are read with the usage of <code> QSettings::value(const QString & key, const QVariant & defaultValue)</code> method. In the following example, all default values are equal 10.

{% highlight cpp %}
void MainWindow::loadSettings()
{
  QSettings settings;
  m_prefs.width = settings.value("width", 10).toInt();
  m_prefs.height = settings.value("height", 10).toInt();
  m_prefs.mines = settings.value("mines", 10).toInt();
}
{% endhighlight %}

<b>Note: QSettings store the settings as the key-value pairs. The key is the QString defining the name of the variable (here: "height", "width" and "mines"). The value is stored as QVariant, that's why <i>.toInt()</i> call is necessary.</b><br><br>

Storing the settings with QSettings is also very straightforward. The saveSettings function should use <code>QSettings::setValue(const QString & key, const QVariant & value)</code> method:
{% highlight cpp %}
void MainWindow::saveSettings()
{
    QSettings settings;
    settings.setValue("width", m_prefs.width);
    settings.setValue("height", m_prefs.height);
    settings.setValue("mines", m_prefs.mines);
}
{% endhighlight %}

The saveSettings function should be called when the user decided to close the program. There is no point in saving them earlier; as long as the application runs, they can be changed multiple times. When the application is being closed we can be sure that the settings will not be changed in this lifecycle anymore. Overwrite the <i>closeEvent</i> for the MainWindow and call the saveSettings from that point:

{% highlight cpp %}
class MainWindow : public QMainWindow
{
    Q_OBJECT
public:
    void closeEvent(QCloseEvent *event);
private:
    void loadSettings();
    void saveSettings();

    SPreferences m_prefs;
};
{% endhighlight %}

{% highlight cpp %}
void MainWindow::closeEvent(QCloseEvent *event)
{
    saveSettings();
    event->accept();
}{% endhighlight %}

The job is done! This minimal example provided the whole implementation for basic use of QSettings in the application.
</li>
</ol>

### Where is the config file?
If you don't set the file location (what can be done with the use of special QSettings constructor), the default location is used. The default location depends on your OS (<a href="http://doc.qt.io/qt-4.8/qsettings.html#platform-specific-notes" target="_blank">Locations where application settings are stored</a>). You can also check the file location by calling `QSettings::fileName()` method.

### What's inside the config file?
As it was mentioned by the LoadSettings topic, the configuration file stores the keys as QStrings, and the values as QVariants. Let's take a look how it looks like in practice. The config file made in this example contains text:
{% highlight cpp %}
[General]
height=14
mines=10
width=14
{% endhighlight %}

### More info
For more detailed information, refer to <a href="http://doc.qt.io/qt-4.8/qsettings.html" target="_blank">Qt docs: QSettings</a>. If you want to see QSettings in action, you can check out my <a href="https://github.com/katecpp/sheep_sweeper" target="_blank">Minesweeper game on GitHub</a>. The examples in this post are coming from this repo.


