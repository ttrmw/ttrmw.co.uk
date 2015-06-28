---
title: "Rails, React & npm modules with Browserify"
layout: post
date: 2015-06-28
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
    "reactify": "^1.1.1",
    "react": "^0.13.3"
  }
  "engines": { 
    "node": ">= 0.10"
  }
}

{% endhighlight %} 

[Browserify](https://www.npmjs.com/package/browserify) gives us `require()` in the browser by parsing modules containing these calls and bundling them for the client-side.

[Browserify Incremental](https://www.npmjs.com/package/browserify-incremental) allows for incremental rebuilds of our modules - meaning only changed files will be parsed.

[Reactify](https://www.npmjs.com/package/reactify) is a Browserify plugin which provides a transformer for JSX. We need to farm this out to Browserify in order for the require() calls to be appropriately handled. 

[React](https://www.npm.js.com/package/react) we know about, of course. We add it as a dependency here, as we will need to be able to `require()` it for third party modules to work with it. Though `react-rails` also provides the library, we will use this version.

In this case, we specify the modules as `devDependencies` - Browserify will prepare static files for us - but you would want to specify them as `dependencies` if you need the modules on the server.

If you're using git, add the `node_modules` directory to your .gitignore: 

{% highlight sh %} 
$ echo 'node_modules/' >> .gitignore
{% endhighlight %} 

And go ahead and run:

{% highlight sh %} 
$ npm install
{% endhighlight %} 

to install these modules.

Config items
=======

Add this to `config/application.rb` to instruct Browserify to transform .jsx files:

{% highlight ruby %} 
config.browserify_rails.commandline_options = "--transform reactify --extension=\".jsx\""{% endhighlight %}

Add these to `config/environments/development.rb` and `config/environments/production.rb`:

{% highlight ruby %}
#development.rb:
config.react.variant = :development

#production.rb:
config.react.variant = :production
{% endhighlight %} 

The development react variant will give us more debugging information, which we wont want in the production environment. 

Javascript 
=====
It is my preference to store React components under `app/assets/javascripts/components`, and index these in a `components.js` file in the `javascripts` asset directory:

{% highlight sh %}
$ cd app/assets/javascripts
$ touch components.js && mkdir components
{% endhighlight %} 

Next, we must make React available to the clientside javascript. 

In a typical `react-rails` application we would use a sprockets require directive. We cannot do that here, as we will need any React plugins that depend on `require()`ing React to be able to do so. Instead, we will include `react_ujs` in the normal manner, and `require()` React and its add ons.

{% highlight javascript %} 
//app/assets/javascripts/components.js

//= require_self
//= require react_ujs

React    = require('react');

//add other modules as needed, eg:
//ReactDnD = require('react-dnd');
// ... 
{% endhighlight %}

and use an ordinary sprocket require to include this file globally:

{% highlight javascript %} 
//app/assets/javascripts/application.js

//..
//= require components
//

{% endhighlight %} 



Components 
=====

The process for creating React components is more or less the same as with `react-rails`, but there are a couple of gotchas.

Firstly, since we are using `reactify` to handle jsx transformation, the component files will not need to be named `.js.jsx`, as sprockets will not be performing the transformation. Simply `.jsx` will be fine.

Additionally, we will need to export from the file so our components can be required, eg:

{% highlight js %} 
module.exports = Widgets;
{% endhighlight %}

and for each component require the `.jsx` file: 

{% highlight javascript %} 
// for Widgets.jsx, eg

Widgets = require ('./components/Widgets.jsx');
{% endhighlight %}

So a component might look something like:

{% highlight javascript %} 
var Widgets = React.createClass({ 
  render: function() { 
    return (
      <div className="widgets">
        <FooWidgets/>
      </div>
    );
  }
});

module.exports = Widgets;
{% endhighlight %}

and be required with: 

{% highlight javascript %} 
// app/assets/javascripts/components.js
Widgets = require('./components/Widgets.jsx');

{% endhighlight %}



Rendering React components
=====

Finally, we can call the `react_rails` view helper to render our components: 

{% highlight ruby %} 
<%= react_component 'Widgets' %>
{% endhighlight %} 

We can also use this view helper to pass props into the component, as per the [react-rails documentation](https://github.com/reactjs/react-rails#rendering--mounting).
