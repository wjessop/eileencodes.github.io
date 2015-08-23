---
layout: post
title: "Just Use `ack`"
date: 2014-01-15
categories: tips
---

<p>I spent a good portion of yesterday obsessing over my grep colors. They worked fine on my VM (Ubuntu) but not on my mac. I then figured out that mac is FreeBSD and Linux grep is GNU. So the <code>export GREP_COLORS='1;32'</code> just isn't going to work.</p>
<p>I then spent too much time trying to get that to work when all the Stackoverflow sources said just use ack. So I gave in and installed it because I was tired of doing it "the hard way".</p>
<p>It's amazing. I know most of you probably already use ack. If you don't you are "doing it wrong" as I have been for some time. Now stop wasting time reading this and go download it.&nbsp;</p>
<p>Installation instructions <a href="http://beyondgrep.com/install/" target="_blank">here</a>.</p>
<p>Colors and line numbers are on by default! Yay! To change the colors simple add this to your .bash_profile or whatever your preference:</p>

{% highlight text %}
alias ack='ack --color-filename="red bold" --color-match="yellow bold" --color-lineno=white'
{% endhighlight %}

<p>You're welcome.</p>
