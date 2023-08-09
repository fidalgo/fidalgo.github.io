---
layout: post
title: Service Objects in the Wild
subtitle: A Safari Through Testing with Minitest and the Actor Gem
tags:
  [
    "Ruby",
    "Minitest",
    "Testing",
    "Service Objects",
    "Actor Gem",
    "Mocks",
    "Stubs",
    "Software Development",
    "Code Quality",
    "Test-Driven Development",
    "Rails",
    "Software Architecture",
    "Business Logic",
    "Object-Oriented Programming",
    "Software Engineering",
    "Development Techniques",
    "Coding",
    "Beginner Friendly",
  ]
---

Testing is a crucial part of software development, and understanding how to effectively use tools like Minitest, mocks, and stubs is essential.
This blog post explores these concepts in the context of Ruby, specifically focusing on the Actor gem for creating service objects.

# Introduction

As a mentor, I've frequently come across questions from developers transitioning from RSpec or other languages to Minitest.
The shift can be challenging, especially when it comes to understanding the nuances of testing, mocking, and stubbing in a new environment.
These conversations with my mentees have not only highlighted the common struggles but also the need for a
comprehensive guide that bridges the gap.

In this blog post, I aim to provide that bridge, focusing specifically on testing with Minitest and the Actor gem.
Whether you're an RSpec veteran exploring Minitest or a developer coming from another language, this post is
designed to ease your transition and enhance your testing skills in Ruby.

