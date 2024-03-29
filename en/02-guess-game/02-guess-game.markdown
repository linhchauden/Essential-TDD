# Guess Game #

## Objectives ##

- How to test random behavior ?
- Illustrate inverting dependencies.
- How to make your code depend on abstractions instead of concrete implementation ?
- Illustrate Single Responsibility Principle. No And, Or, or But.
- Illustrate programming to an interface not to an implementation.
- When to use partial stub on a real object ? Illustrated by spec 7 and 8.
- Random test failures due to partial stub. Fixed by isolating the random number generation.
- Small methods focused on doing one thing.
- How to defer decisions by using Mocks ?
- Using mock that complies with Gerard Meszaros standard.
- How to use as_null_object ?
- How to write contract specs to keep mocks in sync with code ?

## Guessing Game Description ##

Write a program that generates a random number between 0 and 100 (inclusive). The user must guess this number. Each correct guess (if it was a number) will receive the response "Guess Higher!" or "Guess Lower!". Once the user has successfully guessed the number, you will print various statistics about their performance as detailed below:

* The prompt should display : "Welcome to the Guessing Game"
* When the program is run it should generate a random number between 0 and 100 inclusive
* You will display a command line prompt for the user to enter the number representing their guess. Quitting is not an option. The user can only end the game by guessing the target number. Be sure that your prompt explains to them what they are to do.
* Once you have received a value from the user, you should perform validation. If the user has given you an invalid value (anything other than a number between 1 and 100), display an appropriate error message. If the user has given you a valid value, display a message either telling them that there were correct or should guess higher or lower as described above. This process should continue until they guess the correct number.
* Once the user has guessed the target number correctly, you should display a "report" to them on their performance. This report should provide the following information:
	- The target number
	- The number of guesses it took the user to guess the target number
	- A list of all the valid values guessed by the user in the order in which they were guessed.
	- A calculated value called "Cumulative error". Cumulative error is defined as the sum of the absolute value of the difference between the target number and the values guessed. For example : if the target number was 30 and the user guessed 50, 25, 35, and 30, the cumulative error would be calculated as follows:
	
	|50-30| + |25-30| + |35-30| + |30-30| = 35
	
	Hint: See http://www.w3schools.com/jsref/jsref_abs.asp for assistance
	- A calculated value called "Average Error" which is calculated as follows: cumulative error / number of valid guesses. Using the above number set, the average error is 8.75.
	- A text feedback response based on the following rules:
	- If average error is 10.0 or lower, the message "Incredible guessing!"
	- If average error is higher than above but under 20.0, "Good job!"
	- If average error is higher than 20 but under 30.0, "Fair!"
	- Anything other score: "You are horrible at this game!"
	
## Version 1 ##

The random generator spec will never pass.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random

    result.should == 50
  end
end
```

guess_game.rb

```ruby
class GuessGame
  def random
    Random.new.rand(1..100)
  end
end
```

##	Version 2 ##

The above spec deals with the problem of randomness. You cannot use stub to deal with this spec because you will stub yourself out. The spec checks only the range of the generated random number is within the expected range.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random

    expected_range = 1..100
    expected_range.should cover(result)
  end
end
```

guess_game.rb

```ruby
class GuessGame
  def random
    Random.new.rand(1..100)
  end
end
```

Note: Using expected.include?(result) is also ok (does not use rspec matcher).

## Version 3 ##

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100 
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = mock('Console')
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
end
```

This spec shows how you can defer decisions about how you will interact with the user, it could be standard out, GUI, client server app etc. Fake object is injected into the game object. 
	 
 The interface output is discovered during the mocking and it hides the details about the type of interface that must be implemented to communicate with an user. Game delegates any user interfacing code to a concrete console object therefore it obeys single responsibility principle. Console objects also obey the single responsibility principle.
	
We could have implemented this similar to the code breaker game in RSpec book by calling the puts method on output variable, by doing so we tie our game object to the implementation. This results in tightly coupled objects which is not desirable. We want loosely coupled objects with high cohesion.

guess_game.rb

```ruby
class GuessGame
  def initialize(console=STDOUT)
    @console = console
  end
  
  def random
    Random.new.rand(1..100)
  end
  
  def start
    @console.output("Welcome to the Guessing Game")
  end
end
```

## Version 4 ##

Using mock that complies with Gerard Meszaros standard. Use double and if expectation is set, then it is a mock, otherwise it can be used as a stub.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console')
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
end
```

