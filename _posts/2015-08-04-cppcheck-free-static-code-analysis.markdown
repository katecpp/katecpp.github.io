---
layout: post
title:  "Cppcheck - free static code analysis"
permalink: cppcheck
date:   2015-08-04
category: cpp
tags: [static-analysis]
---

CppCheck is a very helpful tool for C++ programmers. It performs the static code analysis of C++ project and discovers some types of error which can be easily overlooked by developers and compilers: out of bounds or uninitialized variables, redundant code, always true/false comparisons, exception safety and many others (all checks are listed <a href="http://sourceforge.net/p/cppcheck/wiki/ListOfChecks">here</a>). If you want to maintain high code quality you should include the static code checks among the development routines.

### Setup
CppCheck works on Linux and Windows. It is a free software under the GNU General Public License.
Download CppCheck from the <a href="http://sourceforge.net/projects/cppcheck/">project page</a> or install <a href="http://installion.co.uk/ubuntu/precise/universe/c/cppcheck/install/index.html">via command line</a>.

### Usage via console
Open the console and navigate to the project directory.
   
**Check specific file and save the result to .txt file**:
{% highlight bash %}
cppcheck filename.cpp 2> result.txt
{% endhighlight %}

**Check all files in current directory recursively**: 
{% highlight bash %}
cppcheck . 2> result.txt
{% endhighlight %}

**Perform all possible checks**

By default only error messages are shown. To enable more messages use <em>enable</em> flag: `--enable=all` will perform all checks possible. Other possible values are `warning`, `performance`, `information`, `style`, `unusedFunctions`. 
{% highlight bash %}
cppcheck --enable=all filename.cpp 2> result.txt
{% endhighlight %}

### Example result and interpretation
I have cloned an open-source project <a href="https://github.com/QNapi/qnapi">QNapi</a> and then tested the whole repository with the cppcheck. With the default check (only error level) no deviations were detected. Congratulations for the team :). However I go further and check with `--enable=warning` flag. Now I see 9 warnings about uninitialized variables. Some of them are uninitialized member pointers -- and this thing really deserves correction. Example:

{% highlight bash %}
[src/qnapiapp.cpp:17]: (warning) Member variable 
'QNapiApp::napisy24SubMenu' is not initialized in the constructor.
{% endhighlight %}

The check with `--enable=perfomance` produces additionally warnings like:

{% highlight bash %}
[src/qnapiconfig.cpp:128]: (performance) Prefer prefix ++/-- 
operators for non-primitive types.
{% endhighlight %}
This one should not make a big difference in desktop application and can be optimized by compiler anyway, so we can ignore it. I go further with `--enable=style` and I find:
{% highlight bash %}
[src/forms/frmlistsubtitles.cpp:60]: (style) Expression is 
always false because 'else if' condition matches previous 
condition at line 58.`
{% endhighlight %}
This is interesting --- such piece of code probably does not work intended way:

{% highlight cpp %}
if(highlight && (s.resolution == SUBTITLE_GOOD))
   ++good;
else if(highlight && (s.resolution == SUBTITLE_GOOD))
   ++bad;
{% endhighlight %}

Whatever was the purpose of `bad` variable, we can suspect that is not correctly done since the value of `bad` in never incremented in this snippet.

### Speed
The analysis is very fast - for such small projects it lasts about one second. Detected bugs probably would not be found quickly without the help of this tool. In my opinion running the static analysis test is well-spent time. After having the results, we can make some improvements in our code.

### Integrate cppcheck with eclipse
CppCheck can be easily integrated with Eclipse. Read Code Yarns article <a href="http://codeyarns.com/2015/06/11/how-to-use-cppcheck-with-eclipse-cdt/" target="_blank">How to use CPPCheck with Eclipse CDT</a> for comprehensive step-by-step setup instruction .

### Other static analysis tools
It's rather hard to find free substitute for CppCheck. A lot of commercial static code analysis tools are available on the market (i.e. QAC, Klocwork), but if we focus on the open source tools the choice become dramatically smaller. For now I haven't found any other noteworthy tool for C++. Any comments and suggestions about the alternative are welcome.