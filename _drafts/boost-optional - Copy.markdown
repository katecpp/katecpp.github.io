---
layout: post
title:  "Types of inheritance in C++"
categories: cpp
tags: [inheritance]
---
C++ standard supports three types of inheritance: public, protected and private. One of the main differences is the accessibility of the public and protected members -- their accessibility is narrowed to the type of inheritance. Private members are always not accessible from derived class. The code below presents the members scope depending on the inheritance type.

{% highlight cpp %}
class A 
{
public:
    int publicMember;
protected:
    int protectedMember;
private:
    int privateMember;
};

class B : public A
{
    // publicMember is public
    // protectedMember is protected
    // privateMember is not accessible from B
};

class C : protected A
{
    // publicMember is protected
    // protectedMember is protected
    // privateMember is not accessible from C
};

class D : private A
{
    // publicMember is private
    // protectedMember is private
    // privateMember is not accessible from D
};
{% endhighlight %}


## Public inheritance
Usage of public inheritance is pretty common and means *is-a* relationship. The derived class can be always used when the base class is needed. In the example below, the DerivedPublic class object is passed to the function which awaits Base class and everything is fine.
{% highlight cpp %}
class Base
{
};

class DerivedPublic: public Base 
{
};

void doSmthOnBase(const Base& base){}

DerivedPublic d;
doSmthOnBase(d); // great!
{% endhighlight %}

## Private inheritance
Would you be able to do the same trick when the derived class inherits privately from Base? Unfortunately **not**. Such piece of code will not compile:
{% highlight cpp %}
class Base
{
};

class DerivedPrivate: private Base 
{
};

void doSmthOnBase(const Base& base){}

DerivedPrivate d;
doSmthOnBase(d); // error !
{% endhighlight %}

The compiler will complain about error: <em>'Base' is an inaccessible base of 'Derived' doSmthOnBase(d)</em>. 
That means that, unlike public inheritance, private inheritance is not an "is-a" relationship.

## So what's the use of private inheritance?
Private inheritance means "has-a" relationship (or: "is implemented in terms of"). Similar to composition, you should use private inheritance when you want to make use of the implementation details of base class, but you do not want to inherit the interface.

## When private inheritance is better than composition?
You will have to use private inheritance when you need to have access to protected members/member functions of base class. In other cases almost always you should use composition. Private inheritance will make your code harder to maintain and increase the compilation dependencies (in case of composition, you can use forward declaration and a member-pointer to object).

## Empty Base Optimization (EBO)
There is one more specific case when private inheritance can be reasonable: so called Empty Base Optimization. Imagine EmptyClass with zero non-static members and zero virtual functions nor base classes. Composition of EmptyClas will increase the size of OtherClass_composition.

{% highlight cpp %}
class EmptyClass{};
class OtherClass_composition 
{ 
    private: 
        EmptyClass m_empty; 
        int m_member;
};
{% endhighlight %}

But if you use private inheritance you can save some space.

{% highlight cpp %}class OtherClass_inheritance : private EmptyClass
{   
    private: 
        int m_member;
};
{% endhighlight %}

In this case, the size of OtherClass_inheritance should not increase (but it's dependant on the compiler). With the use of MinGW 4.9.1 the size of OtherClass_composition was 8, while the size of OtherClass_inheritance was 4.

Further readings:

+ <a href="https://isocpp.org/wiki/faq/private-inheritance" target="_blank">Private inheritance on isocpp,</a>

+ <a href="https://en.wikipedia.org/wiki/Composition_over_inheritance" target="_blank">Composition over inheritance rule.</a>
