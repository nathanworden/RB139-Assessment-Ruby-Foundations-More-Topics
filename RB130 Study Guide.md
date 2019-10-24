[toc]

###Blocks

####Closures and Scope

A **closure** is a general programming concept that allows programmers to save a "chunk of code" and execute it at a later time. It's called a "closure" because it's said to bind its surrounding artifacts (ie, variables, methods, objects, etc) and build an "enclosure" around everything so that they can be referenced when the closure is later executed. It's sometimes useful to think of a closure as a method that you can pass around and execute, but it's not defined with an explicit name. 

In Ruby, a closure is implemented through a `Proc` object, a lambda, or a block. That is, we can pass around these items as a "chunk of code" and execute them later.

This "chunk of code" retains references to its surrounding artifacts -- its **binding**.

There are three main ways to work with closures in Ruby:

1. Instantiating an object from the `Proc` class

2. Using **lambdas**

3. Using **blocks**

   

####How blocks work, and when we want to use them

A block is an *argument* to the method call. In other words, our familiar method, `[1, 2, 3].each { |num| puts num }`, is actually *passing in* the block of code to the `Array#each` method.



###### Why is it that sometimes the code in the block affects the return value, and sometimes not?

```ruby 
# Example 1: passing in a block to the `Integer#times` method.

3.times do |num|
  puts num
end
=> 3

# Example 2: passing in a block to the `Array#map` method.

[1, 2, 3].map do |num|
  num + 1
end
=> [2, 3, 4]

# Example 3: passing in a block to the `Hash#select` method.

{ a: 1, b: 2, c: 3, d: 4, e: 5 }.select do |_, value|
  value > 2
end
=> { c: 3, d: 4, e: 5 }
```

The answer lies in how each of those methods are implemented. The entire block is *passed in* to the method like any other argument, and it's up to the method implementation to decide *what* to do with the block, or chunk of code, that you passed in. The method could take that block and execute it, or just as likely, it could completely ignore it.

If your method implementation contains a `yield`, a developer using your method can come in after this method is fully implemented and inject additional code in the middle of this method (without modifying the method implementation), by passing in a block of code. This is indeed one of the major use cases of using blocks, which we'll talk more about later.

###### Passing execution to the block

```ruby
# method implementation
def say(words)
  yield if block_given?
  puts "> " + words
end

# method invocation
say("hi there") do
  system 'clear'
end               # clears screen first, then outputs "> hi there"
```

 The `def say...` code is the method implementation, and the `say...` code is the method invocation. 

###### When to use blocks in your own methods

1. Defer some implementation decisions to method invocation time. It allows method callers to refine a method at invocation time for a specific use case. It allows method implementors to build generic methods that can be used in a variety of ways.
2. Methods that need to perform some "before" and "after" actions - sandwich code. For example, closing a `File` automatically.



####Blocks and Variable Scope

Recall that previously we asked you to memorize local variable scope in terms of inner and outer scope, as determined by where the local variable was initialized. A block creates a new scope for local variables, and only outer local variables are accessible to inner blocks.

```ruby
level_1 = "outer-most variable"

[1, 2, 3].each do |n|                 # block creates a new scope
  level_2 = "inner variable"

  ['a', 'b', 'c'].each do |n2| 				# nested block creates a nested scope
    level_3 = "inner-most variable"

    # all three level_X variables are accessible here
  end

  # level_1 is accessible here
  # level_2 is accessible here
  # level_3 is not accessible here

end

# level_1 is accessible here
# level_2 is not accessible here
# level_3 is not accessible here
```

Remember that this is only for *local variables*. It's even more confusing because sometimes we're invoking methods, but they look just like local variables if you omit the parentheses. Always look at where the local variable was initialized to determine its scope, and to verify that it's indeed a local variable we're dealing with, and not a method. If it's a method, it doesn't follow this rule.

#### Closure and binding

A block is how Ruby implements the idea of a *closure*, which is a general computing concept of a "chunk of code" that you can pass around and execute at some later time. In order for this "chunk of code" to be executed later, it must understand the surrounding context from when it was initialized. In Ruby, this "chunk of code" or closure is represented by a Proc object, a lambda, or a block. Remember that blocks are a form of Proc. Let's take a look at an example:

```ruby
name = "Robert"
chunk_of_code = Proc.new {puts "hi #{name}"}
```

If you try to run that code, nothing will happen. That's because we've created a Proc and saved it to `chunk_of_code`, but haven't executed it yet. We can now pass around this local variable, `chunk_of_code`, and execute it at any time later. Suppose we have a completely different method, and we pass in this `chunk_of_code` into that method, then the method executes that `chunk_of_code`. 

```ruby
def call_me(some_code)
  some_code.call            # call will execute the "chunk of code" that gets passed in
