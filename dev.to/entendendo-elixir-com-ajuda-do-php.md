Entendendo Elixir com ajuda de PHP

Depois de alguns anos e diversas experiências em PHP utilizando WordPress, Laravel, Symfony e Phalcon, tive a oportunidade de trabalhar com Elixir e desde então ela tem sido a minha linguagem principal de desenvolvimento para novos projetos

Para quem nunca teve o contato com uma linguagem funcional pode ser um pouco difícil entender como Elixir funciona, foi o meu caso quando comecei a utilizá-la.

A “ajuda” oferecida por PHP, nesse artigo, não passa de uma comparação feita entre as duas linguagens que por sinal não possuem NADA semelhante.

O que fiz foi implementar a mesma funcionalidade com linguagens diferentes para que você possa comparar a LÓGICA de uma linguagem mais familiar com uma que talvez não seja.

Vou abordar as funcionalidades do Elixir que mais utilizo no dia-a-dia, meio que aplicando a regra 80/20, mas tenho certeza que com esse conhecimento você poderá ao menos ler um arquivo `.ex` ou `.exs` e não se perder completamente, caso você seja novo na linguagem.


First things first!

Para melhor entendimento do artigo é interessante que você pelo menos conheça o básico da linguagem. Caso você ainda não esteja familiarizado com esses assuntos, sugiro a leitura do Starting Guide na documentação oficial do Elixir

Sem mais demoras, vamos ao código!


Arity

Em Elixir uma função não é definida apenas pelo seu nome, mas também pela sua arity ou aridade que nada mais é do que o número de parâmetros que uma função recebe.

[IMAGEM EXEMPLIFICANDO ARIDADE]


No exemplo acima podemos ver que temos 3 declarações para a função de nome hello: `hello/0` que tem aridade 0, `hello/1` que tem aridade 1 e `hello/2` que tem aridade 2.

Isso significa que podemos chamar a função `hello()` de 3 maneiras diferentes e cada uma delas invocará apenas a função com o número de parâmetros correspondente

[IMAGEM INVOCANDO FUNÇÃO HELLO() COM ARIDADE DIFERENTES]


Talvez fique um pouco difícil entender porque esse conceito de aridade é extremamente útil. Ele é base de diversos outros que combinados tornam o desenvolvimento muito mais fluído removendo complexidade das funções.

No próximo tópico o entendimento desse conceito ficará mais claro, vamos seguir?


Pattern Matching

Vamos falar sobre o match operator: `=`. Além de você utilizá-lo para atribuir valores à variáveis, você também pode utilizá-lo para decompor estruturas mais complexas como tuplas.

[IMAGEM UTILIZANDO = PARA CRIAR VARIÁVEIS ATRAVÉS DE TUPLAS]

Ou obter o primeiro valor de uma lista

	[IMAGEM UTILIZANDO = PEGANDO O HEAD DE UMA LISTA]

Caso uma tupla não possa ser equiparada com o lado esquerdo do operador, isso é, caso a tupla tenha um número de campos diferente ou algum valor que não condiz com o lado esquerdo do `=` uma exception acontecerá:

[IMAGE COM MATCHERROR DA TUPLA]

O mesmo acontece com uma lista

[IMAGE COM MATCHERROR DA LISTA]

Porém pattern matching não se limita somente à atribuição de variáveis, podemos explorar esse conceito em parâmetros de funções.

Para ilustrar melhor, vamos criar um módulo chamado `GreetUser` e dentro dele uma função chamada `hello/1`. Para ajudar no entendimento dessa vez vou criar uma classe semelhante em PHP.



Classe PHP
[IMAGEM GREETUSER PHP]

Módulo Elixir
[IMAGEM GREETUSER ELIXIR]


Até aqui, sem novidades, certo? Se chamarmos a função `hello/1` ela vai exibir na tela a mensagem “Hello, Leoni!”.

Mas vamos supor que nem sempre eu teria o nome do meu usuário disponível. Em determinadas vezes a variável `user_name` poderia ser `nil` (`null` em PHP). Como podemos ajustar nosso código para esse requisito?

Classe PHP
[IMAGEM GREETUSER PHP com default null]

Em PHP poderíamos adicionar um valor default para o parâmetro `$userName` e tratar esse caso com um `if` dentro da nossa função exibindo uma mensagem diferente caso o argumento seja `null`.

