# Entendendo Elixir com ajuda de PHP

Depois de alguns anos e diversas experiências em PHP utilizando WordPress, Laravel, Symfony e Phalcon, tive a oportunidade de trabalhar com Elixir e desde então ela tem sido a minha linguagem para desenvolvimento de novos projetos

Para quem nunca teve contato com uma linguagem funcional pode ser um pouco difícil entender como Elixir funciona, foi o meu caso quando comecei a utilizá-la.

**A “ajuda” oferecida por PHP, nesse artigo, não passa de uma mera comparação feita entre as duas linguagens que por sinal não são NADA semelhante.**

O que fiz foi implementar a mesma funcionalidade com linguagens diferentes para que você possa comparar a **lógica** de uma linguagem mais familiar com uma que talvez não seja.

**Vou abordar as funcionalidades do Elixir que mais utilizo no dia-a-dia**, meio que aplicando a regra 80/20, mas tenho certeza que com esse conhecimento você poderá ao menos ler um arquivo `.ex` ou `.exs` e não se perder completamente, caso você seja novo na linguagem.


## <center>First things first!</center>

Caso você não conheça a syntax da linguage, seus operadores e primitivos sugiro a leitura do [_Starting Guide_](https://elixir-lang.org/getting-started/introduction.html) na documentação oficial do Elixir.

Não precisa necessariamente ler por completo já que esse artigo usa o guia, citado acima, como referência.

Sem mais demoras, vamos ao código!

## <center>Pattern Matching</center>

Acho que a melhor definição do conceito de _pattern matching_ é a seguinte:

> Pattern matching é uma poderosa parte de Elixir que nos permite procurar padrões simples em valores, estruturas de dados, e até funções - [Elixir School](https://elixirschool.com/pt/lessons/basics/pattern-matching/)

Mas o que exatamente significa? Para entender melhor precisamos falar sobre o _match operator_ `=`. Além de utilizado para atribuir valores à variáveis, ele também pode ser usado para decompor estruturas mais complexas como tuplas.

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
iex(5)> {_, "Equador", country} = {"Brazil", "Equador", "Chile"}
{"Brazil", "Equador", "Chile"}

iex(6)> country
"Chile"
```

Ou obter o primeiro valor de uma lista

```sh
# iex

iex(1)> [head | tail] = ["Olá", "Hello", "Hola"]
["Olá", "Hello", "Hola"]

iex(2)> head
"Olá"

iex(3)> tail
["Hello", "Hola"]
```

Caso uma tupla não possa ser equiparada com o lado esquerdo do operador, isso é, caso a tupla tenha número de campos diferente ou algum valor que não condiz com o lado esquerdo do `=` uma exception acontecerá:

```sh
# iex

iex(1)> {a, b} = {"São Paulo", "Rio de Janeiro", "Fortaleza"}
# ** (MatchError) no match of right hand side value: {"São Paulo", "Rio de Janeiro", "Fortaleza"}
#    (stdlib) erl_eval.erl:453: :erl_eval.expr/5
#    ...

iex(1)> {_, "United States", country} = {"Brazil", "Equador", "Chile"}
# ** (MatchError) no match of right hand side value: {"Brazil", "Equador", "Chile"}
#    (stdlib) erl_eval.erl:453: :erl_eval.expr/5
#    ...
```

Os exemplos acima exemplificam a primera parte da definição de _pattern matching_. E quanto a parte que fala que podemos procurar padrões em funções? Vamos dar uma olhada.

Para ajudar no entendimento dessa vez vou criar uma classe semelhante em PHP.

Classe PHP
```php
# ./GreetUser.php

<?php

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

Módulo Elixir
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello(user_name) do
        IO.puts("Hello, #{user_name}")
    end

end

GreetUser.hello("Joe")
```


Até aqui, sem novidades, certo? Se chamarmos a função `hello/1` ela vai exibir na tela a mensagem “Hello, Joe”. Mesma coisa com a função `hello()` da classe em PHP.

Mas vamos supor que nem sempre eu teria o nome do meu usuário disponível. Em determinadas vezes a variável `user_name` poderia ser `nil` (`null` em PHP). Como podemos ajustar nosso código para esse requisito?

Classe PHP
```php
# ./GreetUser.php

<?php

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

Em PHP poderíamos adicionar um valor default para o parâmetro `$userName` e tratar esse caso com um `if` dentro da nossa função exibindo uma mensagem diferente caso o argumento seja `null`.

Executando `GreetUser.php` obteríamos o resultado:
```php
"Hello, Joe"
"Hello world!"
```

Embora a mesma solução seja possível, em Elixir podemos utilizar o _pattern matching_ ao nosso favor para simplificar o nosso código sem a necessidade do `if`.

Módulo Elixir
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
Como existem duas funções com o mesmo nome a escolhida será aquela que possuir o parâmetro correto.

Isso quer dizer que somente quando `user_name = nil` é que a primeira função será executada, caso contrário ele “pula” essa função e vai para a próxima `hello/1` declarada neste módulo.

>Importante dizer que devido à aridade da função, `GreetUser.hello(nil)` é diferente de `GreetUser.hello()`. Em uma expressão invocamos `hello/1` e na outra `hello/0`, que não existe e portanto nos retornaria um erro, fique atento a aridade!

Qualquer tipo primitivo da linguagem pode ser utilizado para _pattern matching_. Se o módulo `GreetUser` agora tem que enviar uma mensagem específica para usuários que tenham o nome `"Jane”`, poderíamos fazer da seguinte forma

Classe em PHP
```php
# ./GreetUser.php

<?php

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

Módulo Elixir
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

Dessa vez além de especificarmos qual seria o valor de `user_name` nós também adicionamos o _match operator_ no parâmetro da função: `("Jane" = user_name)`. Fizemos isso para que possamos utilizar o valor dentro da função. Caso não quisessemos utilizar o parâmetro podemos omiti-lo `("Jane")` ou adicionar um `_` _underscore_ na frente dele `("Jane" = _user_name)`. Com isso o Elixir entende que você está desprezando esse valor e que ele não será utilizado.

**Variáveis declaradas, mas não utilizadas recebem um** `warning` **no momento da compilação do seu código, fique atento!**

Aqui já é possível enxergar um pouco melhor como esse conceito reduz a complexidade em nossas funções. Agora que você entende pattern matching não fica melhor ler a função em Elixir do que em PHP? Não precisamos mais ficar “calculando” `if`s para saber qual o resultado esperado, uma bela vantagem, não?


## <center>Guards</center>

Se você gostou do conceito de _pattern matching_ você pode ir além utilizando _guards_. Eles permitem verificações mais completas dos parâmetros que sua função está recebendo.

Guards são definidos com um `when` após a declaração da função. Enquanto o _pattern matching_ é útil para valores explícitos, com _guards_ podemos criar diversas condições para serem checadas. Podemos checar o tipo de um parâmetro, o tamanho dele se ele é ou não `nil`, se ele é maior ou menor que determinado valor entre outras.

Voltando ao nosso exemplo `GreetUser`, podemos utilizar um _guard_ para verificar se `user_name` é uma `string`.

Módulo Elixir
```elixir
# ./greet_user.exs

defmodule GreetUser do

    def hello(user_name) when is_binary(user_name) do
        IO.puts("Hello, #{user_name}")
    end

end

GreetUser.hello("Joe")
```

A função utilizada no guard `is_binary/1` retorna um booleano e com esse resultado (`true` ou `false`) `hello/1` é invocada ou não. Elixir possui diversas funções “[type-checks](https://hexdocs.pm/elixir/Kernel.html#is_atom/1)” como essa que usamos para checar tipos.

Lembrando, que usando _guard_ ou _pattern matching_ você precisa pensar em uma alternativa para o caso em que um parâmetro não se enquadre em nenhuma validação caso contrário um _runtime error_ será lançado

    ** (FunctionClauseError) no function clause matching in GreetUser.hello/1

O Erro acima aconteceria se nós invocarmos com um inteiro, por exemplo:`GreetUser.hello(1)`.

Como alternativa podemos adicionar mais uma declaração de `hello/1` despresando o parâmetro `user_name`. Como vimos antes, basta adicionarmos o _undersocre_ na frente do nome do parâmetro: `_user_name`. Ou simplesmente subistituirmos pelo `_`.

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

##### Importante!
    A ordem de declaração das funções em linguagens funcionais é de extrema importância!
No exemplo acima o uso de `hello/1` ficou restrita à parâmetros do tipo `string`. Como alternativa declaramos novamente `hello/1` **despresando** o parâmetro `user_name`. Até ai tudo legal, mas e se tivessemos invertido as ordens de declaração das funções colocando `hello(_)` antes da `hello/1` que possui _guard_?

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

Executando o código acima teríamos o seguinte aviso:

    warning: this clause cannot match because a previous clause at line 5 always matches

Nosso código rodaria sem problemas, mas a função `hello/1` com o _guard_ `when is_binary/1` nunca seria executada porque a `hello/1` sem _guard_ é sempre validada antes. Por isso preste sempre atenção à ordem de declaração de suas funções!

## <center>Pipe Operator</center>

Esse pode ser um pouco mais complicado para entender, mas em resumo o _pipe operator_ é representado por esse símbolo `|>` e o trabalho dele é passar o resultado de uma expressão como primeiro parâmetro de outra expressão.

Com isso você pode criar _pipelines_ (sacou o nome?) de execução dada determinada expressão (variável ou função).

Para exemplificar, agora nosso módulo `GreetUser` deve ser capaz de cumprimentar diversas pessoas cujo os nomes serão enviados em forma de uma lista. Para fazer isso vamos criar uma função `hello_group/1` que recebe como argumento `users_name` que nada mais é do que uma lista contendo o nome dos usuários em string.

Além disso precisamos também tratar os nomes dos usuários para que eles fiquem capitalized, isso é, somente a primeira letra do seu nome em maiúsculo.

```elixir
# ./greet_user.ex

defmodule GreetUser do
    def hello_group(users_name) when is_list(users_name) do
        # code
    end
    def hello_group(_) do
        IO.puts("Only lists are allowed")
    end
end
```

Aproveitamos para adicionar um guard na nossa função para nos certificar de que o parâmetro é mesmo uma lista e também definimos uma `hello_group/1` default para ser utilizada caso `users_name` não seja uma lista.

O próximo passo é manipular a lista aplicando os requisitos acima utilizando o _pipe operator_. Como primeiro passo, vamos percorrer a lista e normalizar os nomes. Faremos isso com a ajuda do módulo `Enum` que é um módulo nativo do Elixir para trabalhar com elementos enumeráveis, o que é o caso da nossa lista de nomes

```elixir
# ./greet_user.ex

defmodule GreetUser do
    def hello_group(users_name) when is_list(users_name) do
        users_name
        |> Enum.map(fn user_name -> String.captalize(user_name) end)
    end
    def hello_group(_) do
        IO.puts("Only lists are allowed")
    end
end
```

Vamos quebrar o código acima em pontos chaves:
- Invocamos `users_name` dentro da nossa função
- Invocamos `|>` (pipe operator) abaixo de `users_name` poderia ser do lado direito também.
- Invocamos `map/2` do módulo `Enum` para percorrer a lista de nomes e nos retornar uma lista
- Passamos como segundo parâmetro de `map/2` uma função anônima `fn users_name -> end` que será executada para cada nome da lista
- Dentro da função anônima utilizamos a função `capitalize/2` do módulo `String` também nativo do Elixir para capitalizar os nomes.

O que precisa ser entendido desse código é que só passamos o segundo argumento para a função `map/2` porque o primeiro argumento que deve ser uma lista (no nosso caso a lista com os nomes dos usuários) já foi passada à essa função pelo `|>`.

Lembre que: “O _pipe operator_ é responsável por passar o valor de uma expressão como o primeiro parâmetro da próxima expressão” No caso nossa primeira expressão é a variável `users_name` que ao ser processada retorna nossa lista de nomes e uma lista é justamente o que `map/2` recebe no primeiro parâmetro, por isso nossa pipeline funciona!

E o segundo parâmetro passado ao `map/2` foi uma função anônima (requerida por `map/2`) que recebe como argumento o elemento da lista que está sendo percorrida para que possamos manipulá-lo.

Em PHP podemos obter o mesmo resultado até aqui dessa forma:

```php
# ./GreetUser.php

<?php

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


O próximo passo nessa função é juntarmos todos os nomes em uma única string. Para isso utilizaremos outra função nativa do Elixir, a `join/2` do módulo `Enum`. Ela recebe como primeiro parâmetro uma lista e como segundo parâmetro um elemento de junção, no nosso caso utilizaremos uma vírgula seguida de espaço em branco, para separar os nomes dentro da string.

Módulo Elixir
```elixir
# ./greet_user.ex

defmodule GreetUser do
    def hello_group(users_name) when is_list(users_name) do
        users_name
        |> Enum.map(fn user_name -> String.captalize(user_name) end)
        |> Enum.join(", ")
    end
    def hello_group(_) do
        IO.puts("Only lists are allowed")
    end
end
```

Classe PHP
```php
# ./GreetUser.php

<?php

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

Nossa pipeline continua funcionando porque `map/2`, após ser executada, retorna uma lista e o `|>` passa essa lista como primeiro parâmetro para a próxima função `join/2`.

Ótimo! `hello_group/1` já consegue normalizar e juntar os nomes enviados como lista em uma única string! Agora só nos resta imprimir nossa mensagem para o usuário.

Até agora o retorno de `hello_group/1` seria uma string contendo os nomes de usuários que foi passado à ela. Podemos utilizar `hello/1`, que recebe como primeiro parâmetro uma string, para exibir a mensagem padrão

```elixir
# ./greet_user.ex

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

Adicionamos `hello/1` no final de nossa pipeline concluindo os requisitos exigidos para `hello_group/1`.

Veja como ficaria o mesmo código em PHP
```php
# ./GreetUser.php

<?php

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

Poderíamos melhorar o código PHP quebrando-o em diferentes funções, mas repare que mesmo dessa forma em Elixir temos um código MUITO mais limpo e legível.

Caso você ainda esteja um pouco confuso em como o _pipe operator_ funciona, aqui está uma outra forma de vizualizar o mesmo código:

```elixir
# ./greet_user.ex

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

Definimos duas funções `hello_group()`. Uma com [aridade](https://elixirschool.com/en/lessons/basics/functions/#function-naming-and-arity) `hello_group/1` e outra com `hello_group/2`. Apesar da aridade diferente, as duas executam o mesmo papel, mas a `hello_group/1` cria uma _pipeline_ enquanto a `hello_group/2` não, mas as duas chagam no mesmo resultado.

Porém `hello_group/1` é bem mais legível e de fácil entendimento se comparada a `hello_group/2`, concordam? Esse é o propósito do _pipe operator_, deixar nosso código mais legível.

Para concluir aqui vai uma observação muito importante! Em linguagens funcionais nós não temos a opção de utilizar `return`s dentro das funções. As funções retornarão o resultado da última expressão executada, vamos explorar mais sobre isso no próximo tópico.


## <center>Estruturas de Controle</center>

Vimos que em Elixir é possível simplificar muito nossas funções removendo condicionais através do _pattern matching_ e _guards_. Mas as vezes não temos outra alternativa a não ser utilizar um `if` ou `switch`.

As estruturas de controles presentes em Elixir são: `if`, `unless`, `case`, `cond` e `with`.

Como esse é um overview da linguagem, não vou me aprofundar em todas elas com exceção do  `with`, talvez ele cause mais estranheza enquanto as outras são mais familiares.

É importante notar que essas estruturas de controles se comportam como funções e por isso alguns hábitos de outras linguagens devem ser evitados em Elixir. Um exemplo disso é quando você define um valor default para uma variável e dentro de um `if` você troca esse valor. Algo parecido com isso:

```php
public function anotherFunction() {
    $param = null;

    if ($condition) {
        $param = "PHP";
    }

    return $param.
}
```

Esse código não funciona do jeito que você espera em Elixir, como `if` se comporta como uma função ela vai retornar o resultado que foi executado dentro do seu body. Sendo assim o valor que você atribuiu dentro dele para a sua variável não será aplicado pois não faz parte do escopo da função.

Para que isso funcionasse você teria que fazer algo desse gênero:

```elixir
def another_function do
    param = if condition do
        "Elixir"
    else
        nil
    end
end
```

Dessa forma o retorno de `if` seria atribuído à nossa variável.

Dito isso, vamos ver como o `with` funciona.

Você pode pensar nele como uma _pipeline_ que checa o resultado de cada expressão e caso o resultado dessa expressão não foi o que você havia descrito ela não executa o código dentro dela. Ou você também pode validar o erro gerado pela expressão para tratá-lo de uma forma mais coerente.

Para ilustrar melhor, vamos adicionar essa estrutura na nossa função `hello/1`. Criaremos também uma `dummy_function/0` que a única coisa que ela faz é retornar o _atom_ `:ok`, e em `hello/1` validaremos se `dummy_function/0` realmente volta `:ok`

```elixir
# ./greet_user.ex

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

No código acima, a mensagem só será exibida caso `dummy_function/0` retornar `:ok`. Se o retorno for outro a mensagem não será exibida.

Se `dummy_function/0` não retornar o esperado, podemos adicionar uma cláusula também para que a gente seja avisado nesse caso.

```elixir
# ./greet_user.ex

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

Podemos também atribuir variáveis do lado esquerdo do `<-` ou `->` que nem fizemos quando vimos sobre _pattern matching_. Com isso é possível verificar o retorno da função e já atribuir uma variável

```elixir
# ./greet_user.ex

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

No código acima checamos se `dummy_function/0` retorna `{:ok, _response}` Caso um erro ocorra e `dummy_function/0` retornar `{:error, "dummy function"}` nós vamos exibir `"dummy function"`, que foi o erro retornado. E ainda adicionamos uma cláusula default se `dummy_function/0` retornar algo totamente inesperado.

Agora que já temos um bom entendimento sobre o _with_ podemos criar uma _pipeline_ de funções cujo os retornos são verificados pelo with e podemos também tratar individualmente cada etapa:


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

O código acima cria uma pipeline que checa passo a passo o retorno das funções possibilitando o tratamento personalizado para cada erro que possa acontecer.


## <center>And there you have it!</center>

Embora Elixir seja um mundo de funcionalidades e paradigmas diferentes se você possuir um bom entendimento dos tópicos acima você conseguirá se localizar muito bem no código escrito nessa linguagem.

Estão alguns tópicos importantes para você que deseja continuar interagindo com a linguagem:

#### Mix
_Task runner_ da linguagem (equivalente ao _composer_ em PHP), link para o [guia](https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html) e [documentação](https://hexdocs.pm/mix/Mix.html)
#### Phoenix
Web framework (equivalente ao _Symfony_, _Laravel_, etc... em PHP), link para o [site](https://www.phoenixframework.org/) e [documentação](https://hexdocs.pm/phoenix/Phoenix.html)

#### Ecto
ORM (equivalente ao _Doctrine_ em php), link para o [guia](https://elixirschool.com/pt/lessons/ecto/basics/) e [documentação](https://hexdocs.pm/ecto/Ecto.html)
#### Hex
_Package repository_ (equivalente ao _Packgist_ do _Composer_), link para o [site](https://hex.pm/)


## <center>Referências</center>

Abaixo uma lista de sites, vídeos e outros materiais para você ler e que foram utilizados para compor esse artigo:

- Site oficial da linguagem: https://elixir-lang.org/
- Para aprender mais: https://elixirschool.com/pt/
- Mini documentário da origem da linguagem: https://www.youtube.com/watch?v=lxYFOM3UJzo
- Hipster ponto tech com o José Valim sobre Elixir: https://hipsters.tech/elixir-a-linguagem-hipster-hipsters-48/
- Principal evento da linguagem no Brasil: https://twitter.com/elixir_brasil

Espero que esse artigo tenha sido útil para você. Caso tenha alguma sugestão e/ou feedback deixe nos comentários, esse é um dos meus primeiros artigos e gostaria de saber como estou me saindo :grin:

Obrigado pelo seu tempo e até a próxima :vulcan_salute:
