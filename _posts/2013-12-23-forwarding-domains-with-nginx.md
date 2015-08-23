---
layout: post
title: "Forwarding Domains with Nginx"
date: 2013-12-23
categories: nginx
---

<p>Recently I decided to switch the url of this blog from eileenbuildswithrails.com to eileencodes.com. If you're curious about why I changed the name read this <a href="/posts/name-changes-domain-changes-oh-my">post</a>.</p>
<p>With this change I wanted to keep eileenbuildswithrails.com, but forward it to eileencodes.com. Since nginx handles domain names based on alphabetical order the default domain was not forwarding correctly. The key is to forward the domains in the config files themselves.</p>
<p>Setting up the redirect in nginx is really easy once you know how to do it. Simply set up two server definitions; one to forward the old domain to the new domain, and one to handle rewrite of the new domain to www. And that's it. See below:</p>

{% highlight text %}
server {
  listen 80;
  server_name olddomain.com www.olddomain.com;
  rewrite ^(.*) http://www.newdomain.com$request_uri permanent;
}
server {
  listen 80;
  server_name newdomain.com;
  rewrite ^ http://www.newdomain.com$request_uri permanent;
}
{% endhighlight %}
