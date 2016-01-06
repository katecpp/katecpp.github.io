---
layout: post
title:  "Final specifier"
permalink: final
date:   2015-08-22
category: cpp
tags: [inheritance, cpp11]
---
Besides of <a href="http://katecpp.github.io/override/">override specifier</a>, which helps with detecting not-really-overriding methods, C++11 introduces also another specifier helpful for working with inheritance: final specifier. This one can be included to function declaration or a class definition when you want to make sure that this method will not be overriden or the class will not be used as a base class.

## Usage of final method

Usage is very simple. Just add the specifier `final` after method declaration.

{% highlight cpp %}
class Base
{
    virtual void method() final;
};

class Derived : public Base
{
    void method(); // error!
};
{% endhighlight %}

Compiler will report an error, because it's illegal to override final functions.

### When to use it?

The introduction of final method into C++11 raises some questions. If somebody doesn't want to allow to override one function, why did he make it virtual? Wouldn't be better just to remove virtual keyword from the function in the most base class?

The answer is: no, because it's not always possible.

Imagine that you own a base class with some virtual methods you want to override. You do so in class Derived; and from this point, the method should not be overriden in any other class. With `final` specifier you can prevent the method from overriding starting with any class in hierarchy. It is shown in following example:

{% highlight cpp %}
class Base
{
    virtual void method();
};

class Derived : public Base
{
    void method() final;
};

class SecondDerived : public Derived
{
    void method(); // error!
};
{% endhighlight %}

The mechanism can be also used by optimization. Calling a virtual function is slower than calling not-virtual function. You can use final method to achieve devirtualization - if you call virtual method from the static object that defined this method as final, the compiler can devirtualize the call and the call has performance of not-virtual method call.

## Usage of final class

When a class is not designed to be a base class, add final specifier after its name. The compilation will fail if somebody will try to inherit such class.

{% highlight cpp %}
class NotBase final
{
};

class Derived : public NotBase // error!
{
};
{% endhighlight %}

### When it's useful?

Some classes are not intended to be inherited from, like std::vector or std::list, classes without virtual destructor or platform dependent classes.

## Summary

Final specifier is designed for quite specific scenarios. For sure you will not see it very often in the code - the usage is just very particular, in opposite to his sibling `override`, which should be used wherever possible.