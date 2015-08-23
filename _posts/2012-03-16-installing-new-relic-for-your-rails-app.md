---
layout: post
title: "Installing New Relic for Your Rails App"
date: 2012-03-16
categories:
  - learning-rails
  - shoutouts
---

<h3>What is New Relic?</h3>
<p><a href="http://newrelic.com/" target="_blank">New Relic</a> is a performance management system that our company just started using to monitor some of our larger apps. Since this app occasionally drops mysql connection (I'm thinking it's related to Unicorn) we're using it as a test case.</p>
<p>By using New Relic you can get real-time browser performance, app performance, database performance, email notifications. And on top of being a great resource it looks awesome. Some really nice design there.</p>
<h3>Installation and Use</h3>
<p>Installation for Rails Apps is easy. Add the gem to your Gemfile inside your production group.</p>
<p><code>gem 'newrelic_rpm'</code></p>
<p>When you're done with that add <code>newrelic.yml</code> Inside your <code>config/</code> folder inside your app. You can get the default settings from the <a href="https://github.com/newrelic/rpm/blob/master/newrelic.yml" target="_blank">github project</a>.</p>
<p>You'll need a license key and to change your group's based on your configurations if you don't want the default options.</p>
<p>Once deployed New Relic automatically recognizes your Rails App and begins monitoring it. It monitors who is visiting your site and from what country as well as all the processes it is running. Definitely worth checking out.</p>
