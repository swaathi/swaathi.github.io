---
layout: post
title: Redirecting Devise Users To Their Last Visited Page
date: 2016-09-25 08:30:00 +5:30 GMT
share: y
---

It's always a nice gesture to redirect your users to the page they were last visiting. Imagine a scenario where your app sends out mailers with a link to a page that requires a login to see. You can't assume that your user would always be logged in, so when he clicks on this, he'd be redirected to the sign in page. He then signs in and... he's back in the homepage. So he leaves. Retention 0, distrust 1.

<!--break-->

What should happen in this scenario is, the app should automatically redirect the user to the page he was supposed to look at before he got whisked away. There's a lot of ways to do this, and a few ways even recommend persisting the user location to your database. YUCK. Why would you want to make a database call for every page the user visits? What would happen when your app scales? Or what would happen when your user isn't even in the system, where would you persist that data, so that when he does eventually sign up, you can provide a seamless transition? Persisting something as volatile as user location is not a good idea.

Enter session.

> A session is defined as a series of related browser requests that come from the same client during a certain time period. - [Google](https://www.google.co.in/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=what+is+a+session+in+a+web+application) (Actually [Oracle](https://docs.oracle.com/cd/E13222_01/wls/docs81/webapp/sessions.html))

And guess what? The session remains the same for a user who is not logged in, to when he logs in. Aaaand guess what else? You can persist data on your session!

Here's what you can do to achieve this,

### 1. Persist user location to the session.

{% highlight rb %}
  class ApplicationController < ActionController::Base
    protect_from_forgery with: :exception, prepend: true

    before_action :persist_last_visited_path, :authenticate_user!, except: [:home]

    def home
    end

    private
    def persist_last_visited_path
      unless Rails.configuration.ignored_paths.include?(request.path) && request.xhr?
        session[:last_visited_path] = request.path
      end
    end

  end
{% end highlight %}

Any location a user visits in your app, has to go through your `ApplicationController`. So it's a good idea to add a single `before_action` here to persist it. I like to filter out the paths I persist, and those usually include Devise default paths. Since I don't want my user to be in an endless loop after he signs up/in. (User signs up, session persists and he's redirected back to sign up, not good!) Also, I block out XHR requests.

Add a list of ignored paths to your `application.rb`. You can even create an initializer for this.

{% highlight rb %}
  # These paths will be ignored when redirecting the user to last visited page
  # Devise routes need to always be here, so that a redirect loop does not occur
  # after signing in
  config.ignored_paths = %W(/users/sign_in /users/sign_up /users/password /users/sign_out /users/confirm_password)
{% end highlight %}

### 2. Redirect the user if the session variable exists.

Add this to your ApplicationController too,

{% highlight rb %}
  def after_sign_in_path_for(resource)
    if session[:last_visited_path].present?
      session[:last_visited_path]
    else
      root_path
    end
  end
{% end highlight %}

The `after_sign_in_path_for` method is automatically called by Devise once the user logs in. it's a simple check to see if the session variable exists, redirect the user to his last location, or default to `root_path`.

That's it! A simple 2 minute job can really up your users experience.

---
We deeply care about user experience at [Allt](https://allt.in). You're never lost here. Sign up for your free trial now!
