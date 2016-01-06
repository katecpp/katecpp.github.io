---
layout: post
title:  "QCreator TODO plugin"
permalink: qtcreator-todo
date:   2015-11-03
category: cpp
tags: [qt]
---
TODO plugin is very simple but extremely convenient plugin for QtCreator created by Dmitry Savchenko. It creates additional window of To-do Entries, which is displayed in the panel next to Compile Output. The colourful entries are sorted by type and you can easily navigate through all of them --- just click on the entry and you will be moved to the place in the code where the comment appears. It's great feature for the people who write TODO comments and then browse them by the usual <em>search text</em> function.

The To-do Entries window is divided into two sub-windows: entries in current document and entries in active project. Such division helps the proper work organization.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-todo/todo-plugin-qtcreator.png"><img src="{{ site.url }}/assets/post-todo/todo-plugin-qtcreator.png" alt="QtCreator todo plugin" /></a><br>
</p>

The plugin by default recognizes notes: TODO, BUG, FIXME, NOTE, HACK.

{% highlight cpp %}
// TODO: something to do
// BUG: ugly bug here
// FIXME: I need a fix!
{% endhighlight %}

## Plugin setup
The plugin is already installed in the newest QtCreator releases (I use QtCreator 3.4.2). You just need to enable it in<em> Help->About Plugins...->Utilities->Todo</em>.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-todo/todo-plugin-qtcreator2.png"><img src="{{ site.url }}/assets/post-todo/todo-plugin-qtcreator2.png" alt="QtCreator todo plugin" /></a><br>
</p>

## Add your own keywords
After the plugin is enabled, you can add/delete/edit entries types. Go to <em>Tools->Options->To-Do</em> and create some new keywords, for example mark the slow fragments of code with yellow comment SLOW. The keyword importance can be chosen from levels: information, warning, error.

<p style="text-align: center">
<a href="{{ site.url }}/assets/post-todo/todo-plugin-qtcreator3.png"><img src="{{ site.url }}/assets/post-todo/todo-plugin-qtcreator3.png" alt="QtCreator todo plugin" /></a><br>
</p>

I think this small feature can significantly optimize the work on the project!
