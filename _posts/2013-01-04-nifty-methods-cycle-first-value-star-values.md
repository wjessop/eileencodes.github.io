---
layout: post
title: "Nifty Methods: cycle(first_value, *values)"
date: 2013-01-04
categories:
  - learning-rails
  - tips
  - nifty-methods
---

<p>Recently I was working on an app and needed to "cycle" through some values. Previously, the code was using <code>each_with_index</code> and using that index to figure out where the "odd" or "even" class was applied to the table rows. Since I needed to change the code to use <code>map</code> I could no longer use <code>each_with_index</code> effectively.</p>
<p>First my thinking was to use modulo to loop through and add the classes, but that seemed to be complete overkill for my situation.</p>
<p>I then turned to google to find something simpler. It seems Rails already has a dead simple solution. It amazes me every time I find something that I don't feel is necessary to write from scratch already in existence.</p>
<p>The method is <code>cycle(first_value, *values)</code>. This can be used for many things, but the most obvious is cycling through "even" and "odd" classes, but can also be used to more advanced arrays. The Rails API uses cycling through colors as an example. The method also can accept a Hash to create a named cycle.</p>
<p>If you're interested in reading more about this nifty little method, you can find more details in the <a href="http://api.rubyonrails.org/classes/ActionView/Helpers/TextHelper.html#method-i-cycle" target="_blank">Rails API</a>.</p>
