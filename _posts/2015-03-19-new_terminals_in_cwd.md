---
title: "Open new terminals in the current working directory"
layout: post
date: 2015-03-19
categories: linux
---


I noticed that with i3, I really wanted $mod+Return (the keybinding for opening a new terminal) to start terminal windows in the current working directory. There's an easy way to achieve this thanks to a program called [xcwd](https://github.com/schischi-a/xcwd).

xcwd prints the working directory of the currently focused window. This works to our advantage in i3, of course!

With urxvt, for example, we can specify the '-cd' option to open a new terminal in a specified directory, ie:

{% highlight sh %}
urxvt -cd 'somedir'
{% endhighlight %}

So, combined with xcwd, this becomes:

{% highlight sh %}
urxvt -cd '`xcwd`'
{% endhighlight %}

Where the backticks get the output from the xcwd program. This is a dynamic way to launch new terminals in the current working directory.

We can then set up our i3 config like so:

{% highlight sh %}
bindsym $mod+Return exec urxvt -cd "`xcwd`"
{% endhighlight %}

and $mod+Return now opens new terminals in the current working directory of the focused application.


