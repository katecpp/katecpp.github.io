---
layout: post
title:  "First program in Rust: hangman #1"
permalink: rust-hangman-1/
date:   2016-03-27
category: Rustlang
tags: [hangman, tutorial]
---
Rules of **Hangman game** should be well-known to everybody. The player is guessing a word or sentence letter by letter and if he manage to guess the whole secret sentence the hangman's life is saved. Here is the step by step solution of how I created a very simple Hangman game in Rust.

*This is my first program in Rust so it very likely that the code is not faultless nor optimal. Feel free to comment if you see some errors!*

## 1. Creating a project with Cargo

As the [Cargo docs](http://doc.crates.io/guide.html#why-cargo-exists) says, *Cargo is a build tool, which creates metadata files for your project, fetches and builds the project's dependencies, builds the project and introduces conventions.*

I wanted to keep my program simple and didn't plan to create very complicated dependencies. That's why first **I decided that I will not use Cargo and I will invoke rustc manually**. I thought it will be easier this way. I was in terrible mistake! Soon it became hard or impossible to build the project without Cargo. I needed to include library for *random* and the only one obvious solution for that was to, yes, **use cargo**. So here is my advice: **don't be afraid to use Cargo** from the beggining, since it's really nothing scary and in fact you gain a lot by using it.

#### Here is my **short tutorial** of how to use **Cargo**:

**Create new project**:`cargo new <project_name> --bin`  
**Build**: `cargo build`  
And **run**: `cargo run`  

It's not very hard to master these three steps.

#### What it gives to you in practice:

- Nice structure of folders,
- Initialized git repository with .gitignore file,
- Implemented HelloWorld as a skeleton,
- Cargo.toml which handles dependencies in easy way. These dependencies will arrive sooner than you think. It seems to be **the most important point**.

Check also: [Day 1 of 24 days of Rust: Cargo](http://zsiciarz.github.io/24daysofrust/book/day1.html).

## 2. Input sentences

The Hangman game requires some input sentences/words which the player will be guessing. I copied some english proverbs base to text file and removed commas and other punctuation marks. Proverbs are separated with newline sign. Such prepared file is read by the program line by line. Then it randomly chooses one line which will become a secret line to guess. It can be implemented like presented below.

{% gist c4265069706fe05b4736 %}

### 2.1 Random number
To use *random* feature we need to include **rand** crate. This can be done by adding dependency in Cargo.toml file:
{% highlight rust %}
[dependencies]
rand="0.3.0"
{% endhighlight %}
The crate must be also [linked](http://rustbyexample.com/crates/link.html) (line 1) and the **use** declaration must be added (line 3). Now the program is able to choose the random number - this is done in line 30. The [`gen_range`](https://doc.rust-lang.org/rand/rand/trait.Rng.html#method.gen_range) method returns the random number from the given range, in this example the range corresponds to the number of lines read from `input.txt` file.

### 2.2 Reading from file
Reading the file requires the **use** declaration from lines 3--6. In the 17 line two things are happening: the file `input.txt` is [opened](https://doc.rust-lang.org/std/fs/struct.File.html#method.open) and the result of this operation is processed by [**try! macro**](http://doc.rust-lang.org/std/result/index.html#the-try-macro). The try! macro is a part of **error handling** in Rust. It does the job for the method it wraps. Let's take a closer look on it.

### 2.3 Error handling: try!
The `File::open("input.txt")` call may look like it returns the *File* object. But if you check the documentation you will see that it returns *Result<File>* instead. When the given path does not exist it will return an *Error*, not a *File*. And when the path was valid it will return the *Result* which contains (wraps) the *File*. You must first unwrap the *Result* to get the access to the returned object. If you use a lot of functions that return a `Result`, it could add a lot of lines to your code. The **try! macro** handles the situation nicely, without causing your code size grows. Here you can see [the code with and without the usage of try!](https://doc.rust-lang.org/std/macro.try!.html). With try! it's much nicer, don't you think?

The try! macro deserves some longer considerations. You can read more about it here: [Module std::result](https://doc.rust-lang.org/std/result/) or here: [Learning to 'try!' things in Rust](http://www.jonathanturner.org/2015/11/learning-to-try-things-in-rust.html).

### 2.4 Iterating over the lines

The `BufReader::lines()` method, called in line 21, returns the iterator over all the lines from this buffer. The resulted `line` is also wrapped by `Result` -- it's of the type `io::Result<String>`. In line 23 we just ignore the `Result` value by using the `unwrap()` method. It returns only the `String` object. The value of `Result` is forgotten.

The important observation comes with lines 24 and 25. The loaded line is printed (24) and then appended to the vector (25). It works. But if you swap the order of line 24 and 25, the code won't compile. Check it! You will get an error: `error: use of moved value: l`.

I admit, I have seen such errors many times. Many, many times. It's very hard to get used to this "moved values". The reason of this error is so called [ownership](https://doc.rust-lang.org/book/ownership.html). In a short words, the value becomes "moved" (and thus unusable) when it is passed to function "normally" or assigned (binded) to another object. This is quite complicated and wide topic. In many cases you can avoid the "moved values" by passing the value by [reference](https://doc.rust-lang.org/book/references-and-borrowing.html). 

In line 32, the random secret line is returned, wrapped by the Result Ok. That's all what was needed for the first part of Hangman. The lines are read from file and one line is chosen and returned.

## 3. Read user guess

{% gist 2e478eb63b3d45aab60b %}