end

name = "Robert"
chunk_of_code = Proc.new {puts "hi #{name}"}

call_me(chunk_of_code)
```

The output of the above is:

```ruby
hi Robert
=> nil
```

Note that the `chunk_of_code` knew how to handle `puts #{name}`, and specifically that it knew how to process the `name` variable. How did it do that? The `name` variable was initialized *outside* the method definition, and we know that in Ruby local variables initialized outside the method aren't accessible inside the method, unless it's explicitly passed in as an argument. So again, how did that code we pass in know how to process the `name` variable?

 Proc keeps track of its surrounding context, and drags it around wherever the chunk of code is passed to. In Ruby, we call this its **binding**, or surrounding environment/context. A closure must keep track of its surrounding context in order to have all the information it needs in order to be executed later. This not only includes local variables, but also method references, constants and other artifacts in your code -- whatever it needs to execute correctly, it will drag all of it around.

####Write Methods That Use Blocks and Procs

Every method you have ever written in Ruby already takes a block.

One way that we can invoke the passed-in block argument from within the method is by using the `yield` keyword.

```ruby
def echo_with_yield(str)
  yield
  str
end

echo_with_yield("hello!") { puts "world" }             # world
                                                       #=>"hello!"
```

1. The number of arguments at method invocation needs to match the method definition, regardless of whether we are passing in a block.
2. The `yield` keyword executes the block.

####Methods with an explicit block parameter

There are times when you want a method to explicitly require a block; you do that by defining a parameter prefixed by an `&` character in the method definition:

```ruby
def test(&block)
  puts "What's &block? #{block}"
end
```

The `&block` is a special parameter that converts the block argument to what we call a "simple" `Proc` object. Notice that we drop the `&` when referring to the parameter inside the method. Now let's invoke the method and see what happens:

```ruby
test { sleep(1) }

# What's &block? #<Proc:0x007f98e32b83c8@(irb):59>
# => nil
```

As you can see, the `block` local variable is now a `Proc` object. Note that we can name it whatever we please; it doesn't have to be `block`, just as long as we define it with a leading `&`.

 Why do we now need an explicit block instead? 

--> Additional flexibility.  Before, we didn't have a handle (a variable) for the implicit block, so we couldn't do much with it except yield to it and test whether a block was provided. Now we have a variable that represents the block, so we can *pass the block to another method*:

```ruby
def test2(block)
  puts "hello"
  block.call                    # calls the block that was originally passed to test()
  puts "good-bye"
end

def test(&block)
  puts "1"
  test2(block)
  puts "2"
end

test { puts "xyz" }
# => 1
# => hello
# => xyz
# => good-bye
# => 2
```

Note that you only need to use `&` for the `block` parameter in `#test`. Since `block` is already a `Proc` object when we call `test2`, no conversion is needed.

Note that we also use `block.call` inside `test2` to invoke the `Proc` instead of `yield`.



####Arguments and return values with blocks

###### What if you pass the wrong number of arguments to a block?

If you pass too many arguments to a block, the extra block arguments are ignored:

```ruby
# method implementation
def test
  yield(1, 2) # passing 2 block arguments at block invocation time
end

# method invocation
test { |num| puts num } 
# expecting 1 parameter in block implementation
# => 1
```

If you don't pass enough arguments to a block, the extra block local variables are assigned `nil`. 

The rules around enforcing the number of arguments you can call on a closure in Ruby is called its *arity*. In Ruby, blocks have lenient arity rules, which is why it doesn't complain when you pass in different number of arguments

Blocks don't enforce argument count, unlike normal methods in Ruby

###### Return value of yielding to the block

Blocks have a return value, and that return value is determined based on the last expression in the block. This implies that just like normal methods, blocks can either mutate the argument with a destructive method call or the block can return a value.

####When can you pass a block to a method

#### `&:symbol`

```ruby
[1, 2, 3, 4, 5].map(&:to_s)  
```



### Testing With Minitest

#### Testing terminology

#### Minitest vs. RSpec

#### SEAT approach

#### Assertions



### Core Tools/Packaging Code

#### Purpose of core tools

#### Gemfiles



### Regular Expressions

Regular expressions aren't officially a part of this assessment nor are they covered in detail in the course material. You will find them useful, though, when you work on the practice Coding Challenge problems. Before you begin the practice challenges, we recommend that you take some time to study our [Introduction to Regular Expressions](https://launchschool.com/books/regex) book.













