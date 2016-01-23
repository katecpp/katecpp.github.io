---
layout: post
title:  "Sqlite with Qt - step by step"
permalink: sqlite-with-qt
date:   2015-08-28
category: cpp
tags: [qt, how-to]
---
Accessing SQL databases from C++ applications is very simple with Qt library. Here is some short example that presents how to do it. I have chosen <a href="https://www.sqlite.org/" target="_blank">SQLite</a> engine because it's the easiest engine to set up (it requires no server, no configuration...), still it's suitable for the most of possible applications.

### Before you begin - sql drivers

Qt requires drivers to deal with SQL databases. Depending on your distribution, you can have the drivers installed by default with your Qt or not. If you face problems like "QSqlDatabase: QSQLITE driver not loaded" you need to install drivers manually. Error should be followed by a console output that informs you which drivers you have installed: "QSqlDatabase: available drivers:". <a href="http://doc.qt.io/qt-4.8/sql-driver.html" target="_blank">Read more about sql drivers...</a>. 

## Design and create the database

Let's assume, for simplicity, that we need to store people data -- only name and corresponding id. With sqlite you can create such a simple database with two console commands. First line is creating database <em>people</em>. Second one is creating the table <em>people</em>. It cointains a key (integer) and name (text). To leave the sqlite console just type ".quit".

{% highlight bash %}
$ sqlite3 people.db
sqlite> CREATE TABLE people(ids integer primary key, name text);
sqlite> .quit
{% endhighlight %}

Congrats, your database is ready. Now we can create C++ application to play with it.

## Access database from Qt application

Create new Qt project. In .pro file you need to add:

`QT += sql`

It will link your project against QtSql module. If you don't use QtCreator but another IDE, for example Visual Studio, your project may need setting linker dependency to QtSql.lib. You need to do it if your compilation fails with unresolved external symbol error. In VS linker settings is located in `Properties/Configuration properties/Linker/Input`. In the Additional Dependencies add Qt5Sqld.lib for Debug and Qt5Sql.lib for release.

Now you can create a class responsible for database access. Let's call it DbManager.

{% highlight cpp %}
class DbManager
{
public:
    DbManager(const QString& path);
private:
    QSqlDatabase m_db;
};
{% endhighlight %}

The constructor of DbManager sets up the database connection and opens it. The path argument value is the path to your database file.

{% highlight cpp %}
DbManager::DbManager(const QString& path)
{
   m_db = QSqlDatabase::addDatabase("QSQLITE");
   m_db.setDatabaseName(path);

   if (!m_db.open())
   {
      qDebug() << "Error: connection with database failed";
   }
   else
   {
      qDebug() << "Database: connection ok";
   }
}
{% endhighlight %}

## Define needed operations

For managing people resources we need the possibility to:
<ul>
	<li>add new person,</li>
	<li>remove person,</li>
	<li>check if person exists in database,</li>
	<li>print all persons,</li>
	<li>delete all persons.</li>
</ul>

Actions can be implemented in following way: define QSqlQuery, then call <a href="http://doc.qt.io/qt-5/qsqlquery.html#prepare" target="_blank"><em>prepare</em></a> method. Prepare argument can be raw SQL query string, or it can contain placeholders for variables --- placeholders must begin with colon. After <em>prepare</em> call <em><a href="http://doc.qt.io/qt-5/qsqlquery.html#bindValue" target="_blank">bindValue</a></em> to fill the placeholders with proper values. When the query is ready, execute it.

{% highlight cpp %}
bool DbManager::addPerson(const QString& name)
{
   bool success = false;
   // you should check if args are ok first...
   QSqlQuery query;
   query.prepare("INSERT INTO people (name) VALUES (:name)");
   query.bindValue(":name", name);
   if(query.exec())
   {
       success = true;
   }
   else
   {
        qDebug() << "addPerson error:"
                 << query.lastError();
   }

   return success;
}
{% endhighlight %}

QSqlQuery::lastError() is very useful, you should not skip it in your implementation. 

Rules of creating queries are simple, so I think you already know how removing person looks like:

{% highlight cpp %}
if (personExists(name))
{
   QSqlQuery query;
   query.prepare("DELETE FROM people WHERE name = (:name)");
   query.bindValue(":name", name);
   success = query.exec();

   if(!success)
   {
       qDebug() << "removePerson error:" 
                << query.lastError();
   }
}
{% endhighlight %}

Print all persons:

{% highlight cpp %}
QSqlQuery query("SELECT * FROM people");
int idName = query.record().indexOf("name");
while (query.next())
{
   QString name = query.value(idName).toString();
   qDebug() << name;
}
{% endhighlight %}

Check if person with specific name exists:

{% highlight cpp %}
QSqlQuery query;
query.prepare("SELECT name FROM people WHERE name = (:name)");
query.bindValue(":name", name);

if (query.exec())
{
   if (query.next())
   {
      // it exists
   }
}
{% endhighlight %}

To remove all persons, just run:

{% highlight cpp %}
QSqlQuery query;
query.prepare("DELETE FROM people");
query.exec();
{% endhighlight %}

You can find the whole source code on my <a href="https://github.com/katecpp/sql_with_qt" target="_blank">github</a>. All you need to do before running this example is to create your SQLite database and write the valid path to it in main.cpp.
