---
layout: post
title:  "First program in Rust: Hangman"
permalink: rust-hangman/
date:   2016-04-01
category: rustlang
tags: [hangman, tutorial]
---
Rules of **Hangman game** should be well-known to everybody. The player is guessing a word or sentence letter by letter and if he manage to guess the whole secret sentence the hangman's life is saved. Here is the step by step solution of how I created a very simple Hangman game in Rust to play in the console.

*This is my first program in Rust so it very likely that the code is not faultless nor optimal. Feel free to comment if you see some errors! The whole code can be found on [Github](https://github.com/katecpp/Hangman).*

<div class="centered">
<img src="https://github.com/katecpp/Hangman/blob/master/hangman/screenshot/hangman.png?raw=true" alt="Hangman in Rust"/>
</div>


## 1. Creating a project with Cargo

As the [Cargo docs](http://doc.crates.io/guide.html#why-cargo-exists) say, *Cargo is a build tool, which creates metadata files for your project, fetches and builds the project's dependencies, builds the project and introduces conventions.*

I wanted to keep my program simple, so first **I decided that I will not use Cargo and I will invoke rustc manually**. I thought it will be easier this way. It was a terrible mistake! Soon it became hard or impossible to build the project without Cargo. I needed to include a library for *random* and the only one obvious solution was to add it as the dependency in *Cargo.toml*. So here is my advice: **don't be afraid to use Cargo** from the beginning, since it's really nothing scary and you gain a lot by using it.

#### Here is my **short tutorial** of how to use **Cargo**:

{% highlight bash %}
cargo new <project_name> --bin # create new project
cargo build # build
cargo run # run
{% endhighlight %}

It's not very hard to master these three steps.

#### Benefits of using Cargo:

- Nice structure of folders,
- Initialized git repository with .gitignore file,
- Implemented *HelloWorld* as a skeleton,
- Cargo.toml which handles dependencies in an easy way. These dependencies will arrive sooner than you think. It seems to be **the most important point**.

Check also: [Day 1 of 24 days of Rust: Cargo](http://zsiciarz.github.io/24daysofrust/book/day1.html).

## 2. Input sentences

The Hangman game requires some input sentences/words which the player will be guessing. I copied some English proverbs base to text file and removed commas and other punctuation marks. Proverbs are separated with newline sign. The program reads from such [prepared file](https://github.com/katecpp/Hangman/blob/master/hangman/input.txt) line by line and chooses randomly one line as a secret line to guess. The implementation of choosing the secret line is presented and discussed below.

{% gist c4265069706fe05b4736 %}

### 2.1 Random number
To use the *random* feature we need to include [**rand crate**](https://doc.rust-lang.org/rand/rand/index.html) by adding a dependency in *Cargo.toml* file:
{% highlight rust %}
[dependencies]
rand="0.3.0"
{% endhighlight %}
The crate must be also [linked](http://rustbyexample.com/crates/link.html) (line 1) and the **use** declaration must be added (line 3). Now the program is able to choose the random number. The [`gen_range`](https://doc.rust-lang.org/rand/rand/trait.Rng.html#method.gen_range) method returns the random number from the given range. Here the range corresponds to the number of lines read from the `input.txt` file.

### 2.2 Reading from file
Reading the file requires the use declarations from lines 3--6. In the 17 line, two things are happening: the file *input.txt* is [opened](https://doc.rust-lang.org/std/fs/struct.File.html#method.open) and the result of this operation is processed by [**try! macro**](http://doc.rust-lang.org/std/result/index.html#the-try-macro). Then the opened file is passed to the [BufReader object](https://doc.rust-lang.org/std/io/struct.BufReader.html), which provides the methods for the line by line reading. The try! macro is a part of **error handling** in Rust. It does the job for the method call it wraps. Let's take a closer look at it.

### 2.3 Error handling

One of the Rust main characteristics is safety, so it's not a surprise that the error handling in Rust is quite a wide topic. In this short code snippet we already get in touch with `Result<Type>` and two different ways of handling it: **try! macro** and **unwrap/expect**.

#### try! macro

The `File::open` method returns `Result<File>` instead of just `File`. When the given path does not exist it will return *Result Error*, otherwise it will return the *Result Ok* wrapping the opened file. To access the file you must do something with this Result. You can process it with *match*, and provide the path for both possible Results *Ok* and *Err*, like presented in [Rust by Example](http://rustbyexample.com/std_misc/file/open.html):

{% gist 4078208fddc669e9a10dcbf2f0ba9442 %}

Unfortunately, this solution makes your code bigger. 

The **try! macro** handles the situation nicely, **without increasing your code size**. Here you can see [the code with and without the usage of try!](https://doc.rust-lang.org/std/macro.try!.html). With try! it's much nicer, don't you think?

#### expect and unwrap

The `File::open` is not the only one call here that return the *Result* type. We handle the *Results* also in the 12 line and 23 line with methods `unwrap` and `expect`. These both methods are basically the same. You can use them for uwrapping the object from the result when you feel brave enough to ignore the *Result* value. The difference between `unwrap` and `expect` is that the latter one provides custom message in case of error. This approach isn't the safest and can make your program panic! or abort. We will discuss it more in the next section.

Check also: [Module std::result](https://doc.rust-lang.org/std/result/) and [Learning to 'try!' things in Rust](http://www.jonathanturner.org/2015/11/learning-to-try-things-in-rust.html).

### 2.4 Iterating over the lines

The `BufReader::lines()` method returns the iterator over all the lines from this buffer. The resulted *line* is wrapped by *Result* -- it's of the type `io::Result<String>`. In line 23 we just ignore the *Result* value by using the `unwrap` method. It returns only the *String* object and the value of *Result* is forgotten. You can wonder why we don't do the proper error handling e.g. with try! macro. It's because I didn't found the scenario where the *Result Error* is returned. Even in [examples](https://doc.rust-lang.org/std/io/trait.BufRead.html#method.lines), the result value is ignored. **Usually calling *unwrap* isn't the best solution, but [it's not always evil](https://doc.rust-lang.org/stable/book/error-handling.html#a-brief-interlude-unwrapping-isnt-evil).**

#### Ownership is hard at the beginning
The important observation comes with lines 24 and 25. The loaded line is printed (24) and then appended to the vector (25). It works. But if you swap the order of line 24 and 25 (first append to vector and then print), the code won't compile. Check it! You will get an error: `error: use of moved value: l`.

I admit, I have seen such errors many times. It's very hard to get used to this **moved values**. The reason for this error is so called [ownership](https://doc.rust-lang.org/book/ownership.html). In short words, the value becomes **moved** (and thus unusable) when it is passed to function directly or assigned (bound) to another object. This is quite complicated and wide topic. In many cases, you can avoid the moved values by passing them by [reference](https://doc.rust-lang.org/book/references-and-borrowing.html). 

Check also: [Why Rust's ownership/borrowing is hard](http://softwaremaniacs.org/blog/2016/02/12/ownership-borrowing-hard/en/).

In line 32, the random secret line is returned, wrapped by the *Result Ok*. That's all that was needed for the first part of Hangman. The lines are read from the file and one line is chosen and returned.

## 3. Read user guess

Let's start the interaction with user. His job is typing a letter repeatedly until he discovers the whole secret word.

{% gist b56baa97cb41cfd5bbbf %}

The user input is read by function `read_guess`. The whole line is stored in *guess* variable (I didn't find a way to read only one char). Then the *guess* is *trimmed*, which removes any whitespaces from the line (in case the user typed the whitespaces before his guess) and then the first char is taken from resulted string. The [nth() method](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.nth) returns the [`Option<char>`](https://doc.rust-lang.org/std/option/), because the *nth* value may not exist. If you know C++ the *Option* can remind you of [boost::optional](http://katecpp.github.io/boost-optional/). 

The basic validation is done in *validate_user_guess* method, which returns **true if the first non-white sign was an alphabetical letter** and false in all other cases. In line 21 we unwrap the resulted *user_guess* (we know it's valid letter since the *validate_user_guess* returned true) and convert the letter to lowercase (the whole part `.to_lowercase().next().unwrap()` could be skipped if you want to distinguish the case).

## 4. Prepare structure for game data

Let's create a structure that keeps data defining the game state. The fields of GameData are self-explanatory, but if you have doubts refer to comments.

{% gist c7ce75265b4f75107a20c8498f549fd0 %}

Also an enum is introduced for deeper validation of user input. We already have checked if user typed a letter, but this letter can be either **already discovered** (and this choice should be skipped), or **missed** (and we need to decrease the lives number), or **guessed**. The **UserInputStatus** will help us to decide on what to do with the letter.

{% gist 953f0a98a5a89323ac726090c5508d95 %}

#### Display the secret word
We need to print the secret word with guessed letters discovered and others hidden. For this purpose we create a function `format_masked_string`, that returns a string with all undiscovered letters replaced with sign `_`. It also separates the letters with spaces for nicer look. The input argument is the string to be masked, and the mask is just a string containing all discovered letters. 

{% gist da9be6f738e114910891ffee8b5a1aaf %}

### Putting it all together
Now we have all necessary mechanics. We only need to put it all together and enjoy the hangman game. **Match** expression located in lines 25-62 takes care of the proper reaction to the letter typed by the player. [Match](https://doc.rust-lang.org/book/match.html) is simple **switch** equivalent. 

{% gist 4c7f832a1071a7e804529ba29ad44c2b %}

The game is already playable, but the user interface is poor. Let's make it more user friendly.

## 5. Beautifying

#### Printing the hangman
Of course we need to print the hangman. I think the code will explain himself ;).

{% gist bfd38f750642d197dd226c0fa69c6aaf %}

#### Use some colors
For a little bit nicer output we can add color to our game. I've done it with the use of [ansi_term crate](https://crates.io/crates/ansi_term). Now the messages to user are **red, green or yellow depending on their character**. This crate is very easy to use. You can save the colorful message to plain String object after calling `to_string` method.

{% gist 4aacc5d74ad1c8c42c88ee08a3567234 %}

#### Clear the screen
This is the feature that I'm afraid will work only on Linux. Every time the game status is changed I redraw the hangman and all. I think it looks better if the old hangman is removed, so I clear the screen with the solution from here: [davidbegin/clear-terminal](https://github.com/davidbegin/clear-terminal).

## Game is ready!

Here is the full, final source code: [hangman/src/main.rs](https://github.com/katecpp/Hangman/blob/master/hangman/src/main.rs).



