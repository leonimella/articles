# Docker e Docker Compose um guia para iniciantes.

Quando comecei a usar _Docker_ e o _Docker Compose_ foi fácil entender quais eram suas funções, mas encontrei dificuldade em descobrir como eu poderia aplicá-los em meus projetos. Depois de muita leitura, tentativa e erro e algumas implementações eu consegui "decifrar" os **pedaços que estavam faltando** e aprendi o básico para utilizá-lo.

Nesse artigo tento explicar alguns desses pedaços que, os inciantes em _Docker_, geralmente se confundem.

## Antes de começarmos
Caso você não conheça nada sobre _Docker_, sugiro que você leia a [documentação oficial](https://docs.docker.com/) antes e se você ainda não tem _Docker_ e _Docker Compose_ instalado, aqui estão alguns links para ajuda-lo:
- Docker
    - [Instalação](https://docs.docker.com/install/)
    - [Instalação em pt-br](https://www.digitalocean.com/community/tutorials/como-instalar-e-utilizar-o-docker-primeiros-passos-pt)
    - [Docker overview](https://docs.docker.com/engine/docker-overview/)
- Docker Compose
    - [Instalação](https://docs.docker.com/compose/install/)
    - [Instalação em pt-br](https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-18-04-pt)
    - [Docker Compose overview](https://docs.docker.com/compose/)

Dividi este artigo por partes que considero importante, sinta-se livre para pular as que você já esteja familiarizado.

---

## <center>Parte 01 - Dockerfile, Imagens e Containers</center>

Geralmente o arquivo `Dockerfile` é o ponto de entrada para quem começa a utilizar o _Docker_. Embora o [guia](https://docs.docker.com/get-started/part2/) explique bem as instruções definidas no arquivo eu não conseguia entender muito bem o que eram as **imagens** e o que eram os **containers**.

O `Dockerfile` define uma **imagem**. E o que é uma imagem? A **imagem** é um conjunto de recursos responsável pela execução de um código ou uma aplicação. Geralmente a imagem que você define sempre tem outra imagem como base. Você pode pensar nela como se fosse um pacote semelhante a um módulo do `node_modules`, ou um `package` dentro do `vendor` da sua aplicação, não é a forma mais correta de definir, mas pode ajudar a enxergar como ela funciona. Esses pacotes de recursos servem executar alguma função dentro da sua aplicação, já as imagens servem para construir o **container** que você vai rodar sua aplicação.

Todo **container** roda em cima de uma **imagem**, é uma dependência direta, se tem um **container** sendo executado tem uma **imagem** por trás dele. Por exemplo, vamos supor que nós queremos executar o comando `elixir -v`. Nós precisamos definir uma **imagem** capaz de interpretar a linguagem Elixir e depois colocar um **container** para executar essa imagem. Vamos criar o `Dockerfile` e inserir nele o mínimo do mínimo para nosso exemplo:
```dockerfile
FROM elixir:1.10
```

Agora vamos construir a imagem que estamos definindo com o comando `build` (mais sobre a `--tag` depois):

```shell
docker build --tag=elixir:example . # o ponto final é importante!
```

Pronto! Temos a imagem criada, você pode conferir ela listando as imagens no seu sistema, executando `docker image ls`

Agora que temos uma imagem definida, vamos criar nosso container e executar o comando que queremos:

```shell
docker run --rm elixir:example elixir -v
```

E pronto! A imagem que criamos foi utilizada nesse container que executou o comando `elixir -v` e foi destruído após essa execução graças a flag `--rm` que passamos.

### "Pegadinhas"

Nosso `Dockerfile` é muito muito simples e nesse caso não precisaríamos ter definido um. O mesmo resultado pode ser obtido ao executar o comando:

```shell
docker run --rm elixir:1.10 elixir -v
```
Agora ao invés rodar um container em cima da imagem que criamos com o `docker build ...`, rodamos o container tendo como base a imagem padrão do Elixir 1.10: `elixir:1.10`. Já que nossa imagem só tem como dependência a `elixir:1.10` cortamos caminho e executamos o container direto com ela.

Mas vamos supor que além da linguagem Elixir instalada, precisássemos também do Git. Ai sim declaramos um `Dockerfile` que reuna as duas dependências:

```dockerfile
FROM elixir:1.10

RUN apt-get update && apt-get install git -y
```

Dessa forma a imagem gerada contém esses dois recursos que seriam necessários pela nossa aplicação.

E pra fechar essa primeira parte, e quanto a essa flag que utilizamos `--tag`? Quando "tageamos" uma imagem estamos versionando ela, você pode fazer um paralelo com o sistema de **branches** do Git. Ao tagear uma imagem nós salvamos tudo o que foi instalado nessa imagem naquela versão. Por exemplo, a imagem `elixir:1.10` significa que estamos utilizando uma imagem com nome Elixir e que a versão dessa **imagem** é a 1.10, **por convenção a versão de Elixir sendo executada dentro dessa imagem também é a 1.10.**

Isso se repete para todas as outras linguagens e recursos: `php:7.3`, `python:3.8`, `ubuntu:18.04` e etc...

---

## <center>Parte 02 - Operando o Container</center>

O **container** é a parte responsável por executar o que foi definido na sua imagem, sendo assim é recomendado evitar qualquer mudança ou edição diretamente. Caso você instale ou modifique alguma coisa no container a alteração feita fica disponível somente no próprio **container** e não na **imagem** base. Dito isso, de vez em quando precisamos debugar, modificar ou experimentar alguma coisa em nossas aplicações.

Vamos cobrir algumas operações e configurações básicas que podem ajudar você a dar aquele ajuste fino na sua aplicação e ver o que está acontecendo dentro dela.

### 2.1 Criando
O comando `docker run` cria um container dada determinada imagem e ele ficará la até que você o remova. Se você usa a forma mais simples do comando, `docker run elixir:1.10` o _Docker_ cria um container com um nome default gerado automaticamente, executa o que está definido na imagem, ou o comando que você passou com o output do comando no terminal.

Podemos melhorar isso passando alguns parâmetros para o comando `docker run`.

Dê um nome ao seu container usando a flag `--name`:
```shell
docker run --name=application elixir:1.10
```

Isso facilita muito no manuseio, se você precisar executar um comando nesse container ou pausar, iniciar e remover ele, basta passar o nome selecionado ao invés do id. Por exemplo:

```shell
docker start application

docker stop application

docker exec application {command}
```

Executando o container em _detached_ mode usando a flag `-d`:
```shell
docker run --name=application -d elixir:1.10
```

Após a inicialização do container ele continua rodando em segundo plano, liberando o seu terminal. Provavelmente você vai usar muito essa flag. Você pode conferir o status atual do seu container com o comando `docker ps` e ver que ele continua lá, rodando.

Exponha uma porta de rede de seu container para que ele possa receber tráfego com a flag `--expose`.
```shell
docker run --name=application --expose 4000 -d elixir:1.10
```

Dessa forma o container deixa a porta `4000` exposta para receber tráfego através dela. Provavelmente você vai expor a porta que sua aplicação está ouvindo. Ou você pode vincular com uma porta no seu `localhost` para que toda vez que você faça um request para essa porta, por exemplo: `http://localhost:4000` esse request seja encaminhado para o seu container.
```shell
docker run --name=application -p 4000:4000 -d elixir:1.10
```

A flag `-p 4000:4000` indica que a porta `4000` da sua máquina será encaminhada para a `4000` do container. Resumindo `-p host_port:container_port`.

**É importante entender a diferença entre o vinculo `-p` e a exposição `--expose` de portas do container**. Quando expomos uma porta do container, ela não está vinculada à nossa máquina, ela apenas está disponível para o request, dessa forma ao invés de fazer o request assim: `http://localhost:4000` fazemos o request assim: `http://{container_ip}:4000`. Demorei um tempo para entender isso!

### 2.2 Gerenciando
Quando um container está rodando é corriqueiro executarmos algumas operações nele durante o desenvolvimento. As vezes precisamos ver os logs da aplicação, modificar algum comportamento, checar alguma outra coisa, enfim... Problemas que aparecem durante o desenvolvimento.

Vamos supor que criamos o container seguindo o exemplo do tópico anterior:
```shell
docker run --expose 4000 -d --name application elixir:1.10
```

Se quisermos ver os logs da aplicação que está sendo executada dentro do container, geralmente basta executar:
```shell
docker logs application
```

Executando esse comando todo o log que existe no seu container será exibido no seu terminal. Você pode aplicar a flag _follow_ `-f`, isso fará com que o terminal fique "escutando" os logs e qualquer logs novos serão exibidos na sua tela.
```shell
docker logs application -f
```

Melhorando um pouco mais, podemos filtrar o output inicial desse comando com a quantidade de linhas que quiser ver. Fazemos isso com a flag `--tail=number`.
```shell
docker logs application -f --tail=10
```

Dessa forma seguimos os logs do container e será exibido somente as últimas 10 linhas.

Outra operação muito comum é o "ssh login" nos containers. Para tal operação utilizamos o seguinte comando:
```shell
docker exec -it application bash
```

Esse comando não faz um "ssh login". Na verdade ele vai modificar o input do seu terminal para que todos os comandos a seguir passem a rodar dentro do programa iniciado dentro do container, o `bash` nesse caso. E também vincula o output do container ao seu terminal. Tudo isso é feito pelas flags `-it`.

Você pode rodar comandos para o seu container sem essas flags. Algumas aplicações escrevem arquivos de logs ao invés de só exibi-los no terminal, para ler esses arquivos você poderia uilizar o `docker exec` da seguinte forma:
```shell
docker exec application tail -f /path/to/log_file
```

Agora que temos um entendimento claro da diferença entre **imagens** e **containers** e de alguns comandos básicos, vamos falar sobre o _Docker Compose_.

---

## <center>Parte 03 - Docker Compose não é Docker</center>
Pode parecer besteira, mas é importante ressaltar aqui que _Docker Compose_ e _Docker_ são coisas diferentes!

**Resumindo, o _Docker Compose_ é uma ferramenta de administração de containers**.

Geralmente, no mundo de aplicações descentralizadas, os famosos _Micro Services_, é comum que seu negócio esteja dividido em aplicações diferentes, as vezes em linguagens de programação diferentes. Por causa dessa separação, teoricamente, você precisaria de mais de uma imagem e mais de um container para que tudo funcione.

Mesmo se sua aplicação for um monolito, a chance de você utilizar um banco de dados, web server, redis e outras tecnologias fora da sua aplicação é grande o que "força" você a dividir cada vez mais em containers e imagem menores que sejam responsáveis somente por uma tarefa.

Então, se você tem uma aplicação monolítica, que se conecta a um banco de dados, o ideal seria ter um container com sua aplicação e outra com seu banco de dados, certo? Com o que vimos até agora você poderia criar esses dois containers rodando o `docker run` com especificações um pouco diferentes.

Mas vamos supor que além da aplicação e do banco de dados você também precise do Nginx. Ai você teria que criar mais um container. Percebe que começa a ficar complicado gerenciar esses containers "soltos" que estamos criando para nossa aplicação funcionar? Por causa dessa crescente complexidade você pode optar pelos seguintes cenários:

### 3.1 Rodar todos os containers de forma independente
Nesse cenário você sairia escrevendo `docker run` no seu terminal como se não houvesse amanhã. Destruindo e modificando containers até que eles funcionem da maneira que você espera.

Funciona, mas essa abordagem não é muito prática, imagina que para subir sua aplicação você tenha que iniciar cinco containers diferentes. Lá vai você `docker run ...`, `docker run ...`, `docker run ...`, `docker run ...` e por último `docker run ...` com flags e imagens diferentes em cada comando... Acho que já deu preguiça em você só de pensar no trabalho desse método.

### 3.2 Agregar todas as tecnologias em uma única imagem
Você poderia criar uma imagem que contenha todas as dependências do seu negócio. Parece interessante, certo?

Mas imagine o tanto de configuração necessária no seu `Dockerfile` para que você rode sua aplicação, seu banco de dados, seu web server e etc. Você acabaria tendo um `Dockerfile` mega extenso e difícil de manter. Seria praticamente fazer o setup disso tudo no seu próprio computador. Bem cansativo também.

Essa abordagem não é de todo o mal para ambientes de desenvolvimento ou homologação, mas imagine em um servidor de produção. Na AWS, por exemplo, você utilizar o seu banco de dados dentro de uma EC2 não parece uma boa ideia. Mas pode ser que você esteja usando _Docker_ só em desenvolvimento, nesse caso pode ser uma abordagem válida.

### 3.3 Criar um docker-compose.yml
Essa provavelmente é a melhor abordagem para gerenciar vários containers dependentes. O arquivo `docker-compose.yml` é onde declaramos nossas instruções e o estado que cada container deve ser criado e operado bem como a comunicação entre eles.

Esse tópico é um pouco extenso, vamos elaborar melhor ele na próxima parte deste artigo.

---

## <center>Parte 04 - Docker Compose</center>
Vimos que o _Docker Compose_ é uma ferramenta para gerenciar containers que tenham algum tipo de dependência entre si. Nós declaramos esses containers e seus parâmetros em um arquivo chamado `docker-compose.yml`.

Vamos supor que em uma empresa o negócio dela esteja dividido em três partes: Uma aplicação que utiliza o Phoenix (um _web framework_ feito com Elixir), um servidor web Nginx e um banco de dados Postgres.

Dado esse cenário, vamos criar um `docker-compose.yml` capaz de definir 3 containers diferentes para que cada tecnologia tenha seu próprio ambiente e sejam capazes de conversar entre si.

### 4.1 Criando o docker-compose.yml
A primeira coisa que fazemos no arquivo é declarar os 3 containers que precisamos:

```yml
version: "3.7"
services:
  app:
    build: .
    command: "mix phx.server"

  nginx:
    image: nginx:1.17

  postgres:
    image: postgres:9.5
```

O bloco `version` mostra pra gente qual versão do `docker-compose` está sendo utilizada e o bloco `services` declara os 3 containers necessários: app, nginx e postgres. Cada `service` é um bloco indenpendente com as próprias declarações.

No service `app` temos duas declarações. A primeira é a `build: .` essa declaração define que para construir esse container deve ser utilizado um `Dockerfile` que deve estar no mesmo diretório que o arquivo `docker-compose.yml`. Por esse service representar o container da nossa aplicação, é comum que tenhamos um `Dockerfile` que declara os recursos que precisamos em nossa imagem para que a aplicação rode corretamente.

Nos demais services, nginx e postgres, nós substituímos a declaração `build` pela declaração `image`. Dessa forma ao invés de procurar outro `Dockerfile` para construir uma imagem do Nginx e outro para construir a imagem do Postgres, nós utilizamos uma imagem pronta, fornecida pelo _Docker Hub_ que já possui essas duas tecnologias prontas para uso.

Você poderia declarar `Dockerfiles` diferentes para cada service, mas em nosso caso não é necessário, já que as imagens `nginx:1-17` e a `postgres:9.5` já tem tudo o que precisamos para as respectivas tecnologias.

E a ultima declaração do `app` é  a `command`. Nela nós colocamos o valor `mix phx.server`, que é um comando do Phoenix responsável por inicializá-lo. Assim, quando o container ficar pronto, esse comando será executado e nossa aplicação é iniciada dentro dele.

Não utilizamos `command` nos outros services pois as imagens que escolhemos já lidam com essa parte da inicialização do serviço dentro delas.

### 4.2 Melhorando os serviços
#### 4.2.1 App
Além dos pedaços que já definimos no tópico anterior temos que nos preocupar com alguns outros aspectos da nossa aplicação e do nosso container. Precisamos expor a porta que nossa aplicação ouve, montar um "volume" que vai ligar os arquivos locais com os arquivos do container e também, por comodidade, definir um _working directory_ para facilitar a execução de alguns comandos da nossa aplicação.

Para que todos os requisitos sejam atendidos, modificamos o bloco `app` da seguinte forma:
```yml
app:
    build: .
    container_name: "app"
    volumes:
      - ./:/var/www/application/
    working_dir: "/var/www/application/"
    expose:
      - "4000"
    command: "mix phx.server"
```

Como vimos anteriormente `expose` expõe a porta `4000` do container para que ela receba tráfego. `working_dir` define que o path que passamos à ela, `/var/www/application/` daqui pra frente seja o diretório padrão onde os comandos serão executados. Então quando o container for executar o `mix phx.server` ou qualquer outro comando passado à ele, ele vai executar no diretório `/var/www/application/`.

O `container_name` é basicamente isso, estamos definindo um nome para nosso container.

E por últimos temos o `volume`, essa declaração faz uma "ligação" com o diretório da sua máquina (host) com o diretório do container. Então, nós estamos "ligando" o diretório em que o `docker-compose.yml` está `./` com o diretório `/var/www/application/` dentro do container. Dessa forma todos os arquivos que tem em um diretório aparecerá no outro.

#### 4.2.2 Nginx e Postgres
Para concluir a definição do nosso `docker-compose.yml` vamos complementar os serviços restantes:

```yml
nginx:
    image: nginx:1.17
    container_name: "nginx"
    volumes:
      - ./nginx:/etc/nginx/conf.d
    ports:t
      - "80:80"
```

Adicionamos o `volume`, que colocará os arquivos de configuração do Nginx dentro do caminho correto do container. E por último adicionamos o `port` que vai vincular o nosso `localhost` ao container do `nginx`. Dessa forma, sempre que fazemos um request `http://localhost`, ele será encaminhado para o container do `nginx` que, por sua vez, encaminhará o request para o container `app`, conforme dita as configurações nos arquivos dentro da pasta`./nginx`.

E por último, completamos a definição do Postgres
```yml
 postgres:
    image: postgres:9.5
    container_name: "postgres"
    volumes:
      - ./postgres:/var/lib/postgres
    environment:
      - POSTGRES_DB=your_db
      - POSTGRES_USER=your_user
      - POSTGRES_PASSWORD=your_secret
    ports:
      - "5432:5432"
```

A única descrição nova aqui é a `environment`. Com ela você adiciona _env variables_ ao seu container. As três variáveis que adicionamos servem para o Postgres configurar um banco de dados, usuário e senha.

Vinculamos as `ports` do Postgres com o host para facilitar a conexão ao container do banco de dados. Dessa forma, se você possui algum programa para gerenciar banco de dados, por exemplo, facilita a conexão com esse container.

### 4.3 Completando o docker-compose.yml
Se você seguiu esse guia até agora, a versão final do seu `docker-compose.yml` será assim:

```yml
version: "3.7"
services:
  app:
    build: .
    container_name: "app"
    volumes:
      - ./:/var/www/application/
    working_dir: "/var/www/application/"
    expose:
      - "4000"
    command: "mix phx.server"

  nginx:
    image: nginx:1.17
    container_name: "nginx"
    volumes:
      - ./nginx:/etc/nginx
    ports:
      - "80:80"

  postgres:
    image: postgres:9.5
    container_name: "postgres"
    volumes:
      - ./postgres:/var/lib/postgres
    environment:
      - POSTGRES_DB=your_db
      - POSTGRES_USER=your_user
      - POSTGRES_PASSWORD=your_secret
    ports:
      - "5432:5432"

```

Agora que temos o nosso arquivo do `docker-compose` configurado, vamos ver como utilizá-lo!

---

## <center>Parte 05 - Usando o Docker Compose</center>

No tópico anterior definimos um `docker-compose.yml` com três containers para nossa aplicação ser executada, porém para que tudo funcione nesse momento, vamos fazer pequenas alterações na declaração do nosso serviço `app`:

```yml
app:
    image: elixir:1.10
    container_name: "app"
    volumes:
        - ./:/var/www/application/
    working_dir: "/var/www/application/"
    expose:
        - "4000"
    tty: true
```


Como não temos uma aplicação Phoenix pronta para rodar o comando `mix phx.server` não vai funcionar e nosso container não conseguirá subir.

Substituímos por um container criado a partir de uma image padrão de Elixir `elixir:1.10` e trocamos a declaração `command` por `tty`. A declaração `tty` faz com que o container fique em execução em segundo plano mesmo se nenhum comando for executado, o que é justamente o que queremos.

Outra modificação que precisamos fazer é remover o `volume` do serviço `nginx`. Como não temos aplicação rodando, não precisamos configurar o Nginx para rotear o tráfego para o container desejado.

Resumindo, o novo `docker-compose.yml` fica assim:

```yml
version: "3.7"
services:
  app:
    image: elixir:1.10
    container_name: "app"
    volumes:
        - ./:/var/www/application/
    working_dir: "/var/www/application/"
    expose:
        - "4000"
    tty: true

  nginx:
    image: nginx:1.17
    container_name: "nginx"
    ports:
      - "80:80"

  postgres:
    image: postgres:9.5
    container_name: "postgres"
    volumes:
      - ./postgres:/var/lib/postgres
    environment:
      - POSTGRES_DB=your_db
      - POSTGRES_USER=your_user
      - POSTGRES_PASSWORD=your_secret
    ports:
      - "5432:5432"

```

Feito isso podemos colocar todos esses containers para rodar com o comando
```shell
docker-compose up -d
```

O _Docker_ vai validar nosso arquivo `yml` e se tudo estiver ok vai começar a procurar as imagens que nossos containers precisam para serem executados. Aqui nós também adicionamos a flag `-d` para que nosso terminal seja liberado após a conclusão desse comando.

A primeira vez que você executa esse comando em um projeto pode demorar um pouco. O _Docker_ precisa preparar todas as imagens que esses containers utilizarão e dependendo da sua internet ou do seu computador essa etapa pode levar alguns minutos.

Feito isso o _Docker_ começa a criar os containers na ordem e inicializá-los. E se tudo correu bem você pode listar os containers criados com `docker ps`. O output desse comando mostra todos os containers que estão em execução, adiciona a flag `-a` para ver todos os containers que existe na sua máquina.

Deixo abaixo uma lista com os comandos mais corriqueiros do _Docker Compose_:

Parando todos os containers:
```shell
docker-compose stop
```

Removendo todos os containers:
```shell
docker-compose rm -f
```

Vendo o log de todos os containers:
```shell
docker-compose logs -f
```

Construir os containers sem inicializá-los:
```shell
docker-compose build
```

Vale ressaltar que os containers criados pelo comando `docker-compose` são "comuns", sendo assim os outros comandos de _Docker_ também são válidos. Você pode até parar e retomar a execução de um container em específico.

---

## <center>Parte 06 - Interação entre Containers</center>
Teoricamente, todos os containers que criamos no tópico anterior fazem parte da mesma aplicação e por isso precisam se comunicar. Quando criamos containers utilizando o `docker-compose`, além dos containers, outras entidades são criadas. Uma das mais importantes é a **_Network_**.

Na versão `2.x` todos os containers de todos os arquivos `docker-compose.yml` que eram criados se juntavam em uma mesma `network` com nome `default`. Na versão `3.x` esse comportamento mudou, de modo que cada arquivo `docker-compose.yml` tem sua própria `network`. O `docker-compose` nomeia a `network` do arquivo `docker-compose.yml` utilizando o nome do diretório concatenado com a string `_default`.

O tópico de `network` do _Docker_ é muito extenso para esse artigo, então vou deixar ele para um post dedicado. O que você precisa saber aqui é que na versão `3.x` do `docker-compose` a `network` é definida por arquivos `docker-compose.yml` e um container de fora desse arquivo, por padrão, não se comunica com os que estão definidos nele. É possível conectá-lo à `network`, mas não está no escopo deste artigo.

Enfim, todos os nossos containers fazem parte da mesma `network` e por isso a comunicação entre eles é muito fácil. O _Docker_ facilita nossa vida fornecendo um "DNS" para os containers da mesma rede. Vamos utilizar o container do Postgres como exemplo. Para a nossa aplicação usar o container do postgres, tudo o que precisamos fazer é colocar o **nome do container no host do nosso banco de dados na aplicação**. Simples assim!

Não se esqueça que outros parâmetros de rede também são importantes. Certifique-se de que a porta correta esteja exposta (`expose`) no container desejado e que o request está sendo enviado para aquela porta!

---

## <center>Conclusão</center>

Espero ter ajudado você na compreensão dessa ótima ferramenta de desenvolvimento! Lidar com o _Docker_ pode ser complicado no começo, mas é uma excelente ferramenta para cuidar de um pedaço muito importante da aplicação que geralmente deixamos passar: **A Infraestrutura!**

Eu acho que o _Docker_ lida com esse lado de nosso trabalho de forma primorosa, possibilitando e facilitando muito do nosso dia-a-dia! Por isso, é uma ferramenta que vale muito a pena!

Obrigado pelo seu tempo e até a próxima 🖖
