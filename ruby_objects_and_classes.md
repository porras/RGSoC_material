# Objects and Classes in Ruby

Ruby is an Object Oriented language. Not a Class Oriented language. Objects are central. Classes are accessory. Let's see what I mean.

## Everything is an object

In Ruby every value *is* an object. It also *has* a class, but that's a detail, most of the time.

## All you do is send messages

You'll probably see it named as *method call*, but everything you can do in Ruby, is objects sending messages to other objects, and replying to them.

    user.login(password)

## So, what's a class

First, what's an object, and what makes an object the object it is, and not another object? Hint: not its class. Well, since the only thing you can make with objects is sending them messages and use their replies, it makes sense that the thing that makes an object *that* object is the messages it understands, and how does it reply to them.

    def total_price_with_discount(normal_price)
      "NOW ONLY #{normal_price.to_i * 0.9}"
    end

A class is a very convenient way to create similar objects. Something like an *object factory*.

Since you're likely to need objects that respond to the same set of messages in the same way, it's a good idea to have something that allows you to create them easily, and that's a class.

To create a class, we use the `class` keyword:

    class MyClass
    end

Now we have a class called `MyClass`, which is an object itself and hence understand messages, specifically one called `new`. When we send a class that message, it replies with a new *instance*:

    my_object = MyClass.new

# Methods

Now, that object is not very interesting, as our *factory* (our class) didn't add anything to it. Let's see what we can add.

    class MyClass
      def hello(name)
        "Hello, #{name}!"
      end
    end

Well, this is useful, and it actually covers 80% of my day to day work with Ruby. We *taught* objects created with this class to respond to the message `hello`:

    my_object = MyClass.new
    my_object.hello('Duana')

# Initialization and instance variables (internal state)

Cool, what else can we do? In this next example, we'll instruct our class to do something each time it creates a new object, and we'll also see how objects can have an internal state, invisible from the outside, but useful for its tasks:

    class Greeter
      def initialize(name)
        @name = name
        @greetings = 0
      end

      def hello(name)
        @greetings += 1
        "Hello #{name}! My name is #{@name}"
      end

      def greetings
        "I have greeted #{@greetings} times"
      end
    end

    greeter = Greeter.new('Sergio')
    greeter.hello('Duana') # "Hello Duana! My name is Sergio"
    greeter.hello('Caro')  # "Hello Caro! My name is Sergio"
    greeter.greetings      # "I have greeted 2 times"

# Class methods

Remember when we said classes are objects themselves, and how they responded to the `new` message? Sometimes it's useful to make them respond to more messages, and we can doing it using `self`. `self` is a special Ruby keyword that references the current object. If we call it in the context of a class definition, that current object is the class, and this way we can add methods to it (teach it new messages it can reply to):

    class Greeter
      def self.default
        Greeter.new('Sergio')
      end

      ...
    end

    Greeter.default.hello('Duana') # "Hello Duana! My name is Sergio"

# Inheritance and method lookup

We saw before how classes are nothing special, and just a convenience way of creating new objects similar to each other, and in this way sharing our code and making it shorter and simpler. But what about creating similar *classes*. It turns out, there's also a way, and it's called *inheritance*. But before, let's explain something important.

We saw how everything we do in Ruby is sending messages to objects, and those objects responding to them. Let's see how Ruby decides whether an object can respond to a message, and how to do it. This is called *method lookup*, and it works like this:

  * Does the object itself respond?
  * Does its class instruct it to respond?
  * Does the ancestors of its class instruct it to respond?

You see the word ancestor there, and there's where inheritance comes in. There are two forms of inheritance in Ruby. Let's see the simplest one:

    class Greeter
      def greeting
        "Hello"
      end

      def greet(name)
        "#{self.greeting}, #{name}!"
      end
    end

    class GermanGreeter < Greeter
      def greeting
        "Hallo"
      end
    end

    greeter = GermanGreeter.new
    greeter.greet('Duana') # "Hallo, Duana!"

When we call `hello` on `greeter`, the lookup chain we saw starts:

  * Does it respond itself? NO
  * Does it's class (`GermanGreeter`) instruct it? NO
  * Does its class first ancestor (`Greeter`) instruct it? YES

Then that code it's run. While so, it calls `greeting` on itself, so the lookup rules are used again

  * Does it respond itself? NO
  * Does it's class (`GermanGreeter`) instruct it? YES

And that's it.

# Modules

The other form of inheritance is similar, but different, and sometimes one makes more sense than the other. A module is a subset of a class. It's similar in that it includes methods to instruct objects to respond to messages, but it's different in that it doesn't understand the `new` message so it can't be used to create objects directly. Let's see how it's used:

    class Greeter
      def greet(name)
        "#{self.greeting}, #{name}#{exclamation}"
      end
    end

    module Spanish
      def greeting
        "Hola"
      end
    end

    module Enthusiast
      def exclamation
        "!!!"
      end
    end

    class Sergio < Greeter
      include Spanish
      include Enthusiast
    end

    Sergio.new.greet('Duana') # "Hola, Duana!!!"

Bonus track:

    Sergio.ancestors
    => [Sergio, Enthusiast, Spanish, Greeter, Object, Kernel, BasicObject]

## Following up

On top of any introductory Ruby book which will explain this features in details, there are a couple of very good books specific about this topic:

  * Metaprogramming Ruby, Paolo Perrotta
  * Practical Object-Oriented Design in Ruby, Sandi Metz
