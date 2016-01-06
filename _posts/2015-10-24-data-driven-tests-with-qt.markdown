---
layout: post
title:  "Data driven tests with Qt"
permalink: data-driven-tests-qt
date:   2015-10-24
category: cpp
tags: [tests, qt]
---
Qt provides functional and convenient framework for automate tests. It's easy to setup, since it does not require any third party libraries nor special configuration. It also contains special mechanisms for Qt specific testing, like signals and slots handling, anyway it's also good for non-Qt C++ project.

### Qt test Setup
The only one requirement is to have Qt installed. Then, open your test .pro file and add: `QT += testlib`. In the test .cpp file, include: `#include <QtTest>;`.

That's all. You are able now to write your QT test cases. Pretty simple!

### The need of data driven testing
Imagine you want to test a sorting function which takes as an argument the reference to vector of int and performs the sorting in place.

{% highlight cpp %}
void sort(std::vector<int>& data);
{% endhighlight %}

Of course it's necessary to test it with many inputs. You could start with writing following code:

{% highlight cpp %}
class SelectionSort_test : public QObject
{
    Q_OBJECT

public:
    SelectionSort_test();

private Q_SLOTS:

    void sort_test();
};
{% endhighlight %}

{% highlight cpp %}
void SelectionSort_test::sort_test()
{
    // C++11 extended initialization list
    vector<int> inputVector_1({5,8,9,2,0});
    vector<int> result_1({0,2,5,8,9});
    sort(inputVector_1);
    QCOMPARE(inputVector, result);

    vector<int> inputVector_2({0,1,0,1,0});
    vector<int> result_2({0,0,0,1,1});
    sort(inputVector_2);
    QCOMPARE(inputVector_2, result_2);

    vector<int> inputVector_3({9,8,7,6,5});
    vector<int> result_3({5,6,7,8,9});
    sort(inputVector_3);
    QCOMPARE(inputVector_3, result_3);
}
{% endhighlight %}

It's a bit repetitive and the code size grows very fast with adding new data sets. Besides, if you need to modify the test case, you need to do the change <code>n</code> times, what may lead to some mistakes.

That's where the data driven testing should be used. 

### Data driven tests with Qt
Data driven testing method allows you to create a test case with many various data sets without repeating the same code many times, or using loops.

To start with data driven testing, you need to define another private Q_SLOT with the name of the test function + "_data" postfix, like:

{% highlight cpp %}
private Q_SLOTS:
    void sort_test();
    void sort_test_data();
{% endhighlight %}

The <code>sort_test_data()</code> method contains the data passed to <code>sort_test()</code> method. The body of <code>sort_test_data()</code> needs following elements: first, use <code>QTest::addColumn</code> to define the parts of your data set. Here we need <code>input vector</code> of type vector of int and the <code>result</code> also of type vector of int. Then use <code>QTest::newRow</code> to fill the data sets with data; each row is separate data set.

{% highlight cpp %}
void SelectionSort_test::sort_test_data()
{
    QTest::addColumn<vector<int>>("inputVector");
    QTest::addColumn<vector<int>>("result");

    QTest::newRow("set 1")
            << vector<int>({5,8,9,2,0})
            << vector<int>({0,2,5,8,9});

    QTest::newRow("set 2")
            << std::vector<int>({0,1,0,1,0})
            << std::vector<int>({0,0,0,1,1});

    QTest::newRow("set 3")
            << std::vector<int>({9,8,7,6,5})
            << std::vector<int>({5,6,7,8,9});
}
{% endhighlight %}

After your data sets are prepared, you just need to load them into the test function and run the test. Use <code>QFETCH</code> macro to load the columns defined in <code>sort_test_data()</code> function. The syntax is:

{% highlight cpp %}QFETCH(data_type, column_name /* no quotes */ );{% endhighlight%}

This macro will create local variables with names equal to column names. Note that inside <code>QFETCH</code> macro you shall not use quotes around the column name. If you try to fetch data from not-existing column (wrong column name), the test will assert.

{% highlight cpp %}
void SelectionSort_test::sort_test()
{
    QFETCH(vector<int>, inputVector);
    QFETCH(vector<int>, result);

    sort(inputVector);
    QCOMPARE(inputVector, result);
}
{% endhighlight %}

This is how we created readable, easy to modify test with many various data sets. Above method will run QCOMPARE for every data set from <code>sort_test_data</code> function.

### Custom data types
If you want to run the Qt data driven tests with the custom types, you can receive an error: <em>Type is not registered, please use the Q_DECLARE_METATYPE macro to make it known to Qt's meta-object system</em>. The solution is already given in the message. You just need to add this macro into the header of your custom class. The macro should be outside the namespace, just like presented in below CFoo example:

{% highlight cpp %}
#ifndef CFOO_H
#define CFOO_H

#include <QMetaType>
#include <string>

namespace F
{
class CFoo
{
public:
// some declarations...
private:
    int m_number;
    std::string m_name;
};
}

Q_DECLARE_METATYPE(F::CFoo)
#endif // CFOO_H
{% endhighlight %}

### Test results
The results exactly show which check with which data set failed or succeeded:
<pre>PASS   : SelectionSort_test::sort_test(set 1)
PASS   : SelectionSort_test::sort_test(set 2)
PASS   : SelectionSort_test::sort_test(set 3)
Totals: 3 passed, 0 failed, 0 skipped, 0 blacklisted
********* Finished testing of SelectionSort_test *********</pre>
You can check out my <a href="https://github.com/katecpp/sorting_algorithms" target="_blank">github repository</a> to take a look on the project with QT tests. The applied project structure was created with help of <a href="http://dragly.org/2014/03/13/new-project-structure-for-projects-in-qt-creator-with-unit-tests/" target="_blank">dragly.org post (projects in QtCreator with unit tests)</a> and modified for my needs.