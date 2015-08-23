---
layout: post
title: "Getting Your Local Environment Setup to Contribute to Rails"
date: 2015-03-01 18:56:00
categories:
  - learning-rails
  - tips
  - open-source
---

At [RailsConf][railsconf]{:target="_blank"} I'll be leading a [workshop][workshop]{:target="_blank"} on contributing to Ruby on Rails called "Breaking Down the Barrier: Demystifying Contributing to Rails". My goal is to help you be confident in your ability to contribute to Rails. I'll be focusing on contributing guidelines, advanced git commands, and traversing unfamiliar source code. I've allotted 90 minutes for the workshop so in order for you to get the most out of it you should have your system ready. In this post I'm going to go over the basics of getting set up.

Technically, the easiest way to get Rails running locally is to use the supplied VM. I prefer to have it running on my local machine, but if you're using Windows I highly recommend [using the VM][use-vm]{:target="_blank"}. Although the VM is referred to the "easy way", you likely already have 50% of the things you need installed on your system already if you're actively developing Rails applications. If you do decide to use the VM you can skip these instructions. Please contact me on {{ site.twitter_username }} to let me know if the VM directions are wrong. Or better yet, if you figure it out send a pull request!

Let's get started with the basics of getting set up!

### Ruby Manager

When working with Rails it's likely you'll be using a different Ruby version than you use in your production applications. It's best to use Rails master with the most up-to-date version of Ruby. Currently, you can't use Rails master / future Rails 5 without Ruby 2.2.2. You probably already have a way to have multiple rubies installed on your machine with either [rbenv][rbenv]{:target="_blank"}, [rvm][rvm]{:target="_blank"} or [chruby][chruby]{:target="_blank"}.

I personally use rbenv but I've used rvm in the past and hear good things about chruby. It really doesn't matter which one you use as long as you can have multiple rubies installed on your machine.

Once you have that set up, install Ruby <del>2.2.1</del> 2.2.2 (2.2.1 had a security vulnerability). Don't forget if you have a new version of Ruby you'll need to install bundler before you run bundle install on the Rails repo.

### Fork & Clone Rails

Now we'll get the Rails source code set up. First go to [github.com/rails/rails][rails-repo]{:target="_blank"} and click "fork". Some developers prefer to clone the main Rails repo and set up their fork as an upstream, but unless you have push rights to Rails (a commit bit) I don't think this really makes sense. In my opinion having origin set as your repo works best so this guide is going to show you my preferred method.

Checkout your version of Rails to your local machine:

<code>$ git clone git@github.com/your-fork/rails.git</code>

You'll need to get Rails master main repo as an upstream to your Rails. To do this simply run:

<code>$ git remote add upstream https://github.com/rails/rails.git</code>

Anytime you want to pull changes from Rails master into your master do:

{% highlight text %}
$ git pull --rebase upstream master
$ git push origin master
{% endhighlight %}

Here you are pushing to your origin so your remote origin is always up-to-date with your master branch. Don't work on your master branch and send PR's from there. Always create a new branch. That way you can be working on multiple patches and your master is always clean and ready to checkout a new branch from. Pushing to your origin master also makes it easy to reset any of your branches to master without having to re-pull changes from upstream Rails.

Don't forget to add a <code>.ruby-version</code> file to your Rails repo, but be sure not to check this in. I have a <code>.gitignore_global</code> file that sits in my home directory and ignores all <code>.ruby-version</code> files. Then you should <code>run bundle install</code>.

### Databases

Ok now that you've got the Ruby and Rails source, you'll need to get a few more things installed before you can start running Rails tests. And the most important of those things is databases!

It's not really a requirement that you have ALL the databases installed but it's a good idea to have the default databases that the Active Record supports; SQLite3, MySQL, and PostgreSQL. This will help you test the main adapters that are supported in Rails. It's also a good idea if you're working on any SQL specific parts of Active Record; you want to be sure you aren't negatively changing the behavior of those other databases.

How you install databases is up to you. As a OS X user I install them with [homebrew][homebrew]{:target="_blank"} and follow the instructions output after installation. Remembering all the start/stop commands for databases is a pain though so I use [LaunchRocket][launchr]{:target="_blank"} to control this. It's a OS X preference pane to manage databases installed with homebrew. Additionally, you'll need memcached for some ActionDispatch and ActionController tests.

Once you have the necessary databases installed you'll need to create the databases and users required by the Rails tests.

#### MySQL

First create the users

{% highlight text %}
$ mysql -uroot -p

mysql> CREATE USER 'rails'@'localhost';
mysql> GRANT ALL PRIVILEGES ON activerecord_unittest.*
       to 'rails'@'localhost';
mysql> GRANT ALL PRIVILEGES ON activerecord_unittest2.*
       to 'rails'@'localhost';
mysql> GRANT ALL PRIVILEGES ON inexistent_activerecord_unittest.*
       to 'rails'@'localhost';
{% endhighlight %}

Then create the databases

{% highlight text %}
$ cd activerecord
$ bundle exec rake db:mysql:build
{% endhighlight %}

#### PostgreSQL

If you're a Linux user create the user by running:

<code>$ sudo -u postgres createuser --superuser $USER</code>

If you're an OS X user run:

<code>$ createuser --superuser $USER</code>

And then create the databases:

{% highlight text %}
$ cd activerecord
$ bundle exec rake db:postgresql:build
{% endhighlight %}

#### Creating and Destroying

It's also possible to create both MySQL and PostgresSQL databases at the same time by running:

{% highlight text %}
$ cd activerecord
$ bundle exec rake db:create
{% endhighlight %}

And you can destroy the databases and start over with:

{% highlight text %}
$ cd activerecord
$ bundle exec rake db:drop
{% endhighlight %}

### Running the Tests

Now that you have Ruby, the Rails source code, and the databases installed it's time to run the tests. Now don't just run rake test in the Rails root directory because you will be there all day waiting for railties tests to finish. Simply cd into the library you want to test and run:

<code>$ rake test</code>

To run Active Record tests, be sure to include the database adapter you want to test or else sqlite3, mysql, mysql2, and postgresql adapter tests will all run. To run tests for specific adapters do the following:

{% highlight text %}
$ bundle exec rake test:sqlite3
$ bundle exec rake test:mysql
$ bundle exec rake test:mysql2
$ bundle exec rake test:postgresql
{% endhighlight %}

And don't forget all these commands are available if you run <code>rake -T</code> in the directory you're in.

### See You at the Workshop

I'd tell you more about contributing to Rails but then I would have nothing to talk about at the workshop! I know it will be a lot of fun and you'll learn tons. To read more about my workshop go to the [RailsConf website][workshop]{:target="_blank"}.

If you have any issues at all getting set up ping me on twitter at {{ site.twitter_username }} and I'll do my best to point you in the right direction.

[railsconf]: http://railsconf.com
[workshop]: http://railsconf.com/program/labs#prop_879
[use-vm]: https://github.com/rails/rails-dev-box
[rbenv]: https://github.com/sstephenson/rbenv
[rvm]: https://rvm.io/
[chruby]: https://github.com/postmodern/chruby
[homebrew]: http://brew.sh/
[launchr]: https://github.com/jimbojsb/launchrocket
