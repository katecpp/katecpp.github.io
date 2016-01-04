---
layout: post
title:  "boost::optional"
categories: jekyll
---
## The need of optional values

Optional value is a value that can or can not exist. There is a plenty of real-life examples of optional values, like an object representing a person: it can have a car and an address -- but also it's possible to meet a person without a car or even homeless. The boost::optional (std::experimental::optional) library is designed for handling such situation in a nice way.

## Without optional

First consider how such optional values can be implemented without using any special library. Let's take for an example the Pool class that stores collection of objects Obj. The Pool needs the getLastObj() method, which should return the last handled object -- no problem if the collection is not empty, but what should this method do in case of empty collection?

### Do not call getLastObj() when the Pool is empty

Somebody can say that the Pool::isEmpty method should be implemented, and the getLastObj() should be only called if the Pool is not empty. Anyway the interfaces should be fault-tolerant and work correct uncoditionally. Some safe solution for getLastObj() should be implemented anyway.

### Return default value

{% highlight cpp %}
struct Obj
{
    Obj() : value(0) {}
    int value;
};

class Pool
{
public:
    Pool(){}

    Obj getLastObj()
    {
        if (!m_data.empty())
        {
            return m_data.back();
        }
        else
        {
            // what now?
            return Obj();
        }
    }

private:
    std::vector&amp;lt;Obj&amp;gt; m_data;
};

{% endhighlight %}

The above implementation returns the default-constructed Obj in case of empty collection. It makes sense only if the default-value is impossible for normal existing Object. 
[code language="cpp"]Pool pool;
Obj obj = pool.getLastObj();
std::cout &amp;lt;&amp;lt; &amp;quot;The last value is: &amp;quot; &amp;lt;&amp;lt; obj.value &amp;lt;&amp;lt; std::endl;
// Not exactly true...[/code]
The code above produces invalid output. It says that the last value was equal 0, and it was not because the Pool was empty. The code can be rescued by changing the default constructor of Obj, to be initialized with some ridiculous value -- maybe negative number?
[code language="cpp"]
struct Obj
{
    Obj() : value(-1) {}
    int value;
};

Pool pool;
Obj obj = pool.getLastObj();
if (obj.value != -1)
{
    std::cout &amp;lt;&amp;lt; &amp;quot;The last value is: &amp;quot; &amp;lt;&amp;lt; obj.value &amp;lt;&amp;lt; std::endl;
}
else
{
    std:: cout &amp;lt;&amp;lt; &amp;quot;No object found&amp;quot;;
}[/code]
This solution works only if the object has some value that should never occur in the real life. But what if the correct Obj.value can be positive, negative and is not limited to any range withing int?

<h2>Return additional bool</h2>

[code language="cpp"]bool getLastObj(Obj&amp;amp; obj)
{
    if (!m_data.empty())
    {
       obj = m_data.back();
       return true;
    }
    else
    {
       return false;
    }
}

Pool pool;
Obj obj;
bool retVal = pool.getLastObj(obj);
if (retVal)
{
    std::cout &amp;lt;&amp;lt; &amp;quot;The last value is: &amp;quot; &amp;lt;&amp;lt; obj.value &amp;lt;&amp;lt; std::endl;
}
else
{
    std:: cout &amp;lt;&amp;lt; &amp;quot;No object found&amp;quot; &amp;lt;&amp;lt; std::endl;
}
[/code]
This solution can be ok in some cases, but it's not perfect still. When the bool retValue is separated from the Obj (for example Obj is passed to other function, where the bool retValue is not accessible), there is no possible way to tell if the Obj was valid or not. So probably it will be better idea to have the information inside the Obj object (like it was in "Return impossible value" example). This effect can be achieved in two ways: add the bool member to Obj structure or handle with pointers to Obj (nullptr for not existing Obj). 

<h2>Use nullptr for not existing object</h2>

[code language="cpp"]class Pool
{
public:
    Pool(){}

    Obj* getLastObj()
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
    std::vector&amp;lt;Obj*&amp;gt; m_data;
};

Pool pool;
Obj* obj = pool.getLastObj();

if (obj != nullptr)
{
    std::cout &amp;lt;&amp;lt; obj-&amp;gt;value;
}
else
{
    std::cout &amp;lt;&amp;lt; &amp;quot;No object found&amp;quot; &amp;lt;&amp;lt; std::endl;
}[/code]
There is no possibility to mismatch nullptr with existing object, you can still use the full range of int <i>value</i> member and you didn't need to add additional member to Obj structure. Anyway, who would like to use the pointers in a case like that? It sounds like asking for trouble. Ok, so maybe finally let's take a look on boost::optional!

Boost::optional / std::experimental::optional
I prefer to use this library from std::experimental::optional. Only include
[code language="cpp"]#include &amp;lt;experimental/optional&amp;gt;[/code]
and you are ready to use it.




&nbsp;

&nbsp;

&nbsp;