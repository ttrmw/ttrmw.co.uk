---
title: "Rails, React & npm modules with Browserify"
layout: post
date: 2015-06-24
categories: code
---

This post details setting up a rails application for development with react-rails and npm-modules. 

I have lately been working on an React application with a Rails backend. There was a need to implement drag & drop functionality, for which I wanted to use ReactDnD, a module that depends upon using commonJS style 'require' calls. 

After an aborted attempt to create an asset gem for ReactDnD, I settled upon using Browserify to make npm packages available as assets.


Install Node.js
=====

First, naturally, we will need Node! 

{% highlight sh %}
$ apt-get install nodejs
{% endhighlight %}

If you're not using an apt based distro, hit up [the node site](https://nodejs.org/) for a tar ball.

This should install npm:

{% highlight sh %} 
$ npm -v 
1.4.28
{% endhighlight %} 

Generate the Rails app
=====

If you're not adding to an extant app, go ahead and generate a new Rails application now:

{% highlight sh %} 
$ rails new react-sample
{% endhighlight %} 

Add Gemfile dependencies
=====

The first gem we'll want is `react-rails`:

{% highlight ruby %} 
gem 'react-rails', '~> 1.0.0'
{% endhighlight %} 

react-rails provides React in the asset pipeline, JSX transformation, view helpers & React UJS, serverside pre-rendering & a component generator task.

Whilst this setup doesn't require all of `react-rails`' functionality, it is nevertheless useful. 

If you have no need of commonJS style require calls, you can stop here as react-rails provides all you need to get started developing React interfaces for Rails applications.

Otherwise, add `browserify-rails` to your gemfile:

{% highlight ruby %} 
gem 'browserify-rails', '~> 1.0.1'
{% endhighlight %} 
Create package.json
=====

npm provides a handy utility for generating a package.json file, but you can do this by hand, if you like. See my example at the end of this section for reference!

To use the generator: 

{% highlight sh %} 
$ npm init 
{% endhighlight %} 

and follow the prompts. We aren't concerned with an entry point, and you may not necessarily care about the other fields the initializer adds to the package.json file. Check out [this](http://browsenpm.org/package.json) sick interactive guide to the various parts of a package.json file!

Here's my package file:

{% highlight json %} 
{ 
  "name": "react-sample",
  "devDependencies": { 
    "browserify": "^10.2.4",
    "browserify-incremental": "^3.0.1",
    "reactify": "^1.1.1"
  },
  "engines": { 
    "node": ">= 0.10"
  }
}

{% endhighlight %} 

[Browserify](https://www.npmjs.com/package/browserify) gives us `require()` in the browser by parsing modules containing these calls and bundling them for the client-side.

[Browserify Incremental](https://www.npmjs.com/package/browserify-incremental) allows for incremental rebuilds of our modules - meaning only changed files will be parsed.

[Reactify](https://www.npmjs.com/package/reactify) is a Browserify plugin which provides a transformer for JSX. We need to farm this out to Browserify in order for the require() calls to be appropriately handled. 

Go ahead and run:

{% highlight sh %} 
$ npm install
{% endhighlight %} 

to install these modules.
