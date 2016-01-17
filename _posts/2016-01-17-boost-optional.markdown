---
layout: post
title:  "boost::optional. What to do when you have no value"
permalink: boost-optional
date:   2016-01-17
category: cpp
tags: [boost]
---
Optional value is a value that can or can not exist. There are a plenty of examples of optional values: the person's middle name (not everybody has it) the minimum value of vector (which doesn't exist when the vector is empty), or the last unprocessed command from a queue.

The boost::optional library is designed for handling such situation in a nice way.

I will present the possible implementations to deal with optional values first without and then with the use of boost::optional. The class presented in the example is similar to one I've worked with in a real project. The `Pool` class is a container for objects `Object`, which can represent the commands that should be processed. The important `Object` from the `Pool` is always the last one, so the `Pool` interface should provide the method to get the last object in container: `getLastObject`. Because of the small size of `Object`, we will accept that it's returned by value.

{% highlight cpp %}
struct Object
{
    Object() : m_value(0) {}
    int m_value;
};
{% endhighlight %}

{% highlight cpp %}
class Pool
{
public:
    Pool(){}

    Object getLastObject()
    {
        return m_data.back(); // Oops! Vector can be empty.
    }

private:
    std::vector<Obj> m_data;
};
{% endhighlight %}

The problems are coming with the implementation of `getLastObject`. We need to ensure correct behaviour in case the vector is empty.
But what should be returned then?

## Without boost::optional

### Idea 1. Call `getLastObj` carefully [anti-pattern!]

Implement `Pool::isEmpty` method. Call `getLastObject` only if the `Pool` is not empty.
{% highlight cpp %}
if (!pool.isEmpty())
{
    object = pool.getLastObject();
}
else
{
    std::cout << "The pool was empty!"
}
{% endhighlight %}

If you think this is **the worst possible implementation**, you are right. The interfaces should be fault-tolerant. This isn't. It can result in undefined behaviour when you forget to make the proper check or somebody else try to use it without knowing the limitations. 

This implementation is a clear violation of the most important design guideline according to Scott Meyers: 

> Make interfaces easy to use correctly and hard to use incorrectly.

Don't look at this awful anti-pattern anymore and let's search for valid solutions.

### Idea 2. Return invalid-object value
If the collection is empty, return specific invalid-value for the `Object`.
{% highlight cpp %}
Object Pool::getLastObject()
{
    if (!m_data.empty())
    {
        return m_data.back();
    }
    else
    {
        return Object();
    }
}
{% endhighlight %}

It may be the default value or any other value (negative, extreme...), as long as it is impossible for normal existing Object. Otherwise, you won't be able to distinguish the fake-Objects from real Object that must be processed.

{% highlight cpp %}
Pool pool;
Object object = pool.getLastObject();
std::cout << "The last Object's value is: " << obj.m_value << std::endl;
// The last Object's value is: 0
// But wait, Pool was empty...
{% endhighlight %}

Such code can be misleading --- it's easy to forget to check if the value of `Object` is valid or not and treat the fake `Object` as a real one. Also, it's not always possible to use such implementation. Some objects require a full range of values and for them there is none specific *not-existing-object* value. 

There is also performance drawback of this approach -- we need to call the constructor although we don't really need this fake-Object. We made assumptions that the `Object` is small and the construction is cheap, anyway here the call is totally redundant.

### Idea 3. Return bool value
The return value can indicate whether the `Object` value is valid or not. The `Object` is given by reference to function.
{% highlight cpp %}
bool Pool::getLastObject(Object& object)
{
    if (!m_data.empty())
    {
       object = m_data.back();
       return true;
    }
    else
    {
       return false;
    }
}
{% endhighlight %}

{% highlight cpp %}
Pool pool;
Object object;
bool retVal = pool.getLastObject(object);
if (retVal)
{
    std::cout << "The last value is: " << object.m_value << std::endl;
}
else
{
    std:: cout << "The pool was empty!" << std::endl;
}
{% endhighlight %}

This solution can be sufficient in many cases, but it's not perfect still. When the bool `retValue` is separated from the `Object` (for example `Object` is passed to other function, where the bool `retValue` is not accessible), there is no possible way to tell if the `Object` was valid or not. Probably it will be better idea to have the information coupled the Object object. This effect can be achieved in few ways: use the `std::pair` of bool and Object, add the bool member to `Object` structure or handle with pointers to `Object`, where nullptr will stand for not existing value. 

### Idea 4. Use nullptr for not existing object
Instead of the vector of `Objects`, the `Pool` will store the vector of pointers to `Objects`.

{% highlight cpp %}
class Pool
{
public:
    Pool(){}

