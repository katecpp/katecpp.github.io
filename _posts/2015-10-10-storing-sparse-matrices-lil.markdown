---
layout: post
title:  "Storing sparse matrix – list of lists LIL"
permalink: sparse-matrices-lil/
date:   2015-10-10
category: cpp
tags: [optimization, memory]
---
Usually matrices in C++ programs are represented as two-dimensional arrays. Memory requirement of such array is proportional to <code>m×n</code>, where <code>m</code> and <code>n</code> are the height and width of this array.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-sparse-lil/dense-matrix.png"><img src="{{ site.url }}/assets/post-sparse-lil/dense-matrix.png" alt="Almost all of dense matrix values are significiant" /></a><br>
<em>Dense matrix</em>
</p>

Things look differently if the matrix we need to store is a sparse matrix. Sparse matrix is a matrix which most elements have default value (often zero) and only few elements have other value. Storing such matrix in two-dimensional array would be a big waste of memory space, especially when it is large-sized and the sparsity level is high. Such array can be stored in reduced space with no information loss and with only slight deterioration in performance.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-sparse-lil/sparse-matrix.png"><img src="{{ site.url }}/assets/post-sparse-lil/sparse-matrix.png" alt="Example of sparse matrix" /></a><br>
<em>Sparse matrix</em>
</p>

## List of lists: Efficient representation of sparse array

One of possible approaches to store sparse array is a list of lists (LIL) method. The whole array is represented as list of rows and each row is represented as list of pairs: position (index) in the row and the value. Only elements with value other than default (zero in presented case) are stored. For the best performance the lists should be sorted in order of ascending keys.

Let's consider one-dimensional array for simplicity. It contains a huge amount of elements, but only the initial ones are presented below. Three elements are of non-zero value.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-sparse-lil/diagram-sparse-matrix1.png"><img src="{{ site.url }}/assets/post-sparse-lil/diagram-sparse-matrix1.png" alt="Sparse 1D array" /></a><br>
<em>Sparse 1D array</em>
</p>

It can be represented as following list of pairs (index, value):

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-sparse-lil/1d-sparse-array-lil.png"><img src="{{ site.url }}/assets/post-sparse-lil/1d-sparse-array-lil.png" alt="1D sparse array as list of lists" /></a><br>
<em>Sparse 1D array</em>
</p>

The representation of 2D sparse matrix is similar; just store rows with its sequential number (index) in the another list. You have list (of rows) of lists (row's elements).

## Example of implementation in C++
CSparse1DArray is example draft C++11 implementation of fixed-size 1D sparse array of integers, based on list approach. The value of skipped fields can be customized, it's 0 by default.

{% highlight cpp %}
class CSparse1DArray
{
public:
    CSparse1DArray(size_t length, int defVal = 0);
    CSparse1DArray() = delete;

    int get(size_t idx) const;
    void set(size_t idx, int value);
    size_t size() const;

private:
    std::list<std::pair <size_t, int>> m_list;
    size_t                             m_size;
    int                                m_defVal;
};
{% endhighlight %}


{% highlight cpp %}
CSparse1DArray::CSparse1DArray(size_t length, int defVal)
    : m_list()
    , m_size(length)
    , m_defVal(defVal)
{
}

int CSparse1DArray::get(size_t idx) const
{
    for (auto it = m_list.begin(); it != m_list.end(); ++it)
    {
        if (idx == it->first)
        {
            return it->second;
        }
    }

    return m_defVal;
}

void CSparse1DArray::set(size_t idx, int value)
{
    if (idx >= m_size)
    {
        return;
    }

    auto it = m_list.begin();
    while (it != m_list.end())
    {
        if (it->first > idx)
        {
            break;
        }
        else if (it->first == idx)
        {
            it->second = value;
            return;
        }

        ++it;
    }

    m_list.insert(it, std::pair>size_t, int>(idx, value));
}
{% endhighlight %}

## Ready solutions
It may be clever to use already implemented solutions. For C++ usage, check out the possibilities of <a href="http://www.boost.org/doc/libs/1_45_0/libs/numeric/ublas/doc/matrix_sparse.htm">boost library for sparse matrices</a>. For Python try <a href="http://www.tp.umu.se/~nylen/pylect/advanced/scipy_sparse/index.html">sparse matrices in SciPy library</a>.