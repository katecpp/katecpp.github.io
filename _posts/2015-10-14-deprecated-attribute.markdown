---
layout: post
title:  "[[deprecated]] attribute"
permalink: deprecated-attribute
description: "Deprecated attribute in C++14 allows to mark methods, variables and other entities whose use is discouraged."
date:   2015-10-14
category: cpp
tags: [cpp14]
---
**Attribute deprecated** is a new feature of C++14 standard. It can be mostly useful for programmers who develop classes shared between various teams or projects.

When a method of some class becomes obsolete and this class is used only in its owners' team, it can be just removed and all the occurrences of this function call can be replaced with new valid method calls. This operation becomes unfulfillable when the function is a part of shared interface used by various teams. Deleting the obsolete method would be harmful in that case; we cannot know who, where and how often is calling that method and how long they would need to refactor their code to suit new interface. We want to allow them using the deprecated function.

Anyway, we would like to tell them somehow that they use the obsolete function and it's recommended to replace all the calls. And that's the purpose of attribute deprecated in C++14.

**Calling the method marked as deprecated will result in the compiler warning.** It's possible to provide a custom warning message.

### Example of usage

Simply place the [[deprecated]] attribute before the method declaration.

{% highlight cpp %}
class CSharedInterface
{
public:
    [[deprecated]]
    void run();
};
{% endhighlight %}

Calling the method <code>run()</code> will result in compiler warning:

> warning: 'void CSharedInterface::run()' is deprecated (declared at mainwindow.h:9) [-Wdeprecated-declarations]

The custom warning message can point to new recommended method.

{% highlight cpp %}
    [[deprecated("use start instead")]]
    void run();
{% endhighlight %}

Then the compiler gives warning:

> warning: 'void CSharedInterface::run()' is deprecated (declared at mainwindow.h:9): use start instead [-Wdeprecated-declarations]

Not only methods can be <code>deprecated</code>; it's also possible to mark classes, variables, typedefs, enums and other entities.

One of the readers proposed also another scenario when the deprecated attribute can be useful:

> This may have a hidden temporary usage: find all places where a function is being called when find references or refactoring tools sometimes don’t catch everything, and actually removing the declaration potentially causes a very long list of errors. - Cleroth Sun

Additional resources:

+ [ISO C++ committee proposal of deprecated attribute](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3760.html)
+ [Attributes specifier sequence on Cppreference](http://en.cppreference.com/w/cpp/language/attributes)
