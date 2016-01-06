---
layout: post
title:  "Struct members order does make a difference"
permalink: struct-members-order
date:   2015-09-26
category: cpp
tags: [optimization, memory]
---
General modern computer address their memory in word-sized chunks, which mostly is 4-bytes <a href="https://en.wikipedia.org/wiki/Word_%28computer_architecture%29" target="_blank">word</a> for x86 architecture or 8-bytes word for x86-64 architecture. To maintain the higher performance, computer applies rules of <a href="https://en.wikipedia.org/wiki/Data_structure_alignment" target="_blank">data alignment</a>, which in practice means that some useless bytes can be added between variables so that the variables are accessible at addresses equal to multiple of the word size. The knowledge of that can be essential when designing structures for systems with strict memory requirements since it will allow you to reduce the size of struct.

## Random members order
Consider the following struct. It contains three types of data: uint8_t, uint32_t and uint64_t. There is not any visible order of how the members were placed in the structure; shortest members uint8_t are intertwining the longer ones.

{% highlight cpp %}
struct SRandomOrder
{
    uint8_t     m_byte1;
    uint64_t    m_long;
    uint8_t     m_byte2;
    uint32_t    m_int;
    uint8_t     m_byte3;
};
{% endhighlight %}

Since the members cannot be re-order by compiler, to achieve proper data alignment this struct will be placed in memory (x86 architecture) like below --- padding is presented as gray fields. The <code>sizeof(SRandomOrder)</code> is equal to 24 bytes. Note there is only 15 bytes of data and 9 bytes of padding!

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-struct/struct-memory-padding.png"><img src="{{ site.url }}/assets/post-struct/struct-memory-padding.png" alt="Structure with ineffective member placement" /></a><br>
<em>Structure with ineffective member placement</em>
</p>

### Structure packed
If the program has strict memory constraints it may be necessary to get rid of this redundant (from business logic point of view) padding. It can be done in two ways; first possibility is to pack the structure.

With GCC this can be done with <code>__attribute((packed))__</code>.

{% highlight cpp %}
struct SRandomOrderPacked
{
    uint8_t     m_byte1;
    uint64_t    m_long;
    uint8_t     m_byte2;
    uint32_t    m_int;
    uint8_t     m_byte3;
} __attribute((packed))__;
{% endhighlight %}

The packed structure takes the minimum of memory space. It's size is 15 bytes and not a single padding is added, but the data alignment rule is violated. The uint64_t member is placed in 3 chunks and the uint32_t is placed in 2 chunks. To load them from memory the computer will need to make more accesses than in previous case; additionally some bit-shifting operations will be performed to get the value of them. It may slow down the program execution on some architectures. On architectures with strict alignment requirements packing structure is impossible.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-struct/struct-packed.png"><img src="{{ site.url }}/assets/post-struct/struct-packed.png" alt="Structure packed" /></a><br>
<em>Structure packed</em>
</p>

### The best order of members
Another way to save some memory space is to declare members in decreasing order of size. It may remove the padding completely; in this case it's impossible and one byte of padding must be added anyway. As a result, the size of this structure is 16 bytes.

{% highlight cpp %}
struct SNiceOrder
{
    uint64_t    m_long;
    uint32_t    m_int;
    uint8_t     m_byte1;
    uint8_t     m_byte2;
    uint8_t     m_byte3;
};
{% endhighlight %}

In this case the natural data alignment has been preserved.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-struct/struct-properly-ordered.png"><img src="{{ site.url }}/assets/post-struct/struct-properly-ordered.png" alt="Structure with properly ordered members" /></a><br>
<em>Structure with properly ordered members</em>
</p>

## Summary
<ul>
	<li>It's best to order the struct members in decreasing or increasing order of size; it will minimize the required memory space and keep proper data alignment,</li>
	<li>if needed, the struct can be packed on some architectures, but it can affect performance negatively.</li>
</ul>