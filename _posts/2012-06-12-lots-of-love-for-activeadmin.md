---
layout: post
title: "Lots of Love for ActiveAdmin"
date: 2012-06-12
categories:
  - learning-rails
  - tips
  - shoutouts
  - gems
  - administration
---

<p>I have been so heavily immersed in work projects that I haven't had the time to write posts about rails - or do much of anything else.</p>
<p>Recently, I have been using the <a href="http://activeadmin.info/" target="_blank">ActiveAdmin</a> gem for the websites I've been building with rails. We are rebuilding our <a href="http://www.evolvingmedia.net" target="_blank">company website</a> with rails and an ActiveAdmin backend. ActiveAdmin is awesome, the documentation could use a little love, but other than that I have no complaints about the system. I do believe everyone should build a rails administration system from scratch at least once to ensure understanding of how authentication works.</p>
<p>I'm going to go over some of the basics of getting up and running with ActiveAdmin and then in later posts will go into more detailed instructions on more complicated things you may want the admin to do like model relationships and integration with paperclip.</p>
<h3>Installation</h3>
<p>To get started add <code>activeadmin</code> and <code>meta_search</code> to your <code>Gemfile</code>. <code>Bundle install</code> and begin development! Activate the default admin user model by generating the resource: <code>rails generate active_admin:resource AdminUser</code>. The default login information is admin@example.com/password.</p>
<h3>Beginning Development</h3>
<p>First, the admin user model must be configured. An important thing to note on installation is that all the fields for the user are visible including encrypted_password, reset_password_token, etc and if you try to update/add a user before changing the available fields you will get a "mass assignment" error. Find the <code>admin_users.rb</code> in <code>app &gt; admin</code> and add the following:</p>

{% highlight ruby %}
ActiveAdmin.register AdminUser do
  index do
    column :id
    column :email
    column :full_name do |field|
      "#{field.first_name} #{field.last_name}"
    end
    column :last_sign_in_at
    default_actions
  end

  show do
    attributes_table do
      row :id
      row :full_name do |field|
        "#{field.first_name} #{field.last_name}"
      end
      row :email
      row :last_sign_in_at
      row :sign_in_count
    end
  end

  form do |f|
    f.inputs "Edit User" do
      f.input :first_name
      f.input :last_name
      f.input :email
      f.input :password
      f.input :password_confirmation
    end
    f.buttons
  end

  filter :id
  filter :email
end
{% endhighlight %}

<p>You'll notice I've added extra fields to the db, namely a first_name and last_name to create full_name. This file changes the main views in the administration interface &mdash; the index view (table of users), the show view (each user's settings), and the form (updating/adding users to the admin panel).</p>
<p>Be sure to update your model to include validations, messages and relationships.</p>
<p>Adding new models is as simple as generating a model and a resource. If you want to create a model that isn't updated through the ActiveAdmin interface, create a <code>has_many</code> join table model (ex, categorizations), but leave out the resource generation.</p>
<p>And that's all you need to get started, very simple gem to use. Although I could get a lot more in-depth about the features of ActiveAdmin I'm going to pause here. I encourage you to play around with it, the ease of use makes development even more fun. It's great to not have to worry about designing your admin interface for fast development.</p>
