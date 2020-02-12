After some years and differents experiences with PHP and some frameworks like WordPress, Laravel, Symfony and Phalcon, I had the opportunity to work with Elixir and since then it has been my go to language for new projects.

For those who have never had a previous experience with a functional language this first contact could be a little confusing, that was my case when I first start with Elixir.

**The "help" offered in this article by PHP it's just a merely comparison made between the two languages, which by the way are NOTHING similar.**

In this article I've just implemented the same functionalities using both languages so you can compare the **logic** of new programming paradigm with a more familiar one offered by PHP.

I will approach the Elixir functionalities that I most use on daily bases applying the 80/20 rule, but I'm sure that with this simple knowledge you could read a `.ex` or `.exs` file and don't lose yourself on the way.

#### Before we begin...
If you don't know Elixir at all I suggest that you read at least the [_Starting Guide_](https://elixir-lang.org/getting-started/introduction.html) on their docs. It will show the syntax, the primitives types and operators of the language.

Without further a due, let's dive in!

## <center>Pattern Matching</center>

The best _pattern matching_ definition is the following:

> Pattern matching is a powerful part of Elixir. It allows us to match simple values, data structures, and even functions - [Elixir School](https://elixirschool.com/en/lessons/basics/pattern-matching/)

But what exactly does this mean? To better understand this we need to take a look at the _match operator_ `=`. This operator can attribute values to variables and can be used to decompose more complex structures, like tuples and lists.

```sh
# iex

iex(1)> {a, b, c} = {"São Paulo", "Rio de Janeiro", "Fortaleza"}
{"São Paulo", "Rio de Janeiro", "Fortaleza"}

iex(2)> a
"São Paulo"

iex(3)> b
"Rio de Janeiro"

iex(4)> c
"Fortaleza"

# Another example
iex(5)> {_, "Ecuador", country} = {"Brasil", "Ecuador", "Chile"}
{"Brasil", "Ecuador", "Chile"}

iex(6)> country
"Chile"
```
  
Also, you can fetch the first item of a list
```sh
# iex

iex(1)> [head | tail] = ["Olá", "Hello", "Hola"]
["Olá", "Hello", "Hola"]

iex(2)> head
"Olá"

iex(3)> tail
["Hello", "Hola"]
```
  
Case a tuple can't be match against the left hand side of the operator, that means, maybe a tuple don't have the same number of fields or the same expected value of the left side of the `=` a exception will occur:
```sh
# iex

iex(1)> {a, b} = {"São Paulo", "Rio de Janeiro", "Fortaleza"}
# ** (MatchError) no match of right hand side value: {"São Paulo", "Rio de Janeiro", "Fortaleza"}
#    (stdlib) erl_eval.erl:453: :erl_eval.expr/5
#    ...

iex(1)> {_, "United States", country} = {"Brasil", "Ecuador", "Chile"}
# ** (MatchError) no match of right hand side value: {"Brasil", "Ecuador", "Chile"}
#    (stdlib) erl_eval.erl:453: :erl_eval.expr/5
#    ...
```

The above code gave us an understanding of the first half of the _pattern matching_ definition. What about the other half that allow us to look for patterns in functions definitions? Let's take a look:

To help with this part I'll create a PHP class so you can compare with the equivalent Elixir module. To start with let's just create a class and module that have a `hello/1` function that receive the `user_name` as parameter.

PHP class
```php
<?php
// ./GreetUser.php

class GreetUser
{
    public function hello($userName)
    {
        echo "Hello, " . $userName;
    }
}

$greetUser = new GreetUser;
$greetUser->hello("Joe");
```

Elixir module
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello(user_name) do
        IO.puts("Hello, #{user_name}")
    end

end

GreetUser.hello("Joe")
```

So far, so good, right? We have a `hello/1` function that will print "Hello, Joe" both in PHP class and in Elixir module.

But let's assume that the `user_name` parameter could be `nil` (`null` in PHP). How can we adjust our code to attend this requirement?

PHP class
```php
<?php
// ./GreetUser.php

class GreetUser
{
    public function hello($userName = null)
    {
        if(is_null($userName)) {
            echo "Hello world!";
            return;
        }
        echo "Hello, " . $userName;
    }
}

$greetUser = new GreetUser;
$greetUser->hello("Joe");
$greetUser->hello(null);
```

Using PHP we could add a default value to the `$userName` and handle this with a `if` statement inside our function in case `$userName` is really  `null` and display a different message.

If execute `GreetUser.php` we would get:
```php
"Hello, Joe"
"Hello world!"
```

Although it's possible to do the same thing with our module in Elixir we can use _pattern matching_ to remove function complexity by get rid of the `if` statement:
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello(nil) do
        IO.puts("Hello world!")
    end
    def hello(user_name) do
        IO.puts("Hello, #{user_name}")
    end

end

GreetUser.hello("Joe")
GreetUser.hello(nil)
```

As there are two functions with the same name, the one chosen will be that has the correct parameter value.

This means that only when `user_name = nil` the first function will be executed, in any other case it'll skip this function definition and go to the next `hello/1` declared within this module.

> It's important to say that due to function arity `GreetUser.hello(nil)` is different then `GreetUser.hello()`. In the former we would call `hello/1` and the last we would call `hello/0` which does not exists and would break. Always have the arity in mind!

Any primitive of the language can be used in _pattern matching_. So let's assume that the `GreetUser` module now have to greet a specific message to users that have the name `"Jane"`, we could achieve this by doing this:

PHP Class
```php
<?php
// ./GreetUser.php

class GreetUser
{
    public function hello($userName)
    {
        if($userName === "Jane") {
            echo "Nice to see you, Jane!";
            return;
        }
        echo "Hello, " . $userName;
    }
}

$greetUser = new GreetUser;
$greetUser->hello("Joe");
$greetUser->hello("Jane");

```

Elixir Module
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello("Jane" = user_name) do
        IO.puts("Nice to see you, #{user_name}!")
    end
    def hello(user_name) do
        IO.puts("Hello, #{user_name}")
    end

end

GreetUser.hello("Joe")
GreetUser.hello("Jane")
```

This time we specified the `user_name` value and add the _match operator_ to the parameter: `("Jane" = user_name)`. By doing this we are assigned `user_name` to `"Jane"` and them we can use the variable inside our function. If you don't want to use the parameter inside your function you can dismiss it by prepending it with an _underscore_: `("Jane" = _user_name)`. With that Elixir understand that this variable will never be used.

**Declared variables that are not been used will raise a warning in compilation time, keep that in mind!**

I hope that with this example you began to see how _pattern matching_ can remove complexity away from our code. Doesn't Elixir module look more simple and better to read than the class in PHP? We no longer have to "calculate" `if`s statement to know what a function will return, pretty good, right?
  
## <center>Guards</center>

If you enjoy the wonders of _pattern matching_ you can go beyond using _guards_. It will allow you to make more complex parameter validation in your functions declaration.

Guards are defined using a `when` after a function declaration. While _pattern matching_ is useful to check explicit values, with _guards_ we can apply some boolean logic to it. We can check a parameter's type, length, if it is `nil`, if it is higher or lower given a determined number and some others validations.

Back to our `GreetUser`, we will use _guard_ to validate if `user_name` is a `string`.

Elixir module
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello(user_name) when is_binary(user_name) do
        IO.puts("Hello, #{user_name}")
    end

end

GreetUser.hello("Joe")
```

The `is_binary/1` function will return a boolean value and according to it (`true` or `false`) `hello/1` get invoked or not. Elixir has several [type-checks](https://hexdocs.pm/elixir/Kernel.html#is_atom/1)” functions like this so you can check a expression type.

> Here is a tip. Either using a _pattern matching_ or _guard_ solution it's always a good practice to at least think about a alternative in case of a parameter mismatch. You could get a runtime error if a parameter won't match a function

    ** (FunctionClauseError) no function clause matching in GreetUser.hello/1

The above error will happen if we call `hello/1` with an integer: `GreetUser.hello(1)`.

As an alternative we could add another declaration of `hello/1` just dismissing the parameter, so everything that is not a `string` will be direct to it:

```elixir
# ./greet_user.exs

defmodule GreetUser do
   
    def hello(user_name) when is_binary(user_name) do
        IO.puts("Hello, #{user_name}")
    end
    def hello(_) do
        IO.puts("Only string is allowed in this function")
    end

end

GreetUser.hello("Joe")
GreetUser.hello(1)
```

##### Important!
**The function declaration order is extreme important!!**
In the above example we restrict `hello/1` to only accept `string`. No problem there, but what if we had reversed the function declaration orders by placing `hello (_)` before the `hello / 1` that has the _guard_?

```elixir
# ./greet_user.exs

defmodule GreetUser do
   
    def hello(_) do
        IO.puts("Only string is allowed in this function")
    end
    def hello(user_name) when is_binary(user_name) do
        IO.puts("Hello, #{user_name}")
    end

end

GreetUser.hello("Joe")
GreetUser.hello(1)
```

Running the code we would get the following:
    warning: this clause cannot match because a previous clause at line 5 always matches

Our code still run with no errors, but `hello/1` with the _guard_ `when is_binary/1` would never be called because `hello/1` without the guard will ALWAYS be evaluated first! So pay attention to function's declaration order!!
