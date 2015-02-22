---
title:  "Building i3 gaps on Ubuntu"
layout: post
date:   2015-02-07
categories: linux
---

I've been using the [i3](http://i3wm.org/) window manager for a month or so now, and so far I've found it a great productivity boon. i3 is a tiling WM which provides a slick keyboard-driven workflow and heaps of configurability.

This post will help you build the i3-gaps variant from source, which adds this ability to create gaps between windows.

Out of the box, i3 doesn’t provide a means to ‘pad’ its containers with gaps as is present in, for example, [bspwm](https://github.com/baskerville/bspwm), but this is made available by the unofficial o4dev mirror.

![i3 - with gaps!](/img/i3_gaps.png)

I am personally a big fan of having a small delimiting space between my windows, and do not find the 'wasted' space to be a significant loss on modern, high resolution displays.

The [o4dev gaps branch](https://github.com/o4dev/i3) of i3 provides a simple configuration option for gaps in i3. Sadly, unlike vanilla i3, there is no package in the Ubuntu sources for i3-gaps.


###Building i3-gaps in Ubuntu
In order to build i3-gaps in Ubuntu, first clone the [repository](https://github.com/o4dev/i3). It is a good idea to clone this into /usr/local/src.

At this point you can run:

{% highlight sh %}
$ make && sudo make install
{% endhighlight %}

make will attempt to build i3, and if it is successful make install will install it.

At this point it is likely you will be greeted with an error highlighting a missing header (whatever.h) file that i3 depends on. On my system there were several such files missing, and the next section details how to go about getting them.



###Dependencies

Sometimes when building from source, the application will depend on libraries that may not be present on the target system.

The easiest way to find these files is using apt-file, a utility in the APT family that allows you to search packages for files, and list package contents without downloading them.

{% highlight sh %}
$ sudo apt-get install apt-file
{% endhighlight %}

apt-file requires that its database be populated before any searching can take place:

{% highlight sh %}
$ apt-file update
{% endhighlight %}

apt-file is now ready for searching. On my system, xcb/xcb.h was the first missing header file:


{% highlight sh %}
$ apt-file search xcb.h
libcairo2-dev: /usr/include/cairo/cairo-xcb.h
libx11-xcb-dev: /usr/include/X11/Xlib-xcb.h
libxcb1-dev: /usr/include/xcb/xcb.h
{% endhighlight %}

The last result - libxcb1-dev - has the required xcb/xcb.h file, and is logically named libxcb1-dev, implying that it is likely an xcb centric library.

Libraries are no different from other packages, as you may guess from their presence in the apt sources - accordingly, we can simply use:

{% highlight sh %}
$ sudo apt-get install libxcb1-dev
{% endhighlight %}

to grab the library. Next, we will want to rerun the make command, and rinse-repeat the above process for each missing dependency.



###Dependencies on Ubuntu 14.04LTS, 14.10

If you'd rather not go through the make, apt-file search, apt-get install process for each dependency, the table below details the packages I needed to successfully build i3-gaps.


| Header 	      | Package 	        		 |
|-----------------|------------------------------|
| xcb.h           | libxcb1-dev         		 |
| xcb_keysyms.h   | libxcb-keysyms1-dev 		 |
| pango.h         | libpango1.0-dev     		 |
| xcb_aux.h       | libxcb-util0-dev   			 |
| xcb_icccm.h     | libxcb-icccm4-dev   		 |
| yajl_gen.h      | libyajl-dev         		 |
| sn-launcher.h   | libstartup-notification0-dev |
| randr.h         | libxcb-randr0-dev            |
| ev.h            | libev-dev                    |
| xcb_cursor.h    | libxcb-cursor-dev            |
| xinerama.h      | libxcb-xinerama0-dev         |

{% highlight sh %}
$ sudo apt-get install libxcb1-dev libxcb-keysyms1-dev libpango1.0-dev libxcb-util0-dev libxcb-icccm4-dev libyajl-dev libstartup-notification0-dev libxcb-randr0-dev libev-dev libxcb-cursor-dev libxcb-xinerama0-dev
{% endhighlight %}

With all these installed, i3-gaps will now build and install successfully. For a nice minimal gap, add the following to your .i3/config file:

{% highlight sh %}
gap_size 10
{% endhighlight %}

One caveat - to enable this option you must disable title bars, as this causes them to glitch out pretty badly. This is easy, though:

{% highlight sh %}
new_window 1pixel
{% endhighlight %}