Embora a mesma solução seja possível, em Elixir podemos utilizar o function pattern matching ao nosso favor para simplificar o nosso código sem a necessidade do `if`.

Módulo Elixir
[IMAGEM GREETUSER ELIXIR com pattern matching nil]

Como vimos no tópico anterior, o nome não é o único parâmetro para definir uma função, embora a aridade da função seja a mesma nos dois casos, `hello/1`, o parâmetro esperado na primeira é explicitamente `nil`.

Isso quer dizer que somente quando `user_name = nil` é que a primeira função será executada, caso contrário ele “pula” essa função e vai para a próxima `hello/1` declarada neste módulo.

Qualquer tipo primitivo da linguagem pode ser utilizado em Pattern Matching. Se o módulo `GreetUser` agora tem que enviar uma mensagem específica para usuários que tenham o nome “João”, poderíamos fazer da seguinte forma

Classe em PHP

Módulo em Elixir

Aqui já é possível enxergar um pouco melhor como esse conceito reduz a complexidade em nosso código. Agora que você entende pattern matching não fica melhor ler a função em Elixir do que em PHP? Não precisamos mais ficar “calculando” `if`s para saber qual o resultado esperado, uma bela vantagem, não?


Guards

Se você gostou do conceito de pattern matching você pode ir além utilizando guards. Eles permitem verificações mais completas dos parâmetros que sua função está recebendo.

Guards são definidos com um `when` após a declaração da função:

[GUARD SIMPLES]

Enquanto o pattern matching é útil para valores explícitos, com guards podemos criar diversas condições para serem checadas. Podemos checar o tipo de um parâmetro, o tamanho dele se ele é ou não `nil`, se ele é maior ou menor que determinado valor.

Voltando ao nosso exemplo `GreetUser`, agora temos a necessidade de checar se o parâmetro `user_name` é ou não uma `string`. No PHP e no Elixir poderíamos “tipar” o parâmetro obrigando que ele seja de um determinado type. Não utilizaremos esse recurso só para dar entendimento à esse exemplo.

Classe PHP
[VALIDAÇ O DO TIPO DE PARAMETRO]

Módulo Elixir
[GUARD APLICADO EM UMA FUNÇ O]

Podemos ver mais uma vez a simplicidade da nossa função em Elixir se comparada a mesma função em PHP. Sem estruturas de controle no mesmo código, gerando melhor legibilidade.

A função utilizada no guard `is_binary()` retorna um booleano dependendo do valor passado a ela e com esse resultado (`true` ou `false`) a função é invocada ou não. Elixir possui diversas funções “type-checks” como essa que usamos para checar tipos.

Lembrando, que tanto em um guard e/ou um pattern matching você sempre precisa pensar em uma alternativa para o caso em que um parâmetro não se enquadre em nenhuma validação caso contrário um RunTimeError será lançado

[RunTimeError de matching inválido]

O Erro acima aconteceu porque a chamada da nossa função `hello/1` não é compatível com nenhuma de suas declarações.

validaçõesPara corrigir podemos declarar nossa `hello/1` com o parâmetro `_user_name`, que aceitará qualquer coisa e qualquer valor, no final do nosso módulo com uma mensagem mais amigável para nosso usuário entender o que houve durante a execução

[RunTimeError corrigido]

Esse é mais um conceito do arsenal de Elixir para que nosso código fique mais legível e menos complexo. Apesar de flexível guards possuem validações limitadas propositalmente pela linguagem. Lembrando que o link para esse e outros tópicos estão disponíveis no final deste artigo.

Pipe Operator

Esse pode ser um pouco mais complicado para entender, mas em resumo o pipe operator é representado por esse símbolo `|>` e o trabalho dele é passar o resultado de uma expressão como primeiro parâmetro de outra expressão.

Com isso você pode criar pipelines (sacou o nome?) de execução dada determinada expressão (variável ou função).

Para exemplificar, agora nosso módulo `GreetUser` deve ser capaz de cumprimentar diversas pessoas cujo os nomes serão enviados em forma de uma lista. Para fazer isso vamos criar uma função `hello_group/1` que recebe como argumento `users_name` que nada mais é do que uma lista contendo o nome dos usuários em string.

Além disso precisamos também tratar os nomes dos usuários para que eles fiquem capitalized, isso é, somente a primeira letra do seu nome em maiúsculo.

[DEFINIÇ O DA HELLO_GROUP]

