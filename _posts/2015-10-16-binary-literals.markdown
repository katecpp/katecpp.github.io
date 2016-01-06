---
layout: post
title:  "Binary literals"
permalink: binary-literals
date:   2015-10-16
category: cpp
tags: [cpp14]
---
Until C++14, standard C++ allowed to define numbers in:

+ decimal notation:     `int number = 7;`

+ hexadecimal notation: `int number = 0x7;`

+ octal notation:       `int number = 07;`

Anyhow, without special compiler extensions it was not allowed to define numbers in binary format. It changes from C++14. The core language supports binary literals. It's possible to define integer with `0b` or `0B` prefix to represent binary number.

`int number = 0b0111;`

It's worth noticing that the GCC compiler has offered the possibility to define binary numbers since <a href="https://gcc.gnu.org/gcc-4.3/changes.html">GCC 4.3</a>.

It may be not the most crucial feature of C++ standard, but I believe there are cases when use of binary representation will improve readability of the code.
