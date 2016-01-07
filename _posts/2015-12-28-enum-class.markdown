---
layout: post
title:  "Enum class. Why should you care?"
permalink: enum-class
date:   2015-12-28
category: cpp
tags: [cpp11]
---
The usage of enums is pretty common and it might seem that this language feature is well-known to everybody. And yet, despite the fact that the C++11 standard has almost 4 years, some people are stuck with old style enum instead of using the enum class version. In this article I will discuss the differences between C++98 enum and C++11 enum class and prove that the latter one is almost always better idea.

## C++98 unscoped enum
C++98 style-enum is unscoped, which means that the names of enumerators are available in global scope. It results in global namespace pollution and the growing possibility of the name conflict. For example, consider an enum which defines the message type. See what happens if you define somewhere a variable called <i>error</i>. Will you be able to spot the bug in a big program?

{% highlight cpp %}
enum MsgType
{
    request,
    ack,
    error
};

int process()
{
   int error = 0;
   /* ... */
   MsgType currentMsgType = getType();
   if (currentMsgType == error){ /* ... */}
}
{% endhighlight %}

To avoid such mistakes the names of enumerator should be more unique, e.g. begin with the enum type prefix, like <i>MsgType_request</i>. You could also wrap the enum with struct or class and refer to enum values with the class name specified. Other (and possibly better) option is using C++11 scoped enum.

## C++11 scoped enum

{% highlight cpp %}
enum class MsgType
{
    request,
    ack,
    error
};
{% endhighlight %}

Such enum is called scoped enum, because the names of enumerators are not known to global scope. To use them you need to specify the enum class name and use the scope resolution operator <b>::</b>.

{% highlight cpp %}
int process()
{
   int error = 0;
   /* ... */
   if (currentMsgType == MsgType::error){ /* ... */}
}
{% endhighlight %}

Now the risk of name conflict with enumerators reached zero level. The enum class name is required also by creation and initialization of enum values.

{% highlight cpp %}
   MsgType currentMsgType = MsgType::request;
   MsgType previousMsgType(MsgType::ack);
{% endhighlight %}

This also allows you to create many enum classes with equal enumerators names. Such code is totally legal and safe --- the names are not conflicting with each other since you use them with enum class name explicitly.

{% highlight cpp %}
enum class MsgType
{ request, ack, error };
enum class ActionType
{ request, ack, error };
enum class Status
{ on, off, error };
{% endhighlight %}

### Implicit conversions. Is it what you want?
There is also another big difference between C++98 enum and C++11 enum class: unscoped enums are implicitly convertible to integers, while the latter one are not. Code below (C++98 enum) would compile with no warnings, but does it have sense in all cases? Maybe you would like to have some compiler tip, that this part of code looks suspicious?

{% highlight cpp %}
MsgType crtMsg = request;

double someResult = crtMsg/2;
double anotherResult = sin(crtMsg);
bool correctSize = (crtMsg < 126.5);
{% endhighlight %}

The same code, for C++11 enum class will not compile:

{% highlight cpp %}
MsgType crtMsg = MsgType::request;

double someResult = crtMsg/2;
// error! no match for operator /
double anotherResult = sin(crtMsg);
// error! cannot convert MsgType to double
bool correctSize = (crtMsg < 126.5);
// error! no match for operator <
{% endhighlight %}

To compile the code you need to explicitly cast the enum to int with static_cast. That will allow you to spot some bugs at early stage.

### Forward declaration of enum
The advantages of forward class declaration instead of including headers are widely known: the compile time is shorter, the translation unit is not so polluted with redundant symbols, the unnecessary recompilation of sources when the header changes can be avoided. If it so great, then why not use it for enums? Let's try it with C++98.

{% highlight cpp %}
enum MsgType; // forward declaration

class C
{
public:
    bool process(MsgType msg);
};
{% endhighlight %}

We added the forward declaration in header of the C class and the proper include in source file. Anyway, something is wrong --- the code does not compile. The compiler says: <code>error: use of enum 'MsgType' without previous declaration
enum MsgType;</code>

It's because the underlying type of enum is not known. The compiler needs to know how many bytes will be required to store the enum and it cannot be decided based on the forward declaration like above. It's necessary to see the biggest value stored inside this enum. The presented enum <code>MsgType</code> could be stored on 1 byte --- the biggest value to store (error) is equal to 2. However it's totally up to compiler how many bytes will be used. My compiler decided to use 4 bytes for that.

But does it mean that the forward declaration of enum is not possible? Let's try the same with enum class. Now it works! We just discovered another difference between C++98 enum and C++11 enum class. The underlying type of enum class is implicitly known by the compiler (it's int by default).

However, I must mention that for C++98 enum the forward declaration is also possible. It's only required to specify underlying type manually:

{% highlight cpp %}
enum MsgType : int; // forward declaration

class C
{
public:
    bool process(MsgType msg);
};
{% endhighlight %}

In conclusion, the forward declaration is possible for both C++98 and C++11 enums, but for enum classes it's a little bit easier.
### Enum as a struct member
Another consequence of unknown underlying type appears when the C++98 enum should be a struct member. The struct size can be different when the code is compiled on various machines!

## Summary
<ul>
	<li>C++11 enum class does not pollute namespace like C++98 enums do,</li>
	<li>C++11 enum class prevents implicit conversions which provides you more safety,</li>
	<li>C++11 enum class size is defined (int by default),</li>
	<li>C++11 enum class forward declaration is easier.</li>
</ul>