guess_game.rb

```ruby
class GuessGame
  def initialize(console=STDOUT)
    @console = console
  end
  
  def random
    Random.new.rand(1..100)
  end
  
  def start
    @console.output("Welcome to the Guessing Game")
  end
end
```

## Version 5 ##

Spec exposes the bug : constructor default value is not correct.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console')
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end
end
```

This spec exposes the bug due to wrong default value in the constructor. 

guess_game.rb

```ruby
class GuessGame
  def initialize(console=STDOUT)
    @console = console
  end
  
  def random
    Random.new.rand(1..100)
  end
  
  def start
    @console.output("Welcome to the Guessing Game")
  end
end
```

## Version 6 ##

Fixed the bug due to wrong default value in the constructor. Concrete classes depend on an abstract interface called output and not specific things like puts or gui related method.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console')
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end
end
```

The fix shows how to invert dependencies on concreted classes to abstract interface. In this case the abstract interface is 'output' and not specific method like 'puts' or GUI related method that ties the game logic to a concrete implementation.

standard_output.rb

```ruby
class StandardOutput
  def output(message)
    puts message
  end
end
```

guess_game.rb

```ruby
require_relative 'standard_output'

class GuessGame
  def initialize(console=StandardOutput.new)
    @console = console
  end
  
  def random
    Random.new.rand(1..100)
  end
  
  def start
    @console.output("Welcome to the Guessing Game")
  end
end
```

## Version 7 ##

Added spec #4. Illustrates the use of as_null_object.

```ruby
 In irb type: 
> require 'rspec/mocks/standalone'

s = stub.as_null_object
```

s acts as a dev/null equivalent for tests. It ignores any messages that it receives. Useful for incidental interactions that is not relevant to what is being tested.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end
  
  it "should prompt the user to enter the number representing their guess." do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:prompt).with('Enter a number between 1 and 100')
    game = GuessGame.new(fake_console)
    game.start    
  end
end
```

 Spec 4 breaks existing spec 2. It is fixed by using as_null_object which ignores any messages not set as expectation.

guess_game.rb

```ruby
require_relative 'standard_output'

class GuessGame
  def initialize(console=StandardOutput.new)
    @console = console
  end
  
  def random
    Random.new.rand(1..100)
  end
  
  def start
    @console.output("Welcome to the Guessing Game")
    @console.prompt("Enter a number between 1 and 100")
  end
end
```

standard_output.rb

```ruby
class StandardOutput
  def output(message)
    puts message
  end
  
  def prompt(message)
    output(message)
    puts ">"
  end
end
```

## Version 8 ##

Added validation. random method deleted because it is required once per game.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end
  
  it "should prompt the user to enter the number representing their guess." do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:prompt).with('Enter a number between 1 and 100')
    game = GuessGame.new(fake_console)
    game.start    
  end
  
  it "should perform validation of the guess entered by the user : lower than 1" do
    fake_console = double('Console')
    game = GuessGame.new(fake_console)
    game.guess = 0
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should perform validation of the guess entered by the user : higher than 100" do
    fake_console = double('Console')
    game = GuessGame.new(fake_console)
    game.guess = 101
    
    game.error.should == 'The number must be between 1 and 100'            
  end
end
```

guess_game.rb

```ruby
require_relative 'standard_output'

class GuessGame
  attr_accessor :guess
  attr_accessor :error
  attr_reader :random
  
  def initialize(console=StandardOutput.new)
    @console = console
    @random = Random.new.rand(1..100)
  end
    
  def start
    @console.output("Welcome to the Guessing Game")
    @console.prompt("Enter a number between 1 and 100 to guess the number")
  end
  
  def guess=(n)
    if (n < 1) or (n > 100)
      @error = 'The number must be between 1 and 100'
    end
  end
end
```

standard_output.rb

```ruby
class StandardOutput
  def output(message)
    puts message
  end
  
  def prompt(message)
    output(message)
    puts ">"
  end
end
```

## Version 9 ##

Refactored specs.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end
  
  it "should prompt the user to enter the number representing their guess." do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:prompt).with('Enter a number between 1 and 100')
    game = GuessGame.new(fake_console)
    game.start    
  end
  
  it "should perform validation of the guess entered by the user : lower than 1" do
    game = GuessGame.new
    game.guess = 0
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should perform validation of the guess entered by the user : higher than 100" do
    game = GuessGame.new
    game.guess = 101
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should give clue when the input is valid" do
    
  end
