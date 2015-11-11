---
layout: post
title: "Rails 5: The Sprockets 4 Manifest"
date: 2015-11-11
categories:
  - rails-5
---

<aside>As we work towards the release of Rails 5 there are a lot of changes will affect your application when you choose to upgrade. To help you keep up with the rapid development and changes I'm starting a series of blog posts to help you be ready to upgrade when that time comes.</aside>

When Rails 5 is released it will require that you upgrade to Sprockets 4. Sprockets 4 has some big changes in how it knows what assets to compile so you will definitely have some updates to make if you rely on Sprockets within Rails. Since there are a lot of changes in Sprockets 4 I'll just be talking about the new `manifest.js` in this post.

With Sprockets you used to tell your `config/initializers/assets.rb` what assets were supposed to be precompiled. In Sprockets 4 you will use a `manifest.js` inside your `app/assets/` directory to tell Sprockets what assets to precompile. This behavior is actually available to use in Sprockets 3 but [sprockets-rails](https://github.com/rails/sprockets-rails/blob/93a45b1c463a063ec7cf4d160107b67aa3db7a1a/lib/sprockets/railtie.rb#L77-L81){:target="_blank"} has a conditional that only activates this in Sprockets 4.

{% highlight ruby %}
# sprockets-rails/lib/sprockets/railtie.rb

if using_sprockets4?
  config.assets.precompile  = %w( manifest.js )
else
  config.assets.precompile  = [LOOSE_APP_ASSETS, /(?:\/|\\|\A)application\.(css|js)$/]
end
{% endhighlight %}

<i>Note: the file type that `manifest.js` is stored as may change. There is disucssion around moving the `manifest.js` to `manifest.yml` because it doesn't make sense that the manifest has a file type that is unrelated to it's usage.</i>

So, how do you use a the new manifest file? In your `app/assets/` directory add a new directory named `config/`. Inside that folder add a file called `manifest.js`.

In the `manifest.js` you'll want to link your JS and CSS directories as well as any other directories you rely on like images, fonts, sounds etc.

Here is an example of a `manfiest.js` that links JS, CSS, fonts, and images.

{% highlight js %}
// JS and CSS bundles
//
//= link_directory ../javascripts .js
//= link_directory ../stylesheets .css


// Images and fonts so that views can link to them
//
//= link_tree ../fonts
//= link_tree ../images
{% endhighlight %}

Previously you didn't need to include fonts and images in your precompiled assets list and Sprockets Rails would use the `LOOSE_APP_ASSETS` constant to figure out those items, but now you have to explicitly include them in your config.

You'll notice that we use `link_directory` for CSS and JS, then the path from the config manifest file to the javascript "file". We compile `.coffee` and `.scss` down to `.js` and `.css`, respectively, so we denote that after the path to the JS and CSS files to tell Sprockets what to compile them into. Images and fonts don't change file type when compiled.

Once you've done that you'll need to remove `config.assets.precompile` from your `config/initializers/assets.rb`.

{% highlight ruby %}
config.assets.precompile += %w( 
  all.css all.js
)
{% endhighlight %}

Smaller apps may only have the `precompile` directive in their applications so in that case you can delete the `config/initializers/assets.rb` file. For larger apps, like Basecamp we have some extra settings regarding assets and didn't want to delete the config file.
