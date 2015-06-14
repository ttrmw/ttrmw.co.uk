---
title: "trivial_tokens gem"
layout: post
date: 2015-06-14
categories: code
---

I have written my first Rails gem, called trivial_tokens!

This gem handles some of the boilerplate of a typical jQuery tokenInput implementation in rails. It adds a getter and setter for the tokenised form of a model's assocations, a query callback to the association's controller, and provides a generator for the typical minimum javascript required.

The gem makes it really easy to implement tokenInput in rails, it adds:


A class method 'tokenize' on ActiveRecord models, which takes the name of a relation on the model as argument. This method adds getter/setter methods for the 'tokenized' form of the relation - ie, a comma delimited string of IDs.

A controller around_action :index callback to provide handling for the 'q' parameter that tokenInput expects.

A generator to provide the default javascript to create a tokenInput field for the tokenized attribute on forms.

Jquery tokeninput assets (CSS & JS)

Using the gem
=====

Trivial Tokens assumes you wish to provide a token input for a rails association of either has_many, or has_and_belongs_to_many type, so ensure that this is set up.

{% highlight ruby %}
class Article < ActiveRecord::Base
  has_and_belongs_to_many :tags
  #...
end
{% endhighlight %}

To add the model methods trivial_tokens requires: add a call to tokenize, passing the association name:

{% highlight ruby %}
class Article < ActiveRecord::Base
  has_and_belongs_to_many :tags
  tokenize :tags
  #...
end
{% endhighlight %}

In the model's controller, permit the tokenised_tags parameter:

{% highlight ruby %}
class Article < ActiveRecord::Base
  #...
  def article_params
    params.require(:article).permit(:tokenized_tags, :other_params)
  end
end
{% endhighlight %}

In the association's controller, add a call to tokenize:


{% highlight ruby %}
class TagsController < ApplicationController
  tokenize
  #...
end
{% endhighlight %}

Next, in application.js, require tokenInput:

{% highlight js %}
//= require jquery.tokeninput
{% endhighlight %}

And in application.css:

{% highlight css %}
*= require token-input
{% endhighlight %}
or
{% highlight css %}
*= require token-input-facebook
{% endhighlight %}
or
{% highlight css %} 
*= require token-input-mac
{% endhighlight %}
depending on your needs.


To generate the minimum javascript required for the jquery tokenInput functionality, run the token_input generator. This generator takes the model and its association as arguments:

{% highlight sh %}
$ rails generate trivial_tokens:token_input article tag
{% endhighlight %}

This will generate a javascript file in `app/assets/javascripts` which will call tokenInput on fields with id:

{% highlight css %}
#article_tokenized_tags
{% endhighlight %}

So go ahead and add a tokenized_tags field to your form view:

{% highlight html %}
<!-- ... -->
<div class="field"> 
  <%= f.label :tokenized_tags %> 
  <%= f.text_field :tokenized_tags %>
</div>
<!-- ... --> 
{% endhighlight %}

that's it! 
