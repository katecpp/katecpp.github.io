---
layout: post
title:  "First program in Rust: hangman"
permalink: rust-hangman-1/
date:   2016-03-27
category: Rustlang
tags: [hangman, tutorial]
---
Rules of **Hangman game** should be well-known to everybody. The player is guessing a word or sentence letter by letter and if he manage to guess the whole secret sentence the hangman's life is saved. Here is the step by step solution of how I created a very simple Hangman game in Rust to play in console.

*This is my first program in Rust so it very likely that the code is not faultless nor optimal. Feel free to comment if you see some errors!*

<div class="centered">
<img src="https://github.com/katecpp/Hangman/blob/master/hangman/screenshot/hangman.png?raw=true" alt="Hangman in Rust"/>
</div>


## 1. Creating a project with Cargo

As the [Cargo docs](http://doc.crates.io/guide.html#why-cargo-exists) says, *Cargo is a build tool, which creates metadata files for your project, fetches and builds the project's dependencies, builds the project and introduces conventions.*

I wanted to keep my program simple, so first **I decided that I will not use Cargo and I will invoke rustc manually**. I thought it will be easier this way. It was terrible mistake! Soon it became hard or impossible to build the project without Cargo. I needed to include library for *random* and the only one obvious solution for that was to, yes, **use cargo**. So here is my advice: **don't be afraid to use Cargo** from the beggining, since it's really nothing scary and in fact you gain a lot by using it.

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
- Cargo.toml which handles dependencies in easy way. These dependencies will arrive sooner than you think. It seems to be **the most important point**.

Check also: [Day 1 of 24 days of Rust: Cargo](http://zsiciarz.github.io/24daysofrust/book/day1.html).

## 2. Input sentences

The Hangman game requires some input sentences/words which the player will be guessing. I copied some english proverbs base to text file and removed commas and other punctuation marks. Proverbs are separated with newline sign. Such prepared file is read by the program line by line. Then it randomly chooses one line as a secret line to guess. It can be implemented like presented below.

{% gist c4265069706fe05b4736 %}

### 2.1 Random number
To use *random* feature we need to include **rand** crate. This can be done by adding dependency in Cargo.toml file:
{% highlight rust %}
[dependencies]
rand="0.3.0"
{% endhighlight %}
The crate must be also [linked](http://rustbyexample.com/crates/link.html) (line 1) and the **use** declaration must be added (line 3). Now the program is able to choose the random number - this is done in line 30. The [`gen_range`](https://doc.rust-lang.org/rand/rand/trait.Rng.html#method.gen_range) method returns the random number from the given range, in this example the range corresponds to the number of lines read from `input.txt` file.

### 2.2 Reading from file
Reading the file requires the **use** declaration from lines 3--6. In the 17 line two things are happening: the file *input.txt* is [opened](https://doc.rust-lang.org/std/fs/struct.File.html#method.open) and the result of this operation is processed by [**try! macro**](http://doc.rust-lang.org/std/result/index.html#the-try-macro). The try! macro is a part of **error handling** in Rust. It does the job for the method it wraps. Let's take a closer look on it.

### 2.3 Error handling: try!
The `File::open` method returns `Result<File>` instead of just `File`. When the given path does not exist it will return *Result Error*. And when the path was valid it will return the *Result Ok* which contains (wraps) the opened file object. To access this file you must do something with the wrapping Result. You can process it with *match*, and provide the path for both possible Results *Ok* and *Err*, like presented in [File::Open chapter of Rust by Example](http://rustbyexample.com/std_misc/file/open.html):

{% highlight rust %}
let mut file = match File::open(&path) {
    // The `description` method of `io::Error` returns a string that
    // describes the error
    Err(why) => panic!("couldn't open {}: {}", display,
                                               Error::description(&why)),
    Ok(file) => file,
};
{% endhighlight %}

This solution unfortunately makes your code bigger. 

The **try! macro** handles the situation nicely, without increasing your code size. Here you can see [the code with and without the usage of try!](https://doc.rust-lang.org/std/macro.try!.html). With try! it's much nicer, don't you think?

The try! macro deserves some longer considerations. You can read more about it here: [Module std::result](https://doc.rust-lang.org/std/result/) or here: [Learning to 'try!' things in Rust](http://www.jonathanturner.org/2015/11/learning-to-try-things-in-rust.html).

### 2.4 Iterating over the lines

The `BufReader::lines()` method, called in line 21, returns the iterator over all the lines from this buffer. The resulted *line* is also wrapped by *Result* -- it's of the type `io::Result<String>`. In line 23 we just ignore the *Result* value by using the `unwrap()` method. It returns only the *String* object. The value of *Result* is forgotten. You can wonder why we just do that instead of the proper error handling like we discussed in 2.3 section. It's because I didn't found the scenario where the *Result Error* is returned. Even in [examples](https://doc.rust-lang.org/std/io/trait.BufRead.html#method.lines) the result value is ignored. **Usually calling *unwrap()* isn't the best solution, but [it's not always evil](https://doc.rust-lang.org/stable/book/error-handling.html#a-brief-interlude-unwrapping-isnt-evil).**

#### Ownership is hard on the beggining
The important observation comes with lines 24 and 25. The loaded line is printed (24) and then appended to the vector (25). It works. But if you swap the order of line 24 and 25 (first append to vector and then print), the code won't compile. Check it! You will get an error: `error: use of moved value: l`.

I admit, I have seen such errors many times. Many, many times. It's very hard to get used to this **moved values**. The reason of this error is so called [ownership](https://doc.rust-lang.org/book/ownership.html). In a short words, the value becomes **moved** (and thus unusable) when it is passed to function "normally" or assigned (binded) to another object. This is quite complicated and wide topic. In many cases you can avoid the "moved values" by passing the value by [reference](https://doc.rust-lang.org/book/references-and-borrowing.html). Check also: [Why Rust's ownership/borrowing is hard](http://softwaremaniacs.org/blog/2016/02/12/ownership-borrowing-hard/en/).

In line 32, the random secret line is returned, wrapped by the Result Ok. That's all what was needed for the first part of Hangman. The lines are read from file and one line is chosen and returned.

## 3. Read user guess

Let's start the interaction with user. He should type his guesses in a loop.

{% gist b56baa97cb41cfd5bbbf %}

Player is asked for typing a letter in line 16, and his input is read by function `read_guess()`. Pay attention to the return type of this function: `Option<char>`. If you know C++ the *Option* can remind you of [boost::optional](http://katecpp.github.io/boost-optional/). Such return type is recommended because the user answer can be invalid sometimes and it may not contain any letter.

In line 34 we read the whole line that user types. I wanted to read only a single *char*, but I didn't find the way to do so. If you know how to do it better, let me know! The line is stored in variable *guess*. Then the *guess* is *trim()*-ed, which removes any whitespaces from the line (in case the user typed the whitespaces before his guess) and then the 0-th (first) char is taken from resulted string. The [*nth()* method](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.nth) returns the `Option<char>`, because the nth value may not exist.

The basic validation is done in *validate_user_guess* method, which generally only returns true when the user input was a letter and false in all other cases. In line 21 we unwrap the result because we already checked that we can do it.

## 4. Prepare structure for game data

The game data can be kept together in a structure. 

{% highlight rust %}
struct GameData {
    secret_line         : String, // the sentence to guess
    discovered_letters  : String, // all letters given by player
    lives               : i32,    // lives left until hangman dies
    status              : String  // latest message to player
}
{% endhighlight %}

Also an enum is introduced for deeper validation of user input. We already know that this is letter for sure, but this letter can be either already discovered (and should be skipped), or missed, or correctly discovered.

{% highlight rust %}
enum UserInputStatus {
    AlreadyDiscovered,
    LetterGuessed,
    LetterMissed,
}

fn check_user_guess(gd: &GameData, user_guess: char) -> UserInputStatus
{
    if gd.discovered_letters.contains(user_guess)
    {
        return UserInputStatus::AlreadyDiscovered;
    }

    if !gd.secret_line.contains(user_guess)
    {
        return UserInputStatus::LetterMissed;
    }

    UserInputStatus::LetterGuessed
}
{% endhighlight %}




## 5. Beautifying

#### Printing the hangman
Of course we need to print the hangman. I think the code will explain himself ;).

{% highlight rust %}
fn print_hangman(gd: &GameData)
{
    match gd.lives
    {
        0 =>
        {
            println!(" _________   ");
            println!("|         |  ");
            println!("|         XO ");
            println!("|        /|\\ ");
            println!("|        / \\ ");
            println!("|            ");
            println!("|            ");
        }

        // end so on...
}
{% endhighlight %}

#### Use some colors
For a little bit nicer output we can add color to our game. I've done it with the use of [ansi_term crate](https://crates.io/crates/ansi_term). Now the messages to user are red, green or yellow depending on their character. This crate is very easy to use. For example the happy message:

{% highlight rust %}
let status = format!("You discovered {}", guess_lower);
gd.status = Green.paint(status).to_string();
{% endhighlight %}

#### Clear the screen
This is the feature that I'm afraid will work only on Linux. Everytime the game status is changed I redraw the hangman and all. I think it looks better if the old hangman is removed, so I clear the screen with the solution from here: [davidbegin/clear-terminal](https://github.com/davidbegin/clear-terminal).

{% highlight rust %}
fn clear()
{
  let output = Command::new("clear").output().unwrap_or_else(|e|{
    panic!("failed to execute process: {}", e)
  });
  println!("{}", String::from_utf8_lossy(&output.stdout));
}
{% endhighlight %}

