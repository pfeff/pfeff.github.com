---
layout: post
title: Solution to "ansible could not find aptitude. please ensure it is installed"
---

# TL;DR: Install aptitude first

Typically, the first thing I do when provisioning a VM is upgrade all the installed packages.
In Bash on Ubuntu, this would mean running the apt-get upgrade dance.
In Ansible, it's this:

{% highlight yaml %}
- apt: cache_valid_time=3600 update_cache=yes upgrade=yes
{% endhighlight %}

If you do this, you'll probably see this error message:

    ansible could not find aptitude. please ensure it is installed

I thought I was stuck in a catch-22, but it's actually not a problem.
The apt module in ansible uses aptitude for bulk upgrades, everything else uses apt-get.

The solution: Use the apt module to install aptitude before running the upgrade.

