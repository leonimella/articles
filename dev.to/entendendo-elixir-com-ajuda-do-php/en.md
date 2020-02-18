🇧🇷 _Artigo original em português: [Entendendo Elixir com a ajuda de PHP](https://dev.to/leonimella/entendendo-elixir-com-ajuda-de-php-20n2)_

After some years and differents experiences with PHP and some frameworks like WordPress, Laravel, Symfony and Phalcon, I had the opportunity to work with Elixir and since then it has been my go to language for new projects.

For those who have never had a previous experience with a functional language this first contact could be a little confusing, that was my case when I first start with Elixir.

**The "help" offered in this article by PHP it's just a merely comparison made between the two languages, which by the way are NOTHING similar.**

In this article I've just implemented the same functionalities using both languages so you can compare the **logic** of a new programming paradigm with a more familiar one offered by PHP.

I will approach the Elixir functionalities that I most use on a daily basis applying the 80/20 rule, but I'm sure that with this simple knowledge you can read a `.ex` or `.exs` file and don't lose yourself on the way.

#### Before we begin...
If you don't know Elixir at all I suggest that you read at least the [_Starting Guide_](https://elixir-lang.org/getting-started/introduction.html) on their docs. It will show the syntax, the primitives types and operators of the language.

Without further ado, let's dive in!

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

Using PHP we could add a default value to the `$userName` and handle this with a `if` statement inside our function in case `$userName` is really `null` and display a different message.

If execute `GreetUser.php` we would get:
```php
"Hello, Joe"
"Hello world!"
```

Although the same solution is possible in Elixir, we can use _pattern matching_ and remove the complexity of the code from our function by get rid of the `if` statement:
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

As there are two functions with the same name, the chosen one will be the one that has the correct parameter value.

This means that only when `user_name = nil` the first function will be executed, in any other case it will skip this function definition and go to the next `hello/1` declared within this module.

> It's important to say that due to function arity `GreetUser.hello(nil)` is different then `GreetUser.hello()`. In the former we would call `hello/1` and the last we would call `hello/0` which does not exists and would break. **Always have the arity in mind**!

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

This time we specified the `user_name` value and add the _match operator_ to the parameter: `("Jane" = user_name)`. By doing this we are matching `user_name` to `"Jane"` and them we can use the variable inside our function. If you don't want to use the parameter inside your function you can dismiss it by prepending it with an _underscore_: `("Jane" = _user_name)`. Elixir understand that this variable will never be used.

**Declared variables that are not used will raise a warning in compilation time, keep that in mind!**

I hope that with this example you began to see how _pattern matching_ can remove complexity away from our code. Doesn't Elixir module look more simple and better to read than the class in PHP? We no longer have to "calculate" `if`s statement to know what a function will return, pretty good, right?
  
## <center>Guards</center>

If you enjoy the wonders of _pattern matching_ you can go beyond using _guards_. It will allow you to make more complex parameter validation in your functions declaration.

_Guards_ are defined using a `when` after a function declaration. While _pattern matching_ is useful to check explicit values, with _guards_ we can apply some boolean logic to it. We can check a parameter's type, length, if it is `nil`, if it is higher or lower given a determined number and some others validations.

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

> Here is a tip. Either using a _pattern matching_ or _guard_ it's always a good practice to at least think about a alternative in case of a parameter mismatch. You could get a runtime error if a parameter won't match a function

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
**The function declaration order is extremely important!!**
In the above example we restrict `hello/1` to only accept `string`. No problem there, but what if we had reversed the function declaration orders by placing `hello(_)` before the `hello(user_name) when is_binary(user_name)`?

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

Our code still run with no errors, but `hello/1` with the _guard_ `when is_binary/1` would never be called because `hello/1` without the guard will **ALWAYS** be evaluated first! **So pay attention to function's declaration order**!!

## <center>Pipe Operator</center>

This one is a bit trick, but in a nutshell the _pipe operator_ is represented by the symbol `|>` and all that it does is pass an expression evaluation (or result) as the first parameter to the next one.

With this you can create a function _pipeline_ (get the name?) given a expression (variable or function).

To illustrate this concept, now `GreetUser` has a new requirement, we'll need to greet some users which their names will came in a list. To do that we'll create another function named `hello_group/1`. This new function will receive the list of names as `users_name`.

```elixir
# ./greet_user.exs

defmodule GreetUser do
    def hello_group(users_name) when is_list(users_name) do
        # code
    end
    def hello_group(_) do
        IO.puts("Only lists are allowed")
    end
end
```

We declared the function and added a _guard_ so we can validate that the parameter is indeed a list, and defined a "default" `hello_group/1` in the chance that `users_name` is not a list.

The next step it's to handle the `users_name` using the _pipe operator_. First we'll go through the list and normalize the names. The first part can be achieve using the `map/2` function from Elixir's `Enum` module.
```elixir
# ./greet_user.exs

defmodule GreetUser do
    def hello_group(users_name) when is_list(users_name) do
        users_name
        |> Enum.map(fn user_name -> String.capitalize(user_name) end)
    end
    def hello_group(_) do
        IO.puts("Only lists are allowed")
    end
end
```

Let's breakdown this code:
- We called `users_name` inside our function
- Then we added the `|>` (_pipe operator_) bellow `users_name`. It could be called by the right side as well.
- To loop the list, we use the `map/2`, that will return another list.
- As `map/2` second parameter we gave a anonymous function `fn user_name -> ... end` that will be executed for each element on the list
- Inside the anonymous function we called the [`capitalize/2`](https://hexdocs.pm/elixir/String.html#capitalize/2) from Elixir `String` module to capitalize the names.

The bit that you need understand is that we pass only `map/2` second parameter because the first parameter is passed by the `|>`.

Remember: "The _pipe operator_ forwards the result of an expression as the first parameter on to the next expression". In our case the first expression is the variable `users_name` and when it gets evaluated will return the list of names. And our pipeline worked because `map/2` expected as the first parameter a list!

Using PHP we could achieve the same with this code:
```php
<?php
// ./GreetUser.php

class GreetUser
{
    public function helloGroup($usersName)
    {
        if(!is_array($usersName)) {
            echo "Only arrays are allowed";
            return;
        }

        $usersNameNormalized = array_map(function ($name) {
            return ucfirst($name);
        }, $usersName);
    }
}
```

The next step is to "glue" all the names in a single `string`. We'll use `join/2` which is another function from `Enum` module. This function will receive in the first parameter a list and in the second parameter receives a string as the joiner element. We'll use just a `", "` to separate the names.

Elixir module
```elixir
# ./greet_user.exs

defmodule GreetUser do
    def hello_group(users_name) when is_list(users_name) do
        users_name
        |> Enum.map(fn user_name -> String.capitalize(user_name) end)
        |> Enum.join(", ")
    end
    def hello_group(_) do
        IO.puts("Only lists are allowed")
    end
end
```

PHP class
```php
<?php
// ./GreetUser.php

class GreetUser
{
    public function helloGroup($usersName)
    {
        if(!is_array($usersName)) {
            echo "Only arrays are allowed";
            return;
        }

        $usersNameNormalized = array_map(function ($name) {
            return ucfirst($name);
        }, $usersName);

        $usersNameString = implode(", ", $usersNameNormalized);
    }
}
```

Our pipeline still functions because `map/2` will return a new list and the `|>` forwards it as the first parameter to the `join/2` function.

Great! `hello_group/1` is able to normalize and glue together the names. Now we just need to print a message to the user.

So far the result of `hello_group/1` would be a string with the users names. We could use `hello/1`, which receives a string as first parameter to print the message to our user.
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello(user_name) when is_binary(user_name) do
        IO.puts("Hello, #{user_name}")
    end
    def hello(_) do
        IO.puts("Only strings are allowed")
    end

    def hello_group(users_name) when is_list(users_name) do
        users_name
        |> Enum.map(fn user_name -> String.capitalize(user_name) end)
        |> Enum.join(", ")
        |> hello
    end
    def hello_group(_) do
        IO.puts("Only lists are allowed")
    end
end

GreetUser.hello_group(["joe", "jane", "jim"])
```

By adding `hello/1` in the pipeline we fulfill all the requirements for `hello_group/1`.

Take a look at the same feature in PHP
```php
<?php
// ./GreetUser.php

class GreetUser
{
    public function hello($userName)
    {
        echo "Hello, " . $userName;
    }

    public function helloGroup($usersName)
    {
        if(!is_array($usersName)) {
            echo "Only arrays are allowed";
            return;
        }

        $usersNameNormalized = array_map(
            function ($name) { return ucfirst($name); },
            $usersName
        );

        $usersNameString = implode(", ", $usersNameNormalized);

        return $this->hello($usersNameString);
    }
}

$greetUser = new GreetUser;
$greetUser->helloGroup(["joe", "jane", "jim"]);

```

We could make the above code better by splitting in different functions, but do notice that even if we make such change the code in Elixir has a LOT more readability and less complexity.

Here is another way to visualize the same code without using the _pipe operator_:
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello(user_name)do
        IO.puts("Hello, #{user_name}")
    end

    def hello_group(users_name) do
        users_name
        |> Enum.map(fn user_name -> String.capitalize(user_name) end)
        |> Enum.join(", ")
        |> hello
    end

    def hello_group(users_name, :notpiped) do
        hello(
            Enum.join(
                Enum.map(
                    users_name,
                    fn user_name -> String.capitalize(user_name) end),
                ", "
            )
        )
    end
end

GreetUser.hello_group(["joe", "jane", "jim"])
GreetUser.hello_group(["joe", "jane", "jim"], :notpiped)
```

Here we defined another function: `hello_group/2`, despite the arity, both functions will return the same result.
But `hello_group/1` is cleaner and easier to understand.

## <center>Control Structures</center>

In Elixir we have access to certain tools that allow us to remove complexity from our code, but sometimes we have no other option and we have to use a `if` or a `switch`.
The control structures available to us are: `if`, `unless`, `case`, `cond` and `with`. This article is an overview of the language and because of that I'm not write about all this structures. I'll focus in the `with` because this one can be harder to understand than the others.

The first thing that we need to understand is that in a functional language the result of a function is always the result of the last expression that get executed, we don't have the `return` keyword in this functional world. Because of this, control structures will return the result of the last expression, so you need to be careful when write them. This can be challenging at first, but you get used to it quite fast.

In a functional language we cannot do some like this:
```php
public function anotherFunction() {
    $param = null;

    if ($condition) {
        $param = "PHP";
    }

    return $param;
}
```

In the above PHP code we are defining a default value to `$param` and based on `$condition`'s his value will change. This wouldn't work in a Elixir code because de `if` is an expression! If this was an `Elixir` code, all that this if expression was doing is to match `$param` to `"PHP"`, and the return of this expression would be an `:ok` atom. The `$param` variable would never match `"PHP"` outside the `if` statement.

To this work in Elixir, one could do as the following:
```elixir
def another_function do
    param = if condition do
        "Elixir"
    else
        nil
    end
end
```

By doing this, the **return** of the `if` would be matched to the `param` variable.
With that in mind, let's move on.

## With

You can think in `with` like a pipeline that checks the result of each expression and if some of those return an unexpected value the code inside the `with` block won't get executed. You can also add fallback clausules to handle some unexpected behaviour or errors.

Let's add this structure in our `hello/1` function and see how it works:
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def dummy_function, do: :ok

    def hello(user_name)do
        with :ok <- dummy_function() do
            IO.puts("Hello, #{user_name}")
        end
    end

end

GreetUser.hello("Joe")
```

We created an `dummy_function/0` that will only return an `:ok` atom and we added an `with` block to check the returned value. If the returned value was indeed an `:ok` atom, them we print `"Hello, Joe"` on the screen.

If `dummy_function/0` won't return an `:ok` we could add a fallback clausule to handle this:
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def dummy_function, do: :error

    def hello(user_name)do
        with :ok <- dummy_function() do
            IO.puts("Hello, #{user_name}")
        else
            _ ->
                IO.puts("dummy_functtion didn't return an :ok atom")
        end
    end

end

GreetUser.hello("Joe")
```

We can match variables and other structures on the left side of the `<-` operator, so we can use the value returned by the function inside de `with` block:
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def dummy_function, do: {:ok, "dummy function"}

    def hello(user_name)do
        with {:ok, _response} <- dummy_function() do
            IO.puts("Hello, #{user_name}")
        else
            {:error, message} ->
                IO.puts("dummy_functtion return an error: #{message}")
            _ ->
                IO.puts("I can't figure what dummy_function has returned")
        end
    end

end

GreetUser.hello("Joe")
```

We checked if `dummy_function/0` will return an `{:ok, response}` result, if it does the function proceed and print the greeting. If `dummy_function/0` returns an `{:error, message}` it prints the message with the error. And at last if `dummy_function/0` returns something unexpected the last clausule, which matches everything, will be executed printing the generic error message. The last clausule will not match on the other cases (`{:ok, response}` and `{:error, message}`) because the others two clausules will match first, remember, **the declaration other matter**!

With this knowledge we can create an _pipeline_ of functions that will be validated in each step and handle any possible error individually:
```elixir
def hello(user_name)do
    with {:ok, response1} <- dummy_function(),
         {:ok, _response2} <- dummy_function_2(reponse1) do
            IO.puts("Hello, #{user_name}")
    else
        {:error, message} ->
            IO.puts("dummy_functtion return an error: #{message}")
        _ ->
            IO.puts("I can't figure what dummy_function has returned")
    end
end
```

## <center> And there you have it!</center>
If you can wrap your mind around these Elixir's concepts I'm guarantee that you will have the basic knowledge to handle quite some understanding of an Elixir's code.

Here are some topics for you to get deeper into the language and its ecosystem

#### Mix
_Task runner_ (similar to _composer_ on PHP), link to the [guide](https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html) and [docs](https://hexdocs.pm/mix/Mix.html)
#### Phoenix
Web framework (similar to _Symfony_, _Laravel_, etc... on PHP), link to the [site](https://www.phoenixframework.org/) and [docs](https://hexdocs.pm/phoenix/Phoenix.html)

#### Ecto
ORM (similar to _Doctrine_ on php), link to the [guide](https://elixirschool.com/pt/lessons/ecto/basics/) and [docs](https://hexdocs.pm/ecto/Ecto.html)
#### Hex
_Package repository_ (similar to _Packgist_ of _Composer_), link to the [site](https://hex.pm/)

## <center> References </center>
And here, some usefull links and references that I used to write this article:

- Official website: https://elixir-lang.org/
- Elixir School: https://elixirschool.com/pt/
- Tiny documentary of the language: https://www.youtube.com/watch?v=lxYFOM3UJzo
- Principal evento da linguagem no Brasil: https://twitter.com/elixir_brasil
- Twitter: https://twitter.com/elixirlang

I hope that you enjoyed read this article and that was useful to you. If you have any suggestion or feedback leave a comment. This is one of my first articles and I would like to know how I'm doing :grin:

Thank you for your time! See you soon :vulcan_salute:
