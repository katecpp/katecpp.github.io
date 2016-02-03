---
layout: post
title:  "Loop unrolling"
permalink: loop-unrolling/
date:   2015-08-14
category: cpp
tags: [optimization, loops]
---
Loop unrolling, also known as loop unwinding, is an optimization which can reduce overhead of running a loop --- number of instructions of checking the loop termination condition and loop counter modification. It requires fixed and known loop count.

For loop executes conditional statement e.g. <code>i < 0</code> and modifies the loop counter e.g <em>i++</em> once per iteration. It is shown in example below --- a simple loop with very small body. To call this body 100 times the loop also executes conditional statement and loop counter incrementation, both 100 times. It means 200 of 300 instructions of this loop are overhead.

{% highlight cpp %}
for(unsigned int i = 0; i < 100; i++)
{
   x[i] = i;
}
{% endhighlight %}

With a little modification the number of these overhead executions can be reduced from 100 to 25 each:

{% highlight cpp %}
for(unsigned int i = 0; i < 100; i += 4)
{
   x[i]   = i;
   x[i+1] = i + 1;
   x[i+2] = i + 2;
   x[i+3] = i + 3;
}
{% endhighlight %}

The number of copies inside loop body is called the loop unrolling factor. Be careful while choosing unrolling factor to not exceed the array bounds.

To ensure your loop is optimized use unsigned type for loop counter instead of signed type. On some compilers it is also better to make loop counter decrement and make termination condition as comparison to zero <a href="http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0205j/CJAJACCH.html">(ARM)</a>.

You can find loop unrolling useful while dealing with performance-critical parts of code which are executed very often. 

#### But... should you really do optimization by hand?

The answer is: it depends on your compiler abilities and of course your needs (it is really necessary to optimize this loop? You should check it with profiler before starting optimizations!). Such optimization done by developer has several disadvantages with the less readable code on the top. It is also easy to make mistake while manually unrolling bigger loop, and such mistake can be hard to spot. In case your compiler can do it for you, leave it to compiler -- it will be done better this way. On these terms your optimization may in fact decrease the performance and make your code less portable <a href="https://software.intel.com/en-us/articles/avoid-manual-loop-unrolling">(Intel)</a>.

On the other hand there are compilers that do not unroll loops and in that case you could do it manually; especially if the loop body is small. But first do some research with profiler if it's worth it.

<a href="http://www.linux-kongress.org/2009/slides/compiler_survey_felix_von_leitner.pdf">Felix von Leitner</a> summarized it shortly:
`Learn what your compiler does, Then let the compiler do it.`


GCC provides loop unrolling when flag -floop-optimize is enabled (which is true at levels -O, -O2, -O3, -Os).
