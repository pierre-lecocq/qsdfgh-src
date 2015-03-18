---
layout: post
title:  "A firewall with iptables"
excerpt_separator: <!--excerpt-->
date:   2014-10-02
categories: articles
tags:
- firewall
- security
---

How to build a simple but efficient firewall with iptables on Linux.
<!--excerpt-->

# The goal

The purpose is to setup a simple firewall thanks to **iptables** on Linux.

It features:

- Block everything from outside
- Allow everything from inside
- Allow incoming trafic from a specific IP range (SSH)
- Allow incoming trafic from every sources HTTP

Simple, nothing fancy, but efficient. Just what we need.

# Prerequisites

There is one package to install in order to test and set your firewall permanently

{% highlight sh %}
sudo apt-get install iptables-persistent
{% endhighlight %}

* The script

Create `/etc/iptables.custom.rules` and add this (adapt to your needs, of course):

{% highlight sh %}
*filter

##################################################################
# Set up

# Flush and empty chain
-F
-X

# Drop everything
-P INPUT DROP
-P FORWARD DROP
-P OUTPUT DROP

# Do not mess with established connections
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# Authorize loopback
-A INPUT -i lo -j ACCEPT
-A OUTPUT -o lo -j ACCEPT

##################################################################
# Basics

# ICMP - for ping (in & out)
-A INPUT -p icmp -j ACCEPT
-A OUTPUT -p icmp -j ACCEPT

# DNS (in & out)
-A OUTPUT -p tcp --dport 53 -j ACCEPT
-A OUTPUT -p udp --dport 53 -j ACCEPT
-A INPUT -p tcp --dport 53 -j ACCEPT
-A INPUT -p udp --dport 53 -j ACCEPT

# NTP (out)
-A OUTPUT -p udp --dport 123 -j ACCEPT

##################################################################
# Services

# SSH
-A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT
-A OUTPUT -p tcp --dport 22 -j ACCEPT

# HTTP (in & out)
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
-A OUTPUT -p tcp --dport 80 -j ACCEPT
-A OUTPUT -p tcp --dport 443 -j ACCEPT

# SQL (in & out)
-A INPUT -p tcp --dport 5432 -s 192.168.1.0/24 -j ACCEPT
-A OUTPUT -p tcp --dport 5432 -j ACCEPT

##################################################################
# Commit

COMMIT
{% endhighlight %}

# Test the rules

In order to test the rules written above, simply use this command (as root):

{% highlight sh %}
iptables-restore < /etc/iptables.custom.rules
{% endhighlight %}

Check the result with:

{% highlight sh %}
iptables -L
{% endhighlight %}

If you modify the rules after this, you can save them into the original file:

{% highlight sh %}
iptables-save > /etc/iptables.custom.rules
{% endhighlight %}

# Make it permanent

Once everything is tested and work as you want, create `/etc/network/if-pre-up.d/iptables` and add:

{% highlight sh %}
#!/bin/sh

/sbin/iptables-restore < /etc/iptables.custom.rules
{% endhighlight %}

And make it executable:

{% highlight sh %}
chmod +x /etc/network/if-pre-up.d/iptables
{% endhighlight %}

And voilà!
