---
layout: post
title: "Those that do not heed the star are bound to execute it repeatedly..."
date: 2014-06-30 21:15:54 +0200
comments: true
categories:
    - automation
    - shell
    - linux
keywords: cron, gotcha, ubuntu
published: true
---

I guess everyone of us is dealing with Cron in some way or other, and I guess everyone has a horror story or two.

Mine is a simple tale of woe, ignorance and well, RTFM.
<!--more-->
{% codeblock lang:bash %}
{
    * */1 * * * /path/to/some/script.sh
}
{% endcodeblock %}

Do you know what the above cron job does? Or to be more precise how often is the job executed?

I thought that it executes the script every hour, and if you thought like me, you are most likely wrong.
My specific circumstances revolve around [Ubuntu][ubuntu], and as usual YMMV.

Turns out that the cron above does indeed execute the job every hour, only problem is that instead of once, it executes it 60 times.
Yes, that means every minute it would be run, fun isn't it.

If you have something demanding that needs to be run, you are in for a not-so-fun ride.
Yes you can use some sort of lock.sh to prevent multiple concurrent runs, but sooner or later you will be out of memory.

So what would be a solution to the problem, rather simple actually.

{% codeblock lang:bash %}
{
    0 */1 * * * /path/to/some/script.sh
}
{% endcodeblock %}

See that zero at the beginning, it tels the cron to run the script on the 0 minute of every hour, thus making sure that it gets executed only once.

Suddenly server has more resources, and scripts are not thrashing as much.

[ubuntu]:{www.ubuntu.com}