end
```

Spec 5 and 6 simplified by removing unnecessary double.

guess_game.rb

```ruby
require_relative 'standard_output'

class GuessGame
  attr_accessor :guess
  attr_accessor :error
  attr_reader :random
  
  def initialize(console=StandardOutput.new)
    @console = console
    @random = Random.new.rand(1..100)
  end
    
  def start
    @console.output("Welcome to the Guessing Game")
    @console.prompt("Enter a number between 1 and 100")
  end
  
  def guess=(n)
    if (n < 1) or (n > 100)
      @error = 'The number must be between 1 and 100'
    end
  end
end
```

## Version 10 ##

Fixed random test failures by isolating random number generation to its own class (partial stub removed). Methods are smaller and focused.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end
  
  it "should prompt the user to enter the number representing their guess." do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:prompt).with('Enter a number between 1 and 100')
    game = GuessGame.new(fake_console)
    game.start    
  end
  
  it "should perform validation of the guess entered by the user : lower than 1" do
    game = GuessGame.new
    game.guess = 0
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should perform validation of the guess entered by the user : higher than 100" do
    game = GuessGame.new
    game.guess = 101
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should give clue when the input is valid and is less than the computer pick" do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is lower')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.stub(:random).and_return { 25 }
    game.guess = 10    
  end

  it "should give clue when the input is valid and is greater than the computer pick" do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is higher')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.guess = 35    
  end
end
```

Spec 7 and 8 illustrates use of mocks and partial stubs. Minimize partial stubs and use them only when it is absolutely required.

guess_game.rb

```ruby
require_relative 'standard_output'
require_relative 'randomizer'

class GuessGame
  attr_reader :guess
  attr_accessor :error
  attr_reader :random
  
  def initialize(console=StandardOutput.new, randomizer=Randomizer.new)
    @console = console
    @random = randomizer.get
  end
    
  def start
    @console.output("Welcome to the Guessing Game")
    @console.prompt("Enter a number between 1 and 100")
  end
  
  def guess=(n)
    @guess = n
    give_clue if valid
  end
  
  private
  
  def valid
    if (@guess < 1) or (@guess > 100)
      @error = 'The number must be between 1 and 100'
      false
    else
      true
    end
  end
  
  def give_clue
    if @guess < @random
      @console.output('Your guess is lower')
    elsif @guess > @random
      @console.output('Your guess is higher')
    else
      # @console.output('Your guess is correct')
    end
  end
end
```

randomizer.rb

```ruby
class Randomizer
  def get
    Random.new.rand(1..100)
  end
end
```

guess_game.rb

```ruby
require_relative 'standard_output'
require_relative 'randomizer'

class GuessGame
  attr_reader :guess
  attr_accessor :error
  attr_reader :random
  
  def initialize(console=StandardOutput.new, randomizer=Randomizer.new)
    @console = console
    @random = randomizer.get
  end
    
  def start
    @console.output("Welcome to the Guessing Game")
    @console.prompt("Enter a number between 1 and 100")
  end
  
  def guess=(n)
    @guess = n
    give_clue if valid
  end
  
  private
  
  def valid
    if (@guess < 1) or (@guess > 100)
      @error = 'The number must be between 1 and 100'
      false
    else
      true
    end
  end
  
  def give_clue
    if @guess < @random
      @console.output('Your guess is lower')
    elsif @guess > @random
      @console.output('Your guess is higher')
    else
      # @console.output('Your guess is correct')
    end
  end
end
```

## Version 11 ##

Added the spec for correct guess. Renamed private method to reflect its abstraction.

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end

  it "should prompt the user to enter the number representing their guess." do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:prompt).with('Enter a number between 1 and 100')
    game = GuessGame.new(fake_console)
    game.start    
  end
  
  it "should perform validation of the guess entered by the user : lower than 1" do
    game = GuessGame.new
    game.guess = 0
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should perform validation of the guess entered by the user : higher than 100" do
    game = GuessGame.new
    game.guess = 101
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should give clue when the input is valid and is less than the computer pick" do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is lower')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.stub(:random).and_return { 25 }
    game.guess = 10    
  end

  it "should give clue when the input is valid and is greater than the computer pick" do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is higher')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.guess = 35    
  end
  
  it "should recognize the correct answer when the guess is correct." do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is correct')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.guess = 25    
  end
end
```

guess_game.rb

```ruby
require_relative 'standard_output'
require_relative 'randomizer'

