---
layout: post
title:  "Binary literals"
permalink: binary-literals
date:   2015-10-16
category: cpp
tags: [cpp14]
---
Until C++14, standard C++ allowed to define numbers in three ways:

{% highlight cpp %}
// decimal notation:
int number = 7;
// hexadecimal notation:
int number = 0x7;
// octal notation:
int number = 07;
{% endhighlight %}

Anyhow, without special compiler extensions or additional libraries it was not allowed to define numbers in binary format. It changes with C++14 -- the core language supports binary literals. 

Define integer with `0b` or `0B` prefix to represent binary number.

{% highlight cpp %}
int number = 0b0111;
{% endhighlight %}

It may be not the most crucial feature of C++ standard, but I believe there are cases when use of binary representation will improve readability of the code.

It's worth noticing that the GCC has offered the possibility to define binary numbers since <a href="https://gcc.gnu.org/gcc-4.3/changes.html">GCC 4.3</a> and <a href="http://theboostcpplibraries.com/boost.utility#ex.utility_06">Boost.Utility</a> offers `BOOST_BINARY` macro which also allows to create numbers in binary form.