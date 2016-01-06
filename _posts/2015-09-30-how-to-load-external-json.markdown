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
    
    $(function(){
        $.getJSON('questions.json',function(data){
            allQuestions = data.quiz;
            console.log('json loaded successfully');
        }).error(function(){
            console.log('error: json not loaded');
        });
    });
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
This solution can be developed further with encoding and decoding json.