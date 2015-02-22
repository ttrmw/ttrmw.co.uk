---
title:  "Brightness control in i3 with intel backlight & keymapping"
layout: post
date:   2015-02-20
categories: linux
---

This post details scripting brightness in Linux and keymapping the scripts in i3.

In i3 in Ubuntu, I found my laptop's Fn-combinations for screen brightness control were non functional out of the box.


###Scripting Screen Brightness in Linux
The backlight is exposed to the user through /sys/class/backlight, which will contain a directory named for the graphics card's vendor, in my case this was intel_backlight.

Whichever vendor the card is, the directory will contain

{% highlight sh %}
$ ls /sys/class/backlight/intel_backlight
actual_brightness  brightness  max_brightness  subsystem  uevent
{% endhighlight %}

The two interesting files here are brightness and max_brightness. Reading the latter will provide a maximum value for the former, writing to which will effect a change in screen brightness.

intel_brightness on my system had a very high max brightness:

{% highlight sh %}
../intel_brightness$ cat max_brightness
937
{% endhighlight %}

Which would allow quite a fine grained control. Other devices can be as low as 15.

In any case, a method for controlling the screen brightness becomes self evident - we can just rewrite the value in this file, between a maximum of the value in ./max_brightness, and a minimum of zero.

{% highlight sh %}
brightness=$(cat /sys/class/backlight/intel_backlight/brightness)

if (($brightness > 0)); then
  let brightness=$brightness-100
  echo "echo $brightness > /sys/class/backlight/intel_backlight/brightness" | sudo zsh #or bash
fi
{% endhighlight %}

The script grabs the value from the brightness file, and if it greater than 0, subtracts 100 from it, echoing the result back into the brightness file.

The echo line is a little weird here:

{% highlight sh %}
  echo "echo $brightness > /sys/class/backlight/intel_backlight/brightness" | sudo zsh #or bash
{% endhighlight %}

The brightness file is owned by root, so we need to escalate the echo privileges

In the obvious case one might try sudo echo. However, the shell that redirects its output into the brightness file runs without escalated privileges. Therefore, we instead pipe the output of the first echo ("echo $brightness >....") into a shell running with escalated privileges.


This script's counterpart increases the brightness:

{% highlight sh %}
max_brightness=$(cat /sys/class/backlight/intel_backlight/max_brightness)
brightness=$(cat /sys/class/backlight/intel_backlight/brightness)

if (($brightness < $max_brightness)); then
  let brightness=$brightness+100
  echo "echo $brightness > /sys/class/backlight/intel_backlight/brightness" | sudo zsh #or bash
fi
{% endhighlight %}

Here we must read the contents of max_brightness to determine an upper limit. Otherwise this script is much the same.

Great! So we can now save these somewhere, chmod +x them and test them out, eg:

{% highlight sh %}
~/scripts$ chmod +x bright_down
~/scripts$ ./bright_down
{% endhighlight %}

You will of course notice a change in screen brightness!



###Keybinding scripts in i3

First thing's first, we're going to need a practical way to run this script.

The script requires elevated permissions to write to the brightness file, but this is impractical for our use case.

We will need to add the script to the sudoers file, so that it can run with elevated permissions without prompting for the sudo password.

The sudoers file contains rules for command permissions, and lives in /etc/sudoers.

To edit the sudo file, it is important to use the visudo editor. This checks to make sure your sudoers file doesn't end up with any malformed input.

{% highlight sh %}
$ sudo visudo
{% endhighlight %}

and add

{% highlight sh %}
your_user your_machine = NOPASSWD: /path/to/script/bright_up
your_user your_machine = NOPASSWD: /path/to/script/bright_down
{% endhighlight %}

Now the scripts can be run with elevated permissions without the need for a password.


The next step is to bind it to a key combination or hardware button. This is easy! i3 makes configuring keybindings a breeze, such as:

{% highlight sh %}
bindsym Mod1+1 workspace 1
{% endhighlight %}

from the default i3 configuration.

here, the second parameter 'Mod1+1' is the key combination. We will need to find the keysym(s) for our intended keybinding, for which we can use xev.

Xev is an application for printing the contents of events from the X server. It will create a window, and print any events that happen to that window - this will allow us to view information about specific keypresses for example.

Go ahead and fire up xev

{% highlight sh %}
$ xev
{% endhighlight %}

You should see a white window, and switching focus to this should cause some output to appear in the originating terminal.

![xev!](/img/xev.png)

When focused on xev, go ahead and fire the key combination you wish to bind to. In the originating terminal, you should see something like:

{% highlight sh %}
...
KeyPress event, serial 35, synthetic NO, window 0x3600001,
    root 0x9d, subw 0x0, time 31609664, (533,520), root:(1345,532),
    state 0x0, keycode 232 (keysym 0x1008ff03, XF86MonBrightnessDown), same_screen YES,
    XLookupString gives 0 bytes:
    XmbLookupString gives 0 bytes:
    XFilterEvent returns: False

...
{% endhighlight %}

The keysym value can be used for keybindings in i3.

In my case, the default Fn combination yielded the XF86MonBrightnessUp/Down keysyms, so I will use those.


Finally, we just need to add the i3 config items:

{% highlight sh %}
bindsym XF86MonBrightnessDown exec --no-startup-id sudo bright_down
bindsym XF86MonBrightnessUp exec --no-startup-id sudo bright_up
{% endhighlight %}

exec tells i3 to execute a program, whilst --no-startup-id is a flag that prevents startup-notification support, which is ideal for our purpose as we are running a non-graphical program (the scripts).


###Providing feedback

It would be useful to provide some feedback as to the change in brightness. This could be done with i3bar - for example including the output of

{% highlight sh %}
cat /sys/class/backlight/intel_brightness/brightness
{% endhighlight %}

on the bar.

Another possible solution is to use notify-send, which is present in Ubuntu. We can add:

{% highlight sh %}
notify-send Brightness "${brightness}/${max_brightness}" -t 200
{% endhighlight %}

to the script directly after writing to the brightness file. This just outputs the current brightness out of the maximum brightness in a notification pop up with a short timeout.

The first argument is the notification title, the second the message. The -t flag sets the timeout in milliseconds.