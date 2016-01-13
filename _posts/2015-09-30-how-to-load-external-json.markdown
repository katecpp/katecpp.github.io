---
layout: post
title:  "How to load external .json to array in js"
permalink: load-json
date:   2015-09-30
category: js
tags: [json]
---
During my work on <a href="http://katecpp.github.io/qt-exam-preparations/">the javascript Qt quiz</a>, I had to decide how to store quiz questions. I have chosen the external json file for that purpose. This post describes this solution.
<ol>
	<li>Write json content with the use of json parser to avoid typos. Parsers like <a href="http://json.parser.online.fr/">http://json.parser.online.fr/</a> will help you to painfully create even the longest json files --- you will see error immediately when you break the syntax.</li>
	<li>The valid json can look like this:

{% highlight js %}
 {
     "quiz": 
     [
         {
             "question": "Question 1",
             "a": "Answer a",
             "b": "Answer b",
             "c": "Answer c",
             "correct": "a"
         },
         {
             "question": "Question 2",
             "a": "Answer a",
             "b": "Answer b",
             "c": "Answer c",
             "correct": "b"
         },
         {
             "question": "Question 3",
             "a": "Answer a",
             "b": "Answer b",
             "c": "Answer c",
             "correct": "c"
         }
     ]
 }
{% endhighlight %}

</li>
	<li>Save json in the same folder as your js script. I named my file <code>questions.json</code>.</li>
	<li>Load the json data with <code>$.getJSON</code>. Following example is complete script that loads the questions from json to <code>allQuestions</code> array.

{% highlight js %}
var allQuestions = new Array();
    
function loadQuestions() {
    $.getJSON('question.json', function (data) {
        allQuestions = data.quiz;
    }).error(function(){
            console.log('error: json not loaded');
        });
    });
}
{% endhighlight %}

</li>
<li>Now your json data is available in <code>allQuestions</code> array and you can access it, for example:<br>

{% highlight js %}
var currentQuestion = allQuestions[0].question;
var answerA         = allQuestions[0].a; 
/* and so on*/
{% endhighlight %}

</li>
</ol>


### Using .done callback
Remember that`$.getJSON` runs asynchronously. The code like below doesn't guarantee correct execution. The implementation of `printQuestion` doesn't matter, it just needs to do anything with the `question` argument.
{% highlight js %}
loadQuestions();
printQuestion(allQuestions[0]);
// or do anything else what requires allQuestions already loaded
{% endhighlight %}

It's possible that by the moment of calling `printQuestion` the JSON will not be loaded yet and the allQuestions size will be 0. To ensure that some operations are done after the JSON is fully loaded, use `.done` callback. It's executed when the JSON loading is completed.

{% highlight js %}
var allQuestions = new Array();
    
function loadQuestions() {
    $.getJSON('question.json', function (data) {
        allQuestions = data.quiz;
    })
    .error(function() {
        console.log('error: JSON not loaded'); 
    })
    .done(function() {
        console.log( "JSON loaded!" );
        printQuestion(allQuestions[0]); 
    });
}

{% endhighlight %}