---
layout: post
title: "Now I've Got Pretty Permalinks and Much More"
date: 2012-08-22
categories:
  - tips
  - nifty-methods
  - gems
---

<p>Note: This blog post referred to an older version of this blog that I had built from scratch. I've attempted to update links so they don't 404, but know that this post is not currently accurate.</p>

<p>I'm off to a pretty good start with my yearly goals. It may not look like it but I've made a lot of changes under the hood to this blog. I was finding my ability to find myself on google pretty low so I added a <a href="/sitemap.xml">sitemap</a> &mdash; and it's not just any sitemap, it auto-updates using sweepers. In the next post I'll go over how to build the sitemap specifically.</p>
<p>A couple other new features are "pretty permalinks" and an <a href="/feed.xml">RSS Feed</a>. For the permalinks I used the <a href="https://github.com/rsl/stringex" target="_blank">Stringex</a> gem. The gem allows you to more easily control your permalinks and comes with a nifty little method that allows you to create the URL's for already existing posts. I then made sure I added rewrite rules to my <code>nginx</code> config so that the old <code>/posts/1</code> urls still functioned. Nginx rewrite rules are a little different from Apache as you can see in my exmaple below:</p>
<p><code>rewrite ^/posts/1(/)?$ /posts/hello-world permanent;</code></p>
<p>Lastly, I updated my 404, 422, and 500 pages to actually look like my site. Yipee!. I should have done that earlier, but when I built eileenbuildswithrails.com I wanted to get it live before the New Year.</p>
<p>Next set of changes will involve adding pages to the cms, including a contact form, ability to add images, and perhaps comments. Thanks for reading!</p>
