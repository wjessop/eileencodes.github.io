---
layout: post
title: "Rails 5: ActionController::Parameters Now Returns an Object Instead of a Hash"
date: 2015-09-20
categories:
  - rails-5
---

<aside>As we work towards the release of Rails 5 there are a lot of changes will affect your application when you choose to upgrade. To help you keep up with the rapid development and changes I'm starting a series of blog posts to help you be ready to upgrade when that time comes.</aside>

A big change coming is how `ActionController::Parameters` works. `ActionController::Parameters` is where all of your `params` come from for your controllers. Calling `params` used to return a hash, but now will return an object.

Note: this doesn't affect accessing the keys in the params hash like `params[:id]`. You can view the PR that implemented this change here: [https://github.com/rails/rails/pull/20868](https://github.com/rails/rails/pull/20868){:target="_blank"}

To access the parameters in the object you can add `#to_h` to the parameters:

{% highlight ruby %}
params.to_h
{% endhighlight %}

If those params aren't explictly permitted you will be returned a hash with only the permitted parameters. If none are permitted you'll get an empy hash (`{}`). This comes in where you may be running `#symbolize_keys` or `#slice` on unpermitted `params`. If you're accessing `params` that aren't being saved to a model/db then you probably aren't explictly permitting those parameters.

If we look at the `#to_h` method in `ActionController::Parameters` we can see it checks if the parameters are permitted before converting them to a hash.

{% highlight ruby %}
# actionpack/lib/action_controller/metal/strong_parameters.rb
def to_h
  if permitted?
    @parameters.to_h
  else
    slice(*self.class.always_permitted_parameters).permit!.to_h
  end
end
{% endhighlight %}

Let's take an example where we are slicing params to use later. If we have this method that slices params we used to be able to write:

{% highlight ruby %}
def do_something_with_params
  params.slice(:param_1, :param_2)
end
{% endhighlight %}

Which would return:

{% highlight ruby %}
  { :param_1 => "a", :param_2 => "2" }
{% endhighlight %}

But now that will return an `ActionController::Parameters` object.

Calling `#to_h` on this would return an empty hash because `param_1` and `param_2` aren't permitted.

To get access to the params from `ActionController::Parameters` you need to first permit the params and then call `#to_h` on the object. The following returns the same thing as slice did previously.

{% highlight ruby %}
def do_something_with_params
  params.permit([:param_1, :param_2]).to_h
end
{% endhighlight %}

Another way to do this would be to use `#to_unsafe_hash` if you know the params are not user supplied and are safe:

{% highlight ruby %}
def do_something_with_params
  params.to_unsafe_h.slice(:param_1, :param_2)
end
{% endhighlight %}

By default `controller` and `action` parameters are allowed. To explicitly always allow other parameters you can set a configuration option in your `application.rb` that allows those parameters. Note: this will return the hash with string keys, not symbol keys.

Config option:

{% highlight ruby %}
  config.always_permitted_parameters = %w( controller action param_1 param_2 )
{% endhighlight %}

Calling slice on the parameters:

{% highlight ruby %}
def do_something_with_params
  params.slice("param_1", "param_2").to_h
end
{% endhighlight %}

If you're not sure when you'll have time to upgrade it would be a good idea to write some tests for your controllers that access the `params`. That way when you do upgrade you'll know to fix the params because your tests will be failing.