Aproveitamos para adicionar um guard na nossa função para nos certificar de que o parâmetro é mesmo uma lista e também definimos uma `hello_group/1` default para ser utilizada caso `users_name` não seja uma lista.

O próximo passo é manipular a lista aplicando os requisitos acima utilizando o pipe operator. Como primeiro passo, vamos percorrer a lista e normalizar os nomes. Faremos isso com a ajuda do módulo `Enum` que é um módulo nativo do Elixir para trabalhar com elementos enumeráveis, o que é o caso da nossa lista de nomes

[ADICIONADO O PRIMEIRO PIPE NA HELLO_GROUP]

	
Vamos quebrar o código acima em pontos chaves:
Invocamos `users_name` dentro da nossa função
Invocamos `|>` (pipe operator) abaixo de `users_name` poderia ser do lado direito também.
Invocamos `map/2` do módulo `Enum` para percorrer a lista de nomes e nos retornar uma lista
Passamos como segundo parâmetro de `map/2` uma função anônima `fn users_name -> end` que será executada para cada nome da lista
Dentro da função anônima utilizamos a função `capitalize/2` do módulo `String` também nativo do Elixir para capitalizar os nomes.

O que precisa ser entendido desse código é que só passamos o segundo argumento para a função `map/2` porque o primeiro argumento que deve ser uma lista (no nosso caso a lista com os nomes dos usuários) já foi passada à essa função pelo `|>`.

Lembre que: “O pipe operator é responsável por passar o valor de uma expressão como o primeiro parâmetro da próxima expressão” No caso nossa primeira expressão é a variável `users_name` que ao ser processada retorna nossa lista de nomes e uma lista é justamente o que `map/2` recebe no primeiro parâmetro, por isso nossa pipeline funciona!

E o segundo parâmetro passado ao `map/2` foi uma função anônima (requerida por `map/2`) que recebe como argumento o elemento da lista que está sendo percorrida para que possamos manipulá-lo.

Em PHP podemos obter o mesmo resultado até aqui dessa forma:

[HELLO_GROUP em PHP]


O próximo passo nessa função é juntarmos todos os nomes em uma única string. Para isso utilizaremos outra função nativa do Elixir, a `join/2` do módulo `Enum`. Ela recebe como primeiro parâmetro uma lista e como segundo parâmetro um elemento de junção, no nosso caso utilizaremos uma vírgula seguida de espaço em branco, para separar os nomes dentro da string.

Nossa pipeline continua funcionando porque `map/2`, após ser executada, retorna uma lista e o `|>` passa essa lista como primeiro parâmetro para a próxima função `join/2`.

Ótimo! `hello_group/1` já consegue normalizar e juntar os nomes enviados como lista em uma única string! Agora só nos resta imprimir nossa mensagem para o usuário.

Até agora o retorno de `hello_group/1` seria uma string contendo os n nomes de usuários que foi passado à ela. Huum… Se `join/2` retorna uma string e nós temos a função `hello/1` que recebe como primeiro parâmetro uma string, podemos utilizá-la para exibir a mensagem padrão dos outros tópicos!

Então, para concluir o tópico pipelines com o pipe operator, adicionamos `hello/1` no final de nossa pipeline concluindo os requisitos de exigidos para `hello_group/1`.

Veja como ficaria o mesmo código em PHP

Sei que poderíamos quebrar esse código em diferentes função de forma para organizar melhor o código, mas repare que mesmo dessa forma em Elixir temos um código MUITO mais limpo e legível.

Uma observação muito importante! Em linguagens funcionais nós não temos a opção utilizar `return`s dentro das funções. As funções retornarão o resultado da última expressão executada, vamos explorar mais sobre isso no próximo tópico, mas fique com essa informação na cabeça pois ela é muito importante!


Estruturas de Controle

