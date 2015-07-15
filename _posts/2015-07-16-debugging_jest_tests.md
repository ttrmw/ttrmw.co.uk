---
title: "Debugging Jest Tests"
layout: post
date: 2015-07-16
categories: code
---

In Ruby tests, I frequently find myself stepping into a REPL when something doesn't behave as expected. With `pry` or similar, this is as simple as:

{% highlight ruby %}
debugger
{% endhighlight %}

So naturally, starting out with testing React components, I quickly found I missed this functionality, so here's how I tackled it.

In your `package.json` file, you will probably have something like:


{% highlight json %}
//...
"scripts": {
  "test": "node ./node_modules/jest-cli/bin.jest"
}
{% endhighlight %}

which points `npm test` at the Jest binary. We can run the same binary with `node debug`:

{% highlight sh %}
node debug ./node_modules/jest-cli/bin/jest.js
{% endhighlight %}

But you will likely encounter issues with jest's use of `harmonize`, so you'll need this flag:

{% highlight sh %}
node debug --harmony ./node_modules/jest-cli/bin/jest.js
{% endhighlight %}

Another gotcha - Jest runs tests in parallel, so for ease of debugging you will want to pass it the `-runInBand` flag:

{% highlight sh %}
node debug --harmony ./node_modules/jest-cli/bin/jest.js --runInBand
{% endhighlight %}

Awesome! This setup should allow you to add `debugger;` to your javascript files as you might with `Pry`.

Now, back to `package.json`! We could probably setup an alias for this in `.bashrc`, but it would certainly be neater to add it as an npm script!

{% highlight json %}
//...
"scripts": {
  //...
  "testdbg": "node debug --harmony ./node_modules/jest-cli/bin/jest.js"
}

{% endhighlight %}

NPM defines the `test` command that runs the script of the same name defined in `package.json`, however any other scripts should be run with:

{% highlight sh %}
npm run scriptname
{% endhighlight %}

So this script can be run with:

{% highlight sh %}
npm run testdbg
{% endhighlight %}

or, if you wish to specify a specific test file:

{% highlight sh %}
npm run testdbg -- TestFile.js
{% endhighlight %}

If you're coming from `Pry` and the like, you'll find node debug works as you'd expect, with `c` and `s` providing continue and step functionality, however if you need an interactive prompt in the current context, you must enter `repl`.

Happy debugging!


