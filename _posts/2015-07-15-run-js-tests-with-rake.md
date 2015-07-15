---
title: "Run Javascript/npm Tests with Rake"
layout: post
date: 2015-07-15
categories: code
---

If, like me, you find the idea of needing two commands to run your Ruby & Javascript test suites objectionable, try adding this to your Rakefile:

{% highlight ruby %}
namespace :test do
  desc 'run NPM tests'
  Rake::TestTask.new(:npm_test) do |t|
    puts `npm test`
  end
end

Rake::Task[:test].enhance { Rake::Task["test:npm_test"].invoke }
{% endhighlight %}

This snippet extends the standard `rake test` task to include `npm test`.
