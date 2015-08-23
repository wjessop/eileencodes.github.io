---
layout: post
title: "Installing Airbrake for Your Rails App"
date: 2012-03-19
categories:
  - learning-rails
  - shoutouts
---

<h3>What is Airbrake</h3>
<p>In addition to New Relic I have also added <a href="https://airbrake.io" target="_blank">Airbrake</a> to my blog. Again, Airbrake is very simple to add add to any Rails App.</p>
<p>The Airbrake App tells you if any of your projects throw and error, and also notifies you which of your app environments did so. You have the option of the error notifying you via email, only receive emails for the production environment or to participate in Airbrake's beta program.</p>
<p>It also has Lighthouse &amp; Github integration as well as tracking your deployments which is nice and convenient.</p>
<h3>Installation and Use</h3>
<p>Adding the Airbrake app is easy and I'll even go over an error and solution that occurred for me. First create an airbrake account and create your first project. When you create the project directions for adding Airbrake to your app will be visible.</p>
<p>Add airbrake to your Gemfile. I found this did not play nicely when I put the app in only my production environment.</p>
<p><code>gem 'airbrake'</code></p>
<p>Run <code>bundle install</code>. Then run <code>script/rails generate airbrake --api-key YOUR_API_KEY_HERE</code>. You can get your api key from that first screen when you create your first project. If this runs successfully you will see the airbrake test run in your terminal, followed by your configuration options which contain your api key, etc.</p>

{% highlight text %}
      append  config/deploy.rb
      create  config/initializers/airbrake.rb
         run  rake airbrake:test --trace from "."
** Invoke airbrake:test (first_time)
** Invoke environment (first_time)
** Execute environment
** Execute airbrake:test
Configuration:
{% endhighlight %}
<p>Now if you're using <a href="https://github.com/capistrano/capistrano/wiki/" target="_blank">Capistrano</a> you're probably wondering if there is anything special you need to do for your deploy. The good news is that Airbrake will add <code>require 'airbrake/capistrano'</code> to your deploy.rb automatically. The bad news is that my capistrano script broke after the addition of airbrake. Previous apps we have Airbrake installed on run fine so I'm not sure if it's certain versions of gems, ruby or rails that throws the errors.</p>
<p>If you see an error that says <code>`require': no such file to load -- rvm/capistrano (LoadError)"</code> then you probably have the same problem I have. After a little bit of research I found others had the same problem and reported that removing <code>require './config/boot'</code> from the deploy.rb fixed the problem. Once I removed this it ran fine and airbrake noted my deploy. The problem I found is a know issue reported on github as Issue #26 - if you'd like to follow the progress of this error you can read up on it <a href="https://github.com/airbrake/airbrake/issues/26" target="_blank">here</a>.</p>
