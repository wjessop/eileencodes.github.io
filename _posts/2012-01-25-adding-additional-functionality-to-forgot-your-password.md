---
layout: post
title: "Adding Additional Functionality to \"Forgot your password?\""
date: 2012-01-25
categories:
  - learning-rails
---

<p>Recently I needed to add "forgot your password?" to a Rails app built by my company. I had never done this before so I watched the <a href="http://railscasts.com/episodes/274-remember-me-reset-password" target="_blank">Remember Me &amp; Reset Password</a> Railscasts screencast. It was infinitely helpful, but I added a few things that were not in the video that I would like to share here. I followed the screencast for the most part: Adding a <code>password_resets</code> controller, in addition to following the <code>auth_token</code> directions. One thing I added was an intermittent page, instructing the user to check their email. I used the <code>index.html.haml</code>layout for this. In addition I did not like it that users who hadn't requested a new password could access these pages, so I created a few redirects to prevent that. My index and new action show this redirect protection.</p>
<p>If a user attemps to go to the password_resets while already logged in they will be redirected to their profile, if they try to get there without being logged in they will be redirected to the login page, same with the new action.</p>

{% highlight ruby %}
# index and new action in my password resets controller
def index
  if current_user
    redirect_to account_url
  else
    redirect_to '/login'
  end
end

def new
  if current_user
    redirect_to account_url
  end
end
{% endhighlight %}

<p>I added the functionality to check whether the user's email was in the database and to return an error if it was invalid.</p>
<p>On the create action I have the page rendering the index instead of redirecting to maintain access to the email address so the view can display "An email has been sent to "insert email address". You must click the link in the email to reset your password."</p>

{% highlight ruby %}
# create action for generate password token for forgot password
def create
  user = User.find_by_email(params[:email])
  if user
      user.generate_password_reset if user
      render :index
  else
    flash.now.alert = "No such email address was found. Please try again."
    render :action => 'new'
  end
end  
{% endhighlight %}

<p>Lastly, I wanted to make it more secure than the token expiring in 2 hours, so I added code that make the token expire if the user successfully changed their password. Once the user successfully changes their password <code>reset_pass_token </code>will be erased from the database and will be redirected to their account profile, so they can edit other settings if they like otherwise, they recieve errors. If the new password save isn't successful within 2 hours the token will also expire.</p>

{% highlight ruby %}
def edit
  @user = User.find_by_reset_pass_token(params[:id])

  if @user.nil?
    redirect_to '/login', :alert => 'Password reset does not exist.'
  elsif @user.reset_pass_expiration < 2.hours.ago
    redirect_to '/login ', :alert => "Password reset has expired."
  end
end

def update
  @user = User.find_by_reset_pass_token!(params[:id])
  if @user.update_attributes(params[:user])
    @user.reset_pass_token = nil
    @user.save!
    # if params are updated create session for user by matching token and pass
    if @user.valid?
      session[:user_id] = @user.id
      redirect_to account_url, :notice => "Your password has been reset!"
    else
      render 'update'
    end
  else
    flash.now[:error] = @user.errors.full_messages
    render :action => "edit"
  end
end
{% endhighlight %}

<p>The setup for "Forgot your password?" was rather easy and I learned a lot from implementing the functionality.</p>
