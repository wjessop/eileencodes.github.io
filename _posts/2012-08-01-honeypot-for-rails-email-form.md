---
layout: post
title: "Honeypot for Rails Email Form"
date: 2012-08-01
categories:
  - learning-rails
  - tips
  - user-experience
---

<p>While working on my companies new website I realized that our contact form didn't have any spam protection and would become a favorite of bots not long after launch so I decided to add a honeypot.</p>
<p>If you're not familiar a honeypot is a hidden form that users can't see, but because bots don't read CSS they fill it in. If the form is filled in it automatically fails. This is a better alternative to the captcha because it doesn't require your users to type out a ridiculously illegible code (which from experience, i can tell you gets frustrating when you've gotten it wrong 60000 times).</p>
<p>I searched through a few gems and after being frustrated with the outcome decided to just do it myself because really it's just a few fields and the form is ajaxed and the gems would send a 200 response essentially removing the form, which was no my intention.</p>
<p>Open your form view and add a field for the honeypot. Be sure to add a label explaining it is a honeypot to ensure accessibility for screen readers and instructing users to not fill in the field.</p>
{% highlight erb %}
<div class="field sweet_honey_for_bots">
  <%= label_tag :sweet_honey, "This is a honeypot, if you see this you're CSS is turned off or you're a bot. If you're not a bot don't fill it in." %>
  <%= text_field_tag :sweet_honey %>
</div>
{% endhighlight %}

<p>Then go to your model and make your field accessible; <code>attr_accessible :sweet_honey</code></p>
<p>One last quick step is to instruct your form what to do on save. You can have it simply fail with no error but that isn't accessible plus if anyone does fill it in, you might want to give a fun little message for them. I have my form rendering a partial because of the ajax-iness, plus I don't want the form to disappear just in case it's not a bot.</p>
{% highlight ruby %}
if params[:sweet_honey].present?
    format.html { render :partial => "emails/email_bot", :layout => false }
elsif @email.save
    ...
else
    ...
end
{% endhighlight %}
<p>This is super simple implementation of a honeypot. I figured why make it more complicated when I just want to prevent send?</p>
