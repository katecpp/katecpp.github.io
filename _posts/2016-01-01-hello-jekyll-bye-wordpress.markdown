---
layout: post
title:  "Hello Jekyll, bye Wordpress.com!"
permalink: hello-jekyll
date:   2016-01-01
category: jekyll
---

It's been several months since I started blogging at Wordpress.com. As the time went by I&nbsp;was more and more confident that WP.com is not what I&nbsp;need.

I find the Wordpress.com blogs ugly. Despite my attempts to make something more out of it, still I&nbsp;had an impression that my page is hurting eyes. I have visited numerous blogs hosted on WP.com and I&nbsp;made my decision -- it's not about me, it's the platform. Almost none of these blogs was looking nice to me, although many of them were popular and contained very interesting content.

It was completely different with the github/Jekyll blogs I visited. They looked simple and minimalistic. Their design was not distracting me from the contents. If this is what I need, why not give it a try? And so I am here.

### What's wrong with Wordpress.com
+ #### Poor choice of themes
The biggest disadvantage is the themes thing. I have access only to limited number of themes that must be chosen from the official WP.com offer. I have no access to html sources! All I can customize are the details like the page logo or the background color. It's really hard to make something satisfying with that! Besides most of the themes seem not designed for technical-articles purpose -- the width of the post is very limited, the font size is quite big and in result I can hardly fit the code snippets there. Really, why use only a half of the average monitor space? It all resulted that I&nbsp;was disappointed with my blog layout.

+ #### Can't use Google Analytics
Google analytics is not working Wordpress.com Free Plan. To use the Google Analytics you need to buy the Business Plan which costs 299$ per year. WP.com provides its own statistics anyway I would prefer to use Google Analytics instead of that.

+ #### Painful bug
When I switch between html and visual mode of post, the special signs inside the `[code]` blocks become corrupted --- `&` is published as `&amp;` and so on. This bug is almost 100% systematic for me. I also saw such corrupted signs on some other WP.com blogs. 

### What I like in github pages/Jekyll
+ #### It looks nice enough
In my opinion the github blogs are very eye-pleasing, even with very simple default themes. The technical posts with code snippets look delightful. And if they are not, there is plenty of ways to improve it, since you have full control over the page look.

+ #### The whole thing is up to you
You own all page sources. You can, but don't have to, modify every little detail on the page. And you can use Google Analytics.

+ #### It's free!
Github pages and Jekyll are totally free. 

### What I'm afraid to lose
+ #### Tags for posts
It seems that there is none ready-to-use mechanism for tagging the posts. However, I found some solution described here: [www.minddust.com/post/tags-and-categories-on-github-pages][minddust] -- not tried it yet but it sounds promising.

+ #### No pingback mechanism
Wordpress pingback mechanism is great. When you mention another blog post, no matter if it's yours or not, the mention can be published in shape of pingback below the original post. That creates a great follow-up hierarchy and the reader can easily navigate through interesting content between various blogs.

+ #### Follow blog
Wordpress blogs can be followed with one click. I did not found possibility to follow Jekyll blog other than RSS.

+ #### Page address
I don't own the personal domain and it will probably stays this way. When comparing the addresses katecpp.wordpress.com and katecpp.github.io the first one seems for me a little bit better for memorizing and more serious.

### The migration process
I found a lot of posts describing the Wordpress to github/Jekyll migration process. These posts was particularly helpful: 

*  [How-to: Migrating Blog from WordPress to Jekyll, and Host on Github][girliemac] by girliemac, 

*  [How I Created a Beautiful and Minimal Blog Using Jekyll, Github Pages, and poole][joshualande] by joshualande,

*  [How to Set Short URLs in Jekyll with Github Pages][joshualande_2] by joshualande,

*  [Bye Wordpress, hello Jekyll!][jandemooij] by jandemooij.

[minddust]: http://www.minddust.com/post/tags-and-categories-on-github-pages/
[girliemac]: http://www.girliemac.com/blog/2013/12/27/wordpress-to-jekyll/
[joshualande]: http://joshualande.com/jekyll-github-pages-poole/
[joshualande_2]: http://joshualande.com/short-urls-jekyll/
[jandemooij]: http://jandemooij.nl/blog/2015/10/03/bye-wordpress-hello-jekyll/