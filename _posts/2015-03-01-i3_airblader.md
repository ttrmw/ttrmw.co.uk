---
title:  "i3 Airblader Fork, building & config"
layout: post
date:   2015-03-01
categories: linux
---


The [Airblader Fork](https://github.com/Airblader/i3) of i3 provides more flexibility for gap configuration, eg separate inner and outer gap sizes.

You can use the same process as detailed [here](http://ttrmw.co.uk/linux/i3_gaps_ubuntu.html) to build the Airblader Fork on Ubuntu.

I needed a couple of extra dependencies to get up and running:

{% highlight sh %}
sudo apt-get install libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev
{% endhighlight %}

After running:

{% highlight sh %}
sudo make && sudo make install
{% endhighlight %}

and hitting shift+$mod+r, I instantly found my old i3-gaps config was incompatible.

The i3 gaps config item:

{% highlight sh %}
gap_size 10
{% endhighlight %}

is not valid in the Airblader fork. Instead, try:

{% highlight sh %}
gaps inner 10
gaps outer 10
{% endhighlight %}

For the same effect. I found I quite like a larger outer gap, however, eg:

![i3 - with moar gaps!](/img/airblader.png)

In addition to the static gap functionality available in the o4dev gaps branch, Airblader is able to do some clever modifications to its gaps in situ.


It accepts commands in the format:

{% highlight sh %}
gaps inner|outer current|all set|plus|minus <pixels>
{% endhighlight %}

(current/all referring to workspaces)

or, for workspace specific instruction:

{% highlight sh %}
workspace <identifier> gaps inner|outer <pixels>
{% endhighlight %}

Naturally, these can be added to your i3 config to really dial in your gap setup. But they can also be used to interesting on-the-fly effect!

Try this in a terminal:

{% highlight sh %}
$ i3-msg gaps outer current plus 100
{% endhighlight %}

Alright, not a particularly useful example, but you can see how you might be able to build clever keybound gap instructions, if this fits your use case.

By the by, i3-msg allows you to interact with i3 from a terminal, eg:

{% highlight sh %}
$ i3-msg exit
{% endhighlight %}

exits i3 and returns to light-dm (or equivalent).


The Airblader Fork adds a couple of other neat features including some additional configuration options for i3bar items. Check it out [here!](https://github.com/Airblader/i3)