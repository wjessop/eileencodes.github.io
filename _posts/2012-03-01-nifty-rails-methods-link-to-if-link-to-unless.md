---
layout: post
title: "Nifty Rails Methods: link_to_if, link_to_unless"
date: 2012-03-01
categories:
  - learning-rails
  - nifty-methods
---

<p>I have for some time been wanting to make a "definitive list of awesome rails methods". There are so many great built in methods that I didn't realize existed when I was trying to build something from scratch. Sometimes it's annoying to find out that thing you're trying to build already exists - but if you haven't put 500 hours into it, it's usually a relief.</p>
<p>So today the featured definitve list of awesome rails methods are <code>link_to_if</code> and <code>link_to_unless</code>.</p>
<p>The nice thing about this method is it's great for changing links based on if a user is logged in, if the page is current, etc. Instead of writing an if else statement you can DRY it up by using <code>link_to_if</code> or <code>link_to_unless</code>. I found it fun to play around with and very useful as well. I personally used it for a complicated if else to mark a category for a prodcut as current. There is also an <code>link_to_unless_current</code> which I didn't end up using because it wouldn't mark both the current category and parent category as active, <code>link_to_if</code> worked much better for that.</p>
<p>Below is an example from the Rails API</p>

{% highlight erb %}
<%= 
  link_to_if(@current_user.nil?, "Login", { :controller => "sessions", :action => "new" }) do
     link_to(@current_user.login, { :controller => "accounts", :action => "show", :id => @current_user })
  end
%>
{% endhighlight %}
<p>More about these methods can be found on the <a href="http://api.rubyonrails.org/classes/ActionView/Helpers/UrlHelper.html#method-i-link_to_if" target="_blank">Rails API</a>.</p>