Vimos que em Elixir é possível simplificar muito nossas funções removendo condicionais através do pattern matching e guards. Mas as vezes não temos outra alternativa a não ser utilizar um `if` ou `switch.

As estruturas de controles presentes em Elixir são: `if`, `unless`, `case`, `cond` e `with`.

Como esse é um overview da linguagem, não vou me aprofundar em todas elas com exceção do  `with`, talvez ele cause mais estranheza enquanto as outras são mais familiares.

É importante notar que essas estruturas de controles se comportam como funções e por isso alguns hábitos que temos em outras linguagens devem ser evitados em Elixir. Um exemplo disso é quando você define um valor default para uma variável e dentro de um `if` você troca esse valor. Algo parecido com isso:

[Código PHP if valor default]

Esse código não funciona do jeito que você espera em Elixir, como `if` se comporta como uma função ela vai retornar o resultado que foi executado dentro do seu body. Sendo assim o valor que você atribuiu dentro dele para a sua variável não será aplicado pois não faz parte do escopo da função.

Para que isso funcionasse você teria que fazer algo desse gênero:

[Código ELIXIR if valor default]

Dessa forma o retorno de `if` seria atribuído à nossa variável.

Dito isso, vamos ver como o with funciona.

Você pode pensar nele como uma pipeline que checa o resultado de cada expressão e caso o resultado dessa expressão não foi o que você havia descrito ela não executa o código dentro dela. Ou você também pode validar o erro gerado pela expressão para tratá-lo de uma forma mais coerente ou executar uma função diferente.

Para ilustrar melhor, vamos adicionar essa estrutura na nossa função `hello/1`. O que vamos checar é se a execução da função `puts/2` do módulo `IO` foi bem sucedida, em uma aplicação do mundo real você provavelmente não precisaria fazer essa checagem de `puts/2` estamos colocando como exemplo só para não fugir do tema.

De acordo com a documentação `puts/2` retorna um atom `:ok` após a execução bem sucedida. Sabendo disso construímos nosso `with`:

[WITH checando puts/2]

No código acima, `another_function/0` só seria executada caso `puts/2` retornar `:ok`. A única validação que fazemos foi se `puts/2` retorna `:ok`. Se não retornar `:ok` a execução vai quebrar, consegue imaginar qual erro? Sim, um pattern matching error uma vez que `puts/2` não retornou o que era esperado do lado esquerdo do operador `<-`.

Podemos adicionar mais uma cláusula ao `with` para equiparar com o erro devolvido, ou podemos só desprezar o valor retornado por `puts/2` caso o retorno não tenha sido `:ok`.

[WITH com cláusula default]

Podemos também atribuir variáveis do lado esquerdo do `<-` para que sejam utilizadas em caso de um pattern matching bem sucedido. Uma coisa bem comum em Elixir são funções que retornam uma tupla de tamanho 2, na primeira posição um atom indicando se a função foi executada com sucesso `:ok` ou um `:error` em caso de erro e na segunda posição o resultado da função ou o motivo da falha:

```elixir
{:ok, result}
# Or
{:error, reason}
```

Com isso é possível além de verificar o retorno da função já atribuir uma variável para ser utilizada por outra função

```elixir
# With com pattern matching em tuplas
```

No código acima checamos se `function/x` retorna `{:ok, result}` e se retornar utilizamos `result` como parâmetro de `another_function/1`. Caso um erro ocorra e `function/x` retorna `{:error, reason}` nós só exibimos o erro.

Agora que já temos um bom entendimento sobre o with a única coisa que falta é a possibilidade de criarmos um pipeline de funções que cujo retornos são tratados pelo with


```elixir
# With com uma pequena pipeline de funções
```

O código acima cria uma pipeline que checa passo a passo o retorno das funções possibilitando o tratamento personalizado para cada erro que possa acontecer.


And there you have it!

Embora Elixir seja um mundo de funcionalidades e paradigmas diferentes se você possuir um bom entendimento dos tópicos acima você conseguirá se localizar muito bem no código escrito nessa linguagem.

Abaixo estão alguns tópicos importantes para você que deseja continuar investindo na linguagem:

Mix - Task runner
Phoenix - Web framework
Ecto - ORM
Hex - Package manager


Referências

Abaixo uma lista de sites, vídeos e outros materiais:

Site oficial da linguagem: https://elixir-lang.org/
Para aprender mais: https://elixirschool.com/pt/
Mini documentário da origem da linguagem: https://www.youtube.com/watch?v=lxYFOM3UJzo
Hipster ponto tech com o José Valim sobre Elixir: https://hipsters.tech/elixir-a-linguagem-hipster-hipsters-48/
Principal evento da linguagem no Brasil: https://twitter.com/elixir_brasil

Espero que esse artigo tenha sido útil para você e se você tiver alguma sugestão e/ou feedback deixe nos comentários abaixo.

Obrigado pelo seu tempo e até a próxima!