Minitest, a popular testing framework in the Ruby ecosystem, offers a compelling alternative to RSpec,
you can learn more about it in the [official Minitest documentation](https://docs.seattlerb.org/minitest/).
Known for its simplicity and minimalism, Minitest provides a straightforward testing experience without unnecessary complexity.
Its speed, coupled with out-of-the-box parallelism, allows for quick development cycles, especially under Ruby on Rails.
Minitest's more "Ruby-like" approach, with less reliance on DSLs,
makes the code transparent and easy to understand. Being the choice of both Ruby and Rails, it aligns well with foundational technologies.
If you value a lightweight, fast, and clear testing framework, giving Minitest a go might be the perfect decision.

We'll be using some fun animal-themed examples involving penguins and crocodiles,
where we leverage the [Actor gem](https://github.com/sunny/actor) to write elegant service objects.
So buckle up, and let's dive into the world of testing with Minitest!

# Understanding Mocks and Stubs

In the world of testing, mocks and stubs are two common techniques used to isolate the code under test from its dependencies.
Though the terms are often used interchangeably, they serve different purposes.
Let's explore these concepts, drawing from the definitive resources by Martin Fowler and his article [Mocks Aren't Stubs](https://www.martinfowler.com/articles/mocksArentStubs.html).

## What Are Stubs?

Stubs provide canned answers to calls made during the test. They are used to control the indirect inputs of the system under test. Stubs don't usually respond to anything outside what's programmed in for the test.

Example: Testing a `PenguinFeeder` class with a Stub

```ruby
require "minitest/autorun"

class PenguinFeeder
  def feed(penguin)
    penguin.eat(:fish)
  end
end

class PenguinFeederTest < Minitest::Test
  def test_feed_penguin
    penguin = Minitest::Mock.new
    penguin.expect :eat, true, [:fish]

    feeder = PenguinFeeder.new
    assert feeder.feed(penguin)
  end
end
```

run it with `ruby penguin_feeder.rb`:

```shell
Run options: --seed 19621

# Running:

.

Finished in 0.001013s, 987.6212 runs/s, 987.6212 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

## What Are Mocks?

Mocks, on the other hand, are objects pre-programmed with expectations.
They are used to verify the indirect outputs of the system under test, ensuring that it interacts with its collaborators in expected ways.

Example: Testing a `CrocodileTrainer` class with a Mock

```ruby
require "minitest/autorun"

class CrocodileTrainer
  def train(crocodile)
    crocodile.perform(:roll_over)
  end
end

class CrocodileTrainerTest < Minitest::Test
  def test_train_crocodile
    crocodile = Minitest::Mock.new
    crocodile.expect :perform, true, [:roll_over]

    trainer = CrocodileTrainer.new
    assert trainer.train(crocodile)
    crocodile.verify
  end
end
```

run it with `ruby crocodile_trainer.rb`:

```shell
Run options: --seed 1695

# Running:

.

Finished in 0.000683s, 1464.3561 runs/s, 1464.3561 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

## When to Use Stubs vs Mocks?

Use Stubs When:

- You want to control the behaviour of a dependency.
- You're not concerned with how the dependency is used, only with its output.

Use Mocks When:

- You want to ensure that a dependency is used correctly by the system under test.
- You're concerned with the interaction between the system and its dependencies.

Understanding the differences between mocks and stubs, and knowing when to use each, is essential for writing effective tests.
Stubs help you control the behaviour of dependencies, while mocks allow you to verify interactions.
By leveraging both techniques, you can write more robust and maintainable tests.

# Mocking and Stubbing in Minitest

Mocking and stubbing are techniques used in testing to isolate the code under test from its dependencies. Here's a quick overview:

- Mocking: Replacing a real object with a fake one that expects certain calls and returns predefined responses.
- Stubbing: Replacing a method on a real object with a fake method that returns a predefined response.

## Minitest's Approach to Mocking and Stubbing

Minitest provides built-in support for both mocking and stubbing. Let's explore how we can use these features through some animal-themed examples.

## Mocking in Minitest: A Penguin example

Example: Testing a PenguinFeeder Class

Suppose we have a class `PenguinFeeder` that feeds penguins. We want to test this class without actually feeding real penguins (they might get too full!). Here's how we can do it:

```ruby
require 'minitest/autorun'

class Penguin
  def eat(food)
    food == :fish
  end
end

class PenguinFeeder
  def feed(penguin)
    penguin.eat(:fish)
  end
end

class PenguinFeederTest < Minitest::Test
  def test_feed_penguin
    penguin = Minitest::Mock.new
    penguin.expect :eat, true, [:fish]

    feeder = PenguinFeeder.new
    assert feeder.feed(penguin)
    penguin.verify
  end
end
```

We created a mock penguin object and expect it to receive the `eat` method with the argument `:fish`.

When we run the test with `ruby penguin_feeder_mock.rb`, we get the following output:

```shell
Run options: --seed 42862

# Running:

.

Finished in 0.000783s, 1277.5014 runs/s, 1277.5014 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

## Stubbing in Minitest: A Crocodile example

Stubbing is another powerful technique in testing. Let's see how we can use stubbing in Minitest with a crocodile-themed example.
Example: Testing a `CrocodileTrainer` class

Imagine we have a class CrocodileTrainer that trains crocodiles to perform tricks. We want to test this class without actually training real crocodiles (they might not cooperate!). Here's our class:

```ruby
require "minitest/autorun"

class Crocodile
  def perform(action)
    "🐊🙄 performing #{action}"
  end
end

class CrocodileTrainer
  def train(crocodile)
    crocodile.perform(:roll_over)
  end
end

class CrocodileTrainerTest < Minitest::Test
  def test_train_crocodile
    crocodile = Crocodile.new
    perform_action = :roll_over
    stubbed_response = "🐊😃 performing #{perform_action}"

    # Stubbing the perform method to return a specific response
    crocodile.stub :perform, stubbed_response do
      trainer = CrocodileTrainer.new

      # Asserting that the stubbed method is called with the correct argument
      assert_equal stubbed_response, trainer.train(crocodile)

      # Asserting that the original method is not called
      refute_equal "🐊🙄 performing #{perform_action}", trainer.train(crocodile)
    end
  end
end
```

We can stub the `perform` method on a real crocodile object to return a predefined response.

When we run the test with `ruby crocodile_trainer_stub.rb`, we get the following output:

```shell
Run options: --seed 24139

# Running:

.

Finished in 0.000652s, 1533.6882 runs/s, 3067.3765 assertions/s.

1 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

Great! We've trained our virtual crocodile without any real-life risks.

# The Actor Gem and Service/Business Logic Layer

The Actor gem is one of the options available for creating service objects or command objects in Ruby.
It offers a robust and simple way to organize business logic, providing a clear structure for
handling inputs, outputs, errors and other service layer facilities, the Actor gem can be a valuable
tool for developers looking to maintain clean and testable code.

## Why Use Actor for Service or Command Objects?

Service objects encapsulate a specific business operation, while command objects represent a command to be executed. Using the Actor gem for these purposes offers several benefits:

- Readability: Clear separation of inputs, outputs, and the operation itself.
- Testability: Easier to test individual components.
- Reusability: Encourages modular design, making it easier to reuse code.

## Importance of a Service/Business Logic Layer

Managing business logic as an architectural component achieves greater modularity. By keeping it separated from the usual Rails application concepts, we can:

- Reduce Complexity: Isolate complex business rules from controllers and models.
- Enhance Maintainability: Make changes to business logic without affecting other parts of the application.
- Improve Testability: Write more focused and efficient tests.

Example: `PenguinAdoptionService`

Let's create a service object using the Actor gem that handles the adoption of penguins. Here's our class:

```ruby
require "service_actor"

class Penguin
  attr_accessor :available_for_adoption

  def initialize(available_for_adoption)
    @available_for_adoption = available_for_adoption
  end

  def adopt(adopter)
    available_for_adoption = false
    adopter.receive('🐧😍')
  end

  def available_for_adoption?
    available_for_adoption == true
  end
end

class Adopter
  attr_reader :can_adopt

  def initialize(can_adopt)
    @can_adopt = can_adopt
  end

  def can_adopt?
    @can_adopt == true
  end

  def receive(message)
    puts message
  end
end

class PenguinAdoptionService < Actor
  input :penguin
  input :adopter

  output :adoption_status

  def call
    if adopter.can_adopt? && penguin.available_for_adoption?
      penguin.adopt(adopter)
      self.adoption_status = 'Adoption Successful!'
    else
      self.adoption_status = 'Adoption Failed!'
    end
  end
end


```

We can now use this service object to handle the adoption process in a clean and structured way.

```ruby
penguin = Penguin.new(true)
adopter = Adopter.new(true)
result = PenguinAdoptionService.call(penguin: penguin, adopter: adopter)
puts result.adoption_status
```

and we get the result:

```
🐧😍
Adoption Successful!
```

# Testing with the Actor Gem

The Actor gem provides a powerful way to manage business logic as an architectural component,
keeping it separate from the usual Rails application concepts.
This separation achieves greater modularity and makes testing more straightforward.
In this section, we'll explore how to test actors using both mocks and stubs.

## Testing with Stubs

Stubs can be used to control the behaviour of dependencies within an actor.
Here's an example of how you might test a `PenguinDance` actor that depends on a `MusicPlayer` actor.

Example: `PenguinDance` Actor with `MusicPlayer` Stub

```ruby
require 'minitest/autorun'
require 'service_actor'

class MusicPlayer
  def self.play(song)
    "Playing #{song}"
  end
end

class PenguinDance < Actor
  input :music_player, default: MusicPlayer

  def call
    music_player.play(:happy_feet)
  end
end

class PenguinDanceTest < Minitest::Test
  def test_dance_to_music
    MusicPlayer.stub :play, true do
      result = PenguinDance.call
      assert result.success?
    end
  end
end
```

## Testing with Mocks

Mocks can be used to ensure that an actor interacts with its dependencies in the expected way.
Here's an example of how you might test a CrocodileSwim actor that depends on a WaterPool actor.
In this example we use a Mock class, that mimics a class built with Actor gem.

Example: `CrocodileSwim` Actor with `WaterPool` Mock

```ruby
require 'minitest/autorun'
require 'service_actor'

class WaterPool < Actor
  input :type

  output :capacity

  def call
    puts "Filling with #{type}"
    self.capacity = :full
  end
end

class CrocodileSwim < Actor
  input :water_pool, default: WaterPool

  output :completed_pools

  def call
    outcome = water_pool.call(type: :fresh_water)

    self.completed_pools = outcome.capacity == :full ? 23 : 0
  end
end

class MockService
  def initialize(expected_arguments, return_value = nil)
    @expected_arguments = expected_arguments
    @return_value = return_value
  end

  def call(args)
    raise 'Unexpected arguments' unless arguments_match?(@expected_arguments, args)

    ServiceActor::Result.new(@return_value)
  end

  private

  def arguments_match?(expected, actual)
    expected == actual
  end
end

class CrocodileSwimMockServiceTest < Minitest::Test
  def test_swim_in_pool
    mock_service = MockService.new({ type: :fresh_water }, { capacity: :full })

    outcome = CrocodileSwim.call(water_pool: mock_service)
    assert_equal 23, outcome.completed_pools
  end
end
```

Since Actor gem allows dependency injection this is perfectly possible, so we're swapping the real implementation with a dummy service, that answers
to the same methods.
In Ruby **_If it walks like a duck and it quacks like a duck, then it must be a duck._**
is a mantra. For more information check [Wikipedia page for Duck Typing](https://en.wikipedia.org/wiki/Duck_typing)

But this is not the only method to use Mocks, with minitest, as minitest has its own mocking facilities.
Actually we can swap our test implementation with:

```ruby
class CrocodileSwimMinitestMock < Minitest::Test
  def test_swim_in_pool
    mock_service = Minitest::Mock.new
    mock_service.expect :call, ServiceActor::Result.to_result({ capacity: :full }), type: :fresh_water
    # due internal actor checks, if absent an error is raised:
    # NoMethodError: unmocked method :nil?, expected one of [:call]
    mock_service.expect :nil?, false

    outcome = CrocodileSwim.call(water_pool: mock_service)
    assert_equal 23, outcome.completed_pools
    # Verify that all methods were called as defined by expected. Will raise an exception otherwise
    mock_service.verify
  end
end
```

This method has the advantage of ensuring all the method calls outlined in by the `expect` method are indeed called, adding an extra layer of checks,
compared with the `MockService` defined earlier. But it's Ruby and it's using dependency injection, so at the end of the day, it's a matter of
choosing the most appropriate implementation.

## Choosing the Right Approach

Both mocks and stubs have their place in testing, and choosing the right approach depends on the specific needs of your test.

Use Stubs When:

- You want to isolate the system under test from its dependencies.
- You need to control the behaviour of dependencies without verifying interactions.

Use Mocks When:

- You want to ensure that the system under test interacts with its dependencies in a specific way.
- You need to verify both the behaviour and the interactions of dependencies.

# Conclusion

Testing is a vital part of software development, and understanding how to use mocks and stubs effectively can lead to
more robust and maintainable tests. By leveraging the Actor gem and the principles outlined in this post,
developers can write clear and concise tests that ensure their code behaves as expected.

Whether you're new to testing or an experienced developer, we hope this post has provided valuable
insights into the world of mocks, stubs, and testing with the Actor gem. Happy coding!
