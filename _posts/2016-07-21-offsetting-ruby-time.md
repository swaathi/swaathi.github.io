---
layout: post
title: Offsetting Ruby Time
date: 2016-07-21 23:00:00 +5:30 GMT
share: y
img-header: "https://cdn-images-1.medium.com/max/800/1*wLnfZpiPjgClzrAeQfEbTQ.jpeg"
---

Managing local time is such a huge challenge when making webapps. There's always
going to be users from all over the world clocking in on different timezones. Basecamp's
[local_time](https://github.com/basecamp/local_time) solves this problem in a elegant way –
it doesn't care how you store timezone, it interprets the users local time and prints it
out on the view accordingly.

But, all hell breaks loose when you try to recreate this
on the server side. Here there's no information on what user is from what timezone.
And even if you persist the users timezone information, surprisingly Rails doesn't
come with an out of the box solution to offseting time(s).

<!--break-->

Here's a super simple way to convert a UTC time to a zone of your choice, by just
inputting the offset.

{% highlight rb %}
  t = Time.now.utc
  => Fri, 21 Jul 2016 07:56:38 UTC +00:00
  t.offset("+05:30")
  => Fri, 21 Jul 2016 13:26:38 UTC +00:00
{% endhighlight %}


{% gist 4d20c8989172351268173c4f16421208 %}
