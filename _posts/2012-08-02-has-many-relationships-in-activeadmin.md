---
layout: post
title: ":has_many Relationships in ActiveAdmin"
date: 2012-08-02
categories:
  - learning-rails
  - tips
  - administration
---

<p>When I began using ActiveAdmin I discovered that there is not a lot of documentation on advanced usage such as displaying the :has_many relationships on the index and show page in the admin of the posts. I wanted to make it easy to add categories to posts and to know how posts were categorized easily.</p>
<p>Once the Category and Post models are created a join model also needs to exist, in this case Categorization. This part is the same as any other <code>:has_many</code>, the hard part is display in the admin.</p>
<p>Go into the admin view for Post and add the following (other fields removed for easy viewing):</p>

{% highlight ruby %}
index do
  column :title, :sortable => :title do |post|
    link_to post.title, [:edit_admin, post]
  end
  
  column :categories do |post|
    table_for post.categories.order('title ASC') do
      column do |category|
        link_to category.title, [ :admin, category ]
      end
    end
  end
  default_actions
end

show do
  attributes_table do
    row :title
    row :content
    table_for post.categories.order('title ASC') do
      column "Categories" do |category|
        link_to category.title, [ :admin, category ]
      end
    end
  end
end

form do |f|
  f.inputs "Add/Edit Post" do
    f.input :title
    f.input :content
    f.input :categories, :as => :check_boxes
  end
  f.buttons
end
{% endhighlight %}

<p>As you can see inside the tables creating a special inner table is required for best display. I add the link to the category edit in so it's easy to edit/view the category from the posts table.</p>
<p>To add checkboxes for category select it's as simple as adding <code>f.input :categories, :as => :check_boxes</code> to your form view.</p>
<p>Want to figure out how to set up active admin? Read my "<a href="/posts/lots-of-love-for-activeadmin" target="_blank">Lots of Love for ActiveAdmin</a>" post. If you have questions, comment on the <a href="https://gist.github.com/3240041" target="_blank">gist</a> on github or find me on <a href="http://www.twitter.com/eileencodes" target="_blank">twitter</a>.</p>
<p>ActiveAdmin is a really great simple way to implement an admin for a simple website or blog.</p>
