---
layout: post
title: "Sorry for the Rough Travels"
date: 2012-03-27
categories:
  - learning-rails
  - updates
---

<p>I am not entirely sure why yet but I have had a lot of trouble lately with this blog and it throwing 500 errors. I have been having a lot of troulble with MySQL - and at first I thought I fixed it and belive I made it worse. I have switched some stuff back and will hopefully soon figure out what the problem is and will be able to write a proper post about my troubles.</p>
<p><a href="http://www.airbrake.io" target="_blank">Airbrake</a> a really nice app for this sort of thing because I don't need to read the logs to find out my site is trying to kill itself.</p>
<h3>This is what is happening:</h3>
<p>For awhile I was getting the following error:</p>

<code>
ActiveRecord::StatementInvalid: Mysql::Error: MySQL server has gone away: SELECT `posts`.* FROM `posts` ORDER BY created_at DESC LIMIT 10 OFFSET 0
</code>


<p>I added <code>reconnect:true</code> to the database.yml because I figured that would fix it.</p>
<p>What happened instead is really weird and I can't figure out why the above change would cause this. The app started throwing a new error, more often. The error is sometimes on posts and sometimes on categories and differs depending on the page but generally looks like this:</p>

<code>ActiveRecord::StatementInvalid: Mysql::Error: : SELECT `categories`.* FROM `categories` WHERE `categories`.`id` = ? LIMIT 1</code>
<p>What is weird is that's it's not getting the ID, but the page URL knows the ID, for example, when I click the category name with the ID 3, the slug is '/categories/3', but still throws the error on that page the mysql query is not getting the ID.</p>
<p>Maybe the answer is simple, but googling for the error doesn't result in much useful information. For now I've switched it back to the verison without the "reconnect:true" to see if the erros switch back again.</p>
<p>If you have any idea why this would be happening and would be interested in helping me out I'd really appreciate it. Find me on <a href="https://github.com/eileencodes" target="_blank">Github</a> or <a href="http://www.twitter.com/eileencodes" target="_blank">Twitter</a>.</p>
