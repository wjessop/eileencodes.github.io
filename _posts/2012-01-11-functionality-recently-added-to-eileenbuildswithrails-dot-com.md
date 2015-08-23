---
layout: post
title: "Functionality Recently Added to eileenbuildswithrails.com"
date: 2012-01-11
categories:
  - learning-rails
  - cms-functionality
---

<p>I've been working on this blog in bits and pieces, when I have time. I've added some new functionality to bring it closer to a real blog CMS.</p>
<h3>Adding Published at date changer</h3>
<p>First I added a Published at date changer so that I could change my post dates if necessary. I think I'm going to do a "definitive list of RoR helper methods" because if it weren't for my pledge to watch all the railscasts videos in under a year I wouldn't have know about <code>date_select</code> and <code>datetime_select</code> and I would have been putting my head through a wall trying to figure it out. I feels almost like Rails cares about my sanity!</p>
<p>To use this helper method make <code>:created_at</code> <code>attr_accessible</code>in your Post model and then add the following to your admin view:

{% highlight erb %}
<div class="field">
   <%= f.label :created_at, "Published at" %>
   <div class="datetime">
      <%= f.datetime_select :created_at %>
   </div>
</div>
{% endhighlight %}

</p>
<h3>Belongs_to user for posts</h3>
<p>I figured in the future I may have some guest bloggers, and it would be nice now to have all the posts have an owner so I woundn't need to go back and add it later.</p>
<p>First I created a migration so that would add the current users ID to the post table. Then I updated the post and user model adding <code>belongs_to :user</code> to the post model and <code>has_many :posts</code> to the user model.</p>
<p>Next, I wrote a method so I could call it in multiple actions if I needed to (for at least the first three posts I wanted to call the method on update to add the user to the table, also I only have one user so I knew nothing would happen).</p>

{% highlight ruby %}
class Admin::PostsController < Admin::AdminController

  #only showing the update method
  def update
    @post = Post.find(params[:id])
    
    if @post.valid?
      add_author_to_post
      save_or_remove_post_categorization
    end
     
    respond_to do |format|
      if @post.update_attributes(params[:post])
        format.html { redirect_to [ :edit_admin_post ], :notice => 'Post was successfully updated.' }
      else
        format.html { render :action => "edit" }
      end
    end
  end
  
  #method created for adding user to posts table
  def add_author_to_post
     @user = User.find_by_id(session[:user_id])
     @user.posts << @post
  end
end
{% endhighlight %}

<p>In my view I was getting my most hated errror "You have a nil object when you didn't expect it". Yea Rails, if I expected it we wouldn't be haivng this conversation...</p>
<p>Anyway, I quickly figured it out was because I did not already have my author filled in so of course rails was complaining. Here is my view for the author section.</p>

{% highlight erb %}
<div class="datetime">
   <% if @post.user.nil? %>
      No author specified
   <% else %>
      <%= @post.user.username %>
   <% end %>
</div>
{% endhighlight %}

<h3>The Future</h3>
<p>The next stesp will be adding the a lot more functionality and I will post on them as I do. I want to add:</p>
<ul>
<li>The ability for comments</li>
<li>Ability to create draft, and future posts</li>
<li>Contact Form</li>
<li>Custom sidebar twitter feed</li>
<li>Post tweets from admin for new post notification</li>
<li>Login and profiles for guest bloggers</li>
</ul>
<p>Functionality way further down that line but will be really awesome</p>
<ul>
<li>Front-end customization from admin area</li>
<li>Creation and reorder for dynamic menus (think WordPress drag and drop menus)</li>
<li>Ability to add and customize modules</li>
<li>And much more...</li>
</ul>