class GuessGame
  attr_reader :guess
  attr_accessor :error
  attr_reader :random
  
  def initialize(console=StandardOutput.new, randomizer=Randomizer.new)
    @console = console
    @random = randomizer.get
  end
    
  def start
    @console.output("Welcome to the Guessing Game")
    @console.prompt("Enter a number between 1 and 100")
  end
  
  def guess=(n)
    @guess = n
    give_feedback if valid
  end
  
  private
  
  def valid
    if (@guess < 1) or (@guess > 100)
      @error = 'The number must be between 1 and 100'
      false
    else
      true
    end
  end
  
  def give_feedback
    if @guess < @random
      @console.output('Your guess is lower')
    elsif @guess > @random
      @console.output('Your guess is higher')
    else
      @console.output('Your guess is correct')
    end
  end
end
```

## Version 12 ##

Added contract specs to illustrate how to keep mocks in sync with code.

console_interface_spec.rb

```ruby
shared_examples "Console Interface" do
  describe "Console Interface" do
    it "should implement the console interface: output(arg)" do
      @object.should respond_to(:output).with(1).argument
    end
    
    it "should implement the console interface: prompt(arg)" do
      @object.should respond_to(:prompt).with(1).argument
    end
  end
end
```

Console Interface spec illustrates how to write contract specs. This avoids the problem of specs passing / failing due to mocks going out of synch with the code. When to use them? If you are using lot of mocks you many not be able to write contract tests for all of them. In this case, think about writing contract tests for the most dependent and important module of your application.

standard_output.rb

```ruby
class StandardOutput
  def output(message)
    puts message
  end
  def prompt(message)
    output(message)
    puts ">"
  end
end
```

standard_output_spec.rb

```ruby
require_relative 'console_interface_spec'
require_relative 'standard_output'

describe StandardOutput do
  before(:each) do
    @object = StandardOutput.new
  end
  
  it_behaves_like "Console Interface"
end
```

guess_game_spec.rb

```ruby
require_relative 'guess_game'

describe GuessGame do
  it "should generate random number between 1 and 100 inclusive" do
    game = GuessGame.new
    result = game.random
    
    expected = 1..100
    expected.should cover(result)
  end
  
  it "should display greeting when the game begins" do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with(greeting)
    game = GuessGame.new(fake_console)
    game.start
  end
  
  it "should display greeting to the standard output when the game begins" do
    game = GuessGame.new
    game.start
  end
  
  it "should prompt the user to enter the number representing their guess." do
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:prompt).with('Enter a number between 1 and 100')
    game = GuessGame.new(fake_console)
    game.start    
  end
  
  it "should perform validation of the guess entered by the user : lower than 1" do
    game = GuessGame.new
    game.guess = 0
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should perform validation of the guess entered by the user : higher than 100" do
    game = GuessGame.new
    game.guess = 101
    
    game.error.should == 'The number must be between 1 and 100'            
  end
  
  it "should give clue when the input is valid and is less than the computer pick" do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is lower')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.stub(:random).and_return { 25 }
    game.guess = 10    
  end

  it "should give clue when the input is valid and is greater than the computer pick" do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is higher')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.guess = 35    
  end

  it "should recognize the correct answer when the guess is correct." do
    fake_randomizer = stub(:get => 25)
    fake_console = double('Console').as_null_object
    fake_console.should_receive(:output).with('Your guess is correct')
    game = GuessGame.new(fake_console, fake_randomizer)
    game.guess = 25    
  end
end
```

guess_game.rb

```ruby
require_relative 'standard_output'
require_relative 'randomizer'

class GuessGame
  attr_reader :guess
  attr_accessor :error
  attr_reader :random
  
  def initialize(console=StandardOutput.new, randomizer=Randomizer.new)
    @console = console
    @random = randomizer.get
  end
    
  def start
    @console.output("Welcome to the Guessing Game")
    @console.prompt("Enter a number between 1 and 100")
  end
  
  def guess=(n)
    @guess = n
    give_feedback if valid
  end
  
  private
  
  def valid
    if (@guess < 1) or (@guess > 100)
      @error = 'The number must be between 1 and 100'
      false
    else
      true
    end
  end
  
  def give_feedback
    if @guess < @random
      @console.output('Your guess is lower')
    elsif @guess > @random
      @console.output('Your guess is higher')
    else
      @console.output('Your guess is correct')
    end
  end
end
```
