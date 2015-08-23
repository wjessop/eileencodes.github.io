---
layout: post
title: "An ActiveRecord Assumption: Using delete_all on a Related Model, StackLevel Too Deep"
date: 2013-09-29
categories: errors gems activerecord mysql
---

<p>Recently I've been working with larger datasets than I previously have in the past, which has led me to learn much more about how ActiveRecord handles mysql queries. When using a gem or a library that does a lot of behind the scenes magic, there is a tendency to rely on that magic; until it bites us in the ass. I got myself into such a situation with <code>delete_all</code> on a related model.</p>
<p>I changed a bit of our code in the PhishMe app to use <code>delete_all</code> instead of <code>destroy_all</code> or a method we previously used to destroy in batches. I wanted this process to be faster and since we didn't need the callbacks <code>delete_all</code> would work perfectly.</p>
<p>I expected that the call for Model.related_mode.delete_all to produce the following mysql query:</p>
{% highlight sql %}
DELETE FROM `related_model` WHERE `related_model`.`id` = '1'
{% endhighlight %}
<p>In fact, the produced mysql that was created by the ActiveRecord query was:</p>

{% highlight sql %}
DELETE `model` WHERE `model`.`id` = '1' AND ((`related_model`.`id` = '2'<br />OR `related_model`.`id` = '3' OR `related_model`.`id` = '4' OR<br />`related_model`.`id` = '5' OR...))
 13 {% endhighlight %}

<p>The above query normally wouldn't cause any issue and you might not even noticed the slowed down query, but if you have thousands of records it blows up, and not gracefully.</p>
<p>When you hit the threshold of the number of records that can be chained, ActiveRecord spits out a "StackLevel too deep". (facepalm) After I thought about it, the issue was obvious. The stack really <em>is</em> too deep. I did actually ask mysql to chain all the methods. I didn't expect this behavior because I had never considered the ramifications of the <code>Model.related_model.delete_all</code> call I was making.</p>
<p>I'm becoming very interested in how our ActiveRecord calls translate into mysql queries. It's easy with "magic" gems to ignore the underlying behavior.</p>
