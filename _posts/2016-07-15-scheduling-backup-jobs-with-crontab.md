---
layout: post
title: Scheduling Backup Jobs With Crontab
date: 2016-07-15 13:30:00 +5:30 GMT
share: y
img-header: "https://cdn-images-1.medium.com/max/800/1*wLnfZpiPjgClzrAeQfEbTQ.jpeg"
---

[Backup](http://backup.github.io/backup/v4/) is an insanely useful gem for, well
backups. I use it in production all the time. It can run independent of your
Rails app, which in in production can be quite helpful. I'm going to skip ahead
and assume your Backup jobs are already set.

<!--break-->

You can also [schedule backups](http://backup.github.io/backup/v4/scheduling-backups/)
however it requires using another gem, [Whenever](https://github.com/javan/whenever).
I don't like to add other dependencies on my production servers. So the easiest
way to over come that was to use Linux's built in scheduling system, [cron](https://help.ubuntu.com/community/CronHowto)!

> Cron is a system daemon used to execute desired tasks (in the background) at designated times.

### Using Cron
To start using cron just type in,

{% highlight bash %}
crontab -e
{% endhighlight %}

This opens up a file in which you can enter any jobs you want to execute periodically.
Each job starts on it's own line. The general rule is,

{% highlight bash %}
<periodic intervals> <system command>
{% endhighlight %}

### Scheduling Backup
To periodically run your Backup jobs, we simply need to add the command to the crontab.
But before that, we need to find the right Backup gem executable, that's easy to do.
Just run,

{% highlight bash %}
which backup
# /usr/local/bin/backup
{% endhighlight %}

Save the executable location, we'll be using it next.

Now open up your cron tab and add,

{% highlight bash %}
0 */6 * * * /bin/bash -l -c '/usr/local/bin/backup perform --trigger mysql_backup'
{% endhighlight %}

This runs your Backup job every 6 hours! Simple! And no extra gems needed.
