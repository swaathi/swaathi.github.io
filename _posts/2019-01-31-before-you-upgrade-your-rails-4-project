---
layout: post
title: Before you upgrade your Rails 4 project to Rails 5
date: 2019-01-31 16:00:00 +5:30 GMT
share: y
---

I finally, FINALLY moved my old Rails projects stuck in Rails 4 to Rails 5. I've been dreading doing this, since it is such a painful process. But a new year is meant for new things (or old things, depending on how you look at this).

<!--break-->

So here are the links that helped me do this,

1. https://www.ombulabs.com/blog/rails/upgrades/upgrade-rails-from-4-2-to-5-0.html
2. https://stackoverflow.com/a/39636093/2572999
3. https://stackoverflow.com/questions/28006358/undefined-method-raise-in-transactional-callbacks-for-activerecordbaseclass

I also had a terrible time updating my gems when I moved to Rails 5. The problem exists in the `development` block of your Gemfile. Gems like `rspec-rails` and `quiet_assets` are no longer required in Rails 5 and instead of quietly failing, it causes so much of drama! My advice would be to create a dummy Rails 5 project and use that projects Gemfile as a reference.

And finally if you have written tests, you're in good hands. If not, good luck to you!

So, have a great time folks! (NOT)
