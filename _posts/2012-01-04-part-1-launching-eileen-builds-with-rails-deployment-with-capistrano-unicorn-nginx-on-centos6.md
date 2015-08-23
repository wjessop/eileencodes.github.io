---
layout: post
title: "Part 1: Launching Eileen Builds with Rails: Deployment with Capistrano, Unicorn, Nginx on Centos6"
date: 2012-01-04
categories:
  - learning-rails
  - deployment
---

<p>Note: I never did finish parts 2-4 for this blog post. I waited too long to write them and forgot how they worked. I've left this post up even though it's not really valid any longer. There are much better articles now on getting this running.</p>

<p>I'm going to do this post in four parts. Part 1: checklist things to do before deployment, Part 2 Capistrano, Part 3: Unicorn, and Part 4: Nginx. I also don't have the ability for draft posts set up yet, so...instead I'm publishing in parts so I don't need to write it all at once.</p>
<p>Launching this Rails app was not easy, although since I had never deployed an app, I wasn't expecting it to be easy. There are a bunch of little things you need to know about deployment if it's your first time that I think it would be helpful for other first-timers. Also, when you've been working on that web app for 24 hours straight you forget the all important mental checklist.</p>
<p>Please feel free to correct me to tell me I'm "doing it wrong" because this is my first deployment and I will fully admit I might have no idea what I'm doing.</p>
<h3>Preparing Your App for Deployment</h3>
<p>One thing I failed to do and I regret a lot is making sure certain files aren't include in git with my <code>.gitignore</code>. Files to remove are:</p>
<ul>
<li>db/*sqlite3</li>
<li>log/*.log</li>
<li>public/assets* (now I did this because I wasn't using any public assets, and didn't want the rouge stylesheets being added)</li>
<li>Gemfile.lock</li>
</ul>
<p>Chances are if your app is running on sqlite3 in development it won't be in production and will more likely be using a MySQL database or PostgreSQL. My app is using MySQL so we'll go over setting up your Gemfile for that.</p>
<p>If you have just gem 'sqlite3' in your Gemfile you'll need to change it to what I have below. While here you'll probably want to add the gems needed for deployment as well.</p>

{% highlight ruby %}
group :development do
  gem 'sqlite3'
end

group :production do
  gem 'mysql'
end

# Use unicorn as the web server
gem 'unicorn'

# Deploy with Capistrano
gem 'capistrano'
gem 'capistrano-unicorn'
{% endhighlight %}

<p>Before deploying you'll need to precomile your assets with:</p>
<pre>rake assets:precompile</pre>
<p>This is really important because I forgot this step and could not figure out why my assets weren't showing up and I was getting a lot of "No Route Matches" in my logs. I didn't find this anywhere other were having this problem and no one said "hey maybe you forgot this really important thing."</p>
<p>Once capistrano is set up you can automate this task during deployment, but we'll cover that in Part 2.</p>
<p>When Parts 2-4 are written they will be linked to at the bottom of this post.</p>
