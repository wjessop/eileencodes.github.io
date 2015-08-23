---
layout: post
title: "Errors: Ruby 1.9.3 - uninitialized constant Capistrano (NameError)"
date: 2012-04-18
categories:
  - learning-rails
  - errors
---

<p>Today I learned that order does matter in your Gemfile.</p>
<p>I got the error <code>uninitialized constant Capistrano (NameError)</code> in the app I was developing when I tried to boot my local server. I had <code>gem 'rvm-capistrano'</code> listed before <code>gem 'capistrano'</code> - I guess when Rails tried to boot the app it read Capistrano from rvm-capistrano and assumed that Capistrano wasn't installed.</p>
<p>I intend to look deeper into this problem, but wanted to notfiy anyone who comes across the same issue.</p>
<p>I ended up solving this problem through <a href="http://www.ruby-forum.com/topic/4071984" target="_blank">Ruby Forum</a> from someone with a very similar problem but slightly different solution.</p>
