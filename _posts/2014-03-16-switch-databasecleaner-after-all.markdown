---
layout: post
title:  "DatabaseCleaner tunning"
date:   2014-03-16 15:04:01
---

I use DatabaseCleaner gem to clean up db before each example in my specs. But sometimes I want it to clean db after some context ran. In other words want to have some shared resources between examples.

It is recommended to make DatabaseCleaner.clean in after(:each) block. What I would want to have is to write in such a way:

{% highlight ruby %}

context "heavy spec", clean_after_all: true do
  before(:all) do
    @shared_resource = .. some heavy db records creation logic
  end
  .. specs that use shared resource
end

{% endhighlight %}

And have this cleaned when context is done.

This can be accomplished with such config in spec_helper.rb file:

{% highlight ruby %}

RSpec.configure do |config|
  
  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation) 
  end

  config.before(:each) do
    DatabaseCleaner.strategy = example.metadata[:clean_after_all] ? nil : :transaction
  end

  config.after(:each) do
    DatabaseCleaner.clean
    DatabaseCleaner.start
  end

  config.after(:all, clean_after_all: true) do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean
    DatabaseCleaner.start
  end

end

{% endhighlight %}