    Object* getLastObject()
    {
        if (!m_data.empty())
        {
            return m_data.back();
        }
        else
        {
            return nullptr;
        }
    }

private:
    std::vector<Object*> m_data;
};
{% endhighlight %}

{% highlight cpp %}
Pool pool;
Object* object = pool.getLastObject();

if (object != nullptr)
{
    std::cout << "The last value is: " << object->m_value << std::endl;
}
else
{
    std::cout << "The pool was empty!" << std::endl;
}
{% endhighlight %}

There is no possibility to mismatch nullptr with existing object, you can still use the full range of int <i>m_value</i> member and you didn't need to add the additional member to Object structure. The unnecessary object construction is also avoided. Anyway, using raw pointers sounds like asking for trouble. You need to take care of pointers deallocation and deal with null references. If you need more arguments why this approach should not be recommended, Sir Charles Antony Richard Hoare calling the null references "A billion dollar mistake" should convince you [(full presentation)][nullref].

> (...) I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years. - Sir Charles Antony Richard Hoare

A lot of various ideas, but all of them have drawbacks. Now let's take a look on boost::optional!

## With boost::optional

### Setup

To include optional from boost, add:
{% highlight cpp %}
#include <boost/optional/optional.hpp>
using boost::optional;
{% endhighlight %}
The library is also a part of std::experimental, so other possibility to include is:
{% highlight cpp %}
#include <experimental/optional>
using std::experimental::optional;
{% endhighlight %}

### Usage
Class template `boost::optional` is a  wrapper for object that can have or have not a valid value. The construction of the optional `Object` looks as follows:
{% highlight cpp %}
Object someObject;
optional<Object> object_initialized = someObject;
// copy constructor of `Object` invoked

optional<Object> object_uninitialized;
// doesn't call the `Object` constructor
{% endhighlight %}
For the uninitialized `optional<Object>` the `Object` constructor is not called.

You can check anytime whether the object is initialized or not. This is done by either `is_initialized` function or by simple putting the object into conditional expression. The `optional<Object>` will convert to `bool`.
{% highlight cpp %}
if (object) // equal to object.is_initialized
{
    // The object was initialized.
}
else
{
    // The object was not initialized.
}
{% endhighlight %}

The implementation of `Pool::getLastObject` with the use of `boost::optional` is presented below. The return type is `optional<Object>`. If the vector was not empty, the `optional<Object>` is initialized with proper value. Otherwise the empty object of type `optional<Object>` is returned.

{% highlight cpp %}
optional<Object> Pool::getLastObject()
{
    if (!m_data.empty())
    {
        return m_data.back();
    }
    else
    {
        return optional<Object>{};
    }
}
{% endhighlight %}

There is also another, shorter way to implement similar functions with the use of special two-arguments constructor. The first argument is a condition. The second argument is an initialization value to be used when the condition is true. When the condition is false the object stays uninitialized. 

{% highlight cpp %}
optional<Object> Pool::getLastObject()
{
    Object retValue = !m_data.empty() ? m_data.back() : Object();
    return optional<Object>{!m_data.empty(), retValue};
}
{% endhighlight %}

Now we can get the last object from Pool with the following code. The `m_value` is accessed from `optional<Object>` with the `->` operator (although it's not a pointer). 
{% highlight cpp %}
Pool pool;
optional<Object> object = pool.getLastObject();
if (object)
{
    std::cout << "The last value is: " << object->m_value << std::endl;
}
else
{
    std:: cout << "The pool was empty!" << std::endl;
}
{% endhighlight %}

You could also use `get()` method to receive from `optional<Object>` the instance of type `Object`.
{% highlight cpp %}
std::cout << "The last value is: " << object.get().m_value << std::endl;
{% endhighlight %}

If the object was uninitialized, calling the `get()` method or the `->`, `*` operators will result in the assertion.

### Summary
+ Boost.optional is designed for the values that can be initialized as well as uninitialized and both situations are normal,

+ Boost.optional doesn't set limitation on the possible values that the object can use,

+ Boost.optional is safer than the usage of nullptr for missing value,

+ Boost.optional doesn't call the wrapped type constructor for uninitialized values,

+ Boost.optional provides easily accessible and direct information about the object state (initialized/uninitialized),

+ Boost.optional makes your code good-looking, safe and easy to debug.

## References
* [Boost.org doc: optional][boostorg], 
* [The Boost C++ Libraries: boost.optional][theboostcpplib]

[boostorg]: http://www.boost.org/doc/libs/1_57_0/libs/optional/doc/html/index.html
[theboostcpplib]: http://theboostcpplibraries.com/boost.optional
[nullref]: http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare


