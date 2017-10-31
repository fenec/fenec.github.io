---
layout: post
title: Having fun with instance_eval and instance_exec in Ruby
comments: true
---
If you ever used Ruby On Rails or one of the popular gems like FactoryGirl (now FactoryBot), Rspec or ActiveAdmin, chances are you have seen these type of constructions:

```ruby
FactoryBot.define do
  factory :user
end
```

As you probably know, It's called _DSL_ - Domain-specific language.

If we break down what happens here, we could see that class method `define` accepts a block with a `factory` method. So, how this `factory` method works?

Looking at the source code, we could find that it all boils down to usage of  `instance_eval`:

```ruby
def define(&block)
  DSL.run(block)
end

class DSL
  ...

  def self.run(block)
    new.instance_eval(&block)
  end
end
```

<https://github.com/thoughtbot/factory_bot/blob/c716ce01b448ce4e0bf855c5a2c63ecb9206322e/lib/factory_bot/syntax/default.rb#L49>

What `instance_eval` does is:

> Evaluates a string containing Ruby source code, or the given block, within the context of the receiver (obj).

In other words, it means that the context (scope) in which your block is evaluated is changed to the receiver of the block. In our case - to `DSL` class.

You could check out the example usage here: <https://ruby-doc.org/core-2.2.0/BasicObject.html#method-i-instance_eval>

If we were to write a code without usage of `instance_eval`, we would have to explicitly define an object for which we call our method:

```ruby
FactoryBot.define do |bot|
  bot.factory :user
end
```
Not so neat anymore, right?

There's also `instance_exec` method, which works the same way but allowing us to pass arguments as block params:

<https://ruby-doc.org/core-2.2.0/BasicObject.html#method-i-instance_exec>

### We can do that too!

Let's write our own DSL using these methods.

Imagine we had a rocket and we wanted our system to print the rocket name along with its crew members. It could look like this:

```ruby
Rocket.crew "Andromeda" do
  member "Jack"
  member "Jill"
end
```

And the implementation will be:

```ruby
class Rocket
  def self.crew(ship_name, &block)
    puts "#{ship_name} members are:"
    new.instance_exec(&block)
  end

  def member(name)
    puts name
  end
end
```

If we run this code we get:
```
Andromeda members are:
Jack
Jill
```

Yey!

### Conclusion

Now you know how this stuff works, you could probably start writing your own DSL (when it's appropriate and useful). This was only a brief look at the power of Ruby metaprogramming but hopefully it could inspire you to explore the topic more deeply. Luckily, there are lot of articles on this topic and you can always dig in the source code of one the popular gems.
