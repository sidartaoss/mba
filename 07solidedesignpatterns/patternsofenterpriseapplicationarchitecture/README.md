# Patterns of Enterprise Application Architecture

## Layering (camadas)

## Separação de responsabilidades

Quando a gente está falando em layering, a gente está falando em separação de camadas, a gente está falando, no final do dia, em separação de responsabilidades.

A gente está pensando em uma forma de abstrair uma parte do sistema em relação a outra parte do sistema para que o sistema em si não tenha apenas uma única responsabilidade e todo o código misturado.

Se eu conseguir separar responsabilidades do meu sistema em partes que façam sentido, ou seja, camadas, vai ficar muito mais fácil eu conseguir olhar para ele e ver responsabilidades.

Mas, para isso, eu tenho que conseguir pensar em abstração. Por quê? Porque na hora em que uma chamada bate no meu sistema e essa chamada vai chamar uma camada, eu tenho que saber, naquele momento que, eu chamando aquela camada, não interessa o que está por trás dela.

## Layer versus tier

Camadas não são pastas.

Pasta é uma separação física.

Camada é uma separação lógica.

Toda vez que a gente está em separação de camadas ou layers, a gente está querendo dizer que é uma separação lógica. Então, eu tenho a minha aplicação, e essa minha aplicação tem pedaços lógicos, que fazem coisas diferentes; com responsabilidades diferentes.

Então, quando eu estou trabalhando com a parte lógica, eu posso ter uma parte que vai acessar banco de dados, eu posso ter uma parte que vai trabalhar com apresentação; eu tenho uma parte que vai cuidar da lógica de negócios.

Então, se você começa a olhar dessa forma, você começa a entender que a sua aplicação é dividida, fatiada por responsabilidades, onde uma camada chama a outra e a camada que está chamando não tem que entender o que está acontecendo por trás da outra, porque a única coisa que ela precisa entender é de abstração.

Ela sabe que, se ela chamar aquele método, ela vai receber um dado do banco de dados, e não interessa qual é o banco de dados, como aquilo foi chamado, qual é a query que foi executada, ela simplesmente chama e acabou.

Conceito de Tier: A Tier é a separação física. O que que isso significa? É aonde a responsabilidade é colocada. Então, a separação lógica é como que as coisas interagem. A separação física é aonde essas coisas vão interagir.

Tier: Significa que você tem pedaços físicos da sua aplicação em lugares diferentes, sejam eles em nodes (i.e., servidores) diferentes, sejam eles em pastas diferentes dentro da sua aplicação.

## 3 Layers architecture (arquitetura em 3 camadas)

- Presentation
	- Display information;
    - Retorno da informação em formatos diferentes;
	- Quando você está retornando a API REST, você está na camada de apresentação;
	- CLI - (Formato de apresentação CLI), AWS CLI, Google CLI, etc.
	- HTTP Requests - REST, formato de apresentação: todo HTTP request vai retornar em um formato específico, JSON, XML, etc.;
	- Apresentação não possui lógica, ela retorna dados, é uma camada burra, ela entrega a informação da maneira que ela é.

- Domain
	- Coração da aplicação
	- É a camada onde nós trazemos a resolução do problema que está sendo proposto;
	- Regras de negócio; é a camada que resolve o problema.

- Data Source
	- Banco de dados;
	- Camada em que vai ter acesso, onde recupera ou persiste dados;
	- Mensageria - também traz informações, ou recebe informações também, então, também faz parte da camada de Data Source.

## Regra de ouro

``
Domínio e Data Source nunca podem depender da apresentação (dados provindos da API REST, mensageria, etc.). 
``

Se o coração começar a depender das bordas da aplicação, significa que mudar a camada de apresentação vai fazer com que o coração da aplicação mude. Aí, temos um problema muito grande; isso vai gerar desestabilização no core da aplicação.

# Domain Logic Architectural Patterns

## Transaction Script

Regras de negócio em torno de transações (para realizar um processo dentro do sistema).

Segue formato mais procedural.

Realizar um conjunto de transações, de forma seqüencial.

Exemplo: 

```
function processarPedido()
    Verificar estoque;
    Aplicar promoções;
    Aplicar cartão fidelidade do cliente;
    Criar pedido no banco de dados;
    Etc.
```

 - Vantagens:
	- Orientado a transações;
    - Totalmente direto ao ponto;
    - Adequado para requisitos simples; quando temos um sistema que é mais simples.

- Desvantagens:
    - Alta complexidade quando o sistema cresce; muitas transações que vão acontecer no sistema, muitas vezes, partes dessas transações querem conversar com outras, mas elas não se falam, porque elas resolvem um problema do começo ao fim;
    - Trabalha normalmente de forma síncrona; tem que partir que tudo deu certo, para aquela transação acabar; se o usuário vai realizar o processamento de um pedido, o usuário vai ter que aguardar o processamento de todo o processo;
    - Quando tem transações parecidas, acaba duplicando código; é muito comum duplicação de código com Transaction Script, porque cada transação faz uma coisa e, se elas fazem coisas parecidas, você duplica, porque elas são independentes.

## Domain model

Domínio de uma aplicação em objetos de domínio; a lógica do negócio é separada em objetos e, normalmente, tem uma aplicabilidade muito grande, principalmente, quando se fala em orientação a objetos.

Os objetos encapsulam a lógica de negócios; você modela as suas regras baseado em objetos que vão se conversando e que vão gerando o resultado final.

Regras de negócio em primeiro lugar; não está pensando em transação, em banco de dados, em apresentação, está simplesmente pensando em resolver o problema de negócio.

- Validações:
    - Ele se valida o tempo inteiro; mantém a invariância do estado do sistema; garante um sistema com estado coerente.
    - Exemplo: se tem um pedido com diversos itens, o total do pedido vai bater de todos os itens, não vai ter a chance de a soma dos itens mudar o total do pedido; por quê? Porque houve regras de validação que impedem esse tipo de inconsistência; Não existe incoerência dentro do estado da aplicação.

- Vantagens:
    - Clareza e expressividade;
    - Flexibilidade: conforme sistema vai evoluindo, vai estendendo o modelo (SOLID Open Closed Principle), sem quebrar o que já foi feito; sistema grita por padrões de projeto para estender novas funcionalidades (Padrões Decorator, Strategy);
    - Foco na evolução;
    - Alta testabilidade, principalmente via teste de unidade, o comportamento da aplicação está muito correto, coeso, por quê? Porque consegue testar objetos separados, em conjunto e verificar o comportamento da aplicação.
    - Em comparação com Transaction Script, a testabilidade é muito ruim, porque a única coisa que você pode garantir é uma entrada e uma saída, mas, no meio do caminho, não está muito claro o que pode acontecer. Quando se trabalha com Domain Model, pode-se testar cada pedacinho e, depois, fazer um teste geral e tudo deve bater.
- Desvantagens:
    - Complexidade inicial, porque fazer a modelagem não é trivial;
    - Curva de aprendizado é maior;
    - Escalabilidade e performance: é algo mais complexo. Dependendo do tipo de aplicação, às vezes, o modelo de domínio tem que ser "suavizado", para poder se ganhar um pouco mais em performance.
    - Exemplo: Um sistema que exige uma alocação de dados na memória muito pequena.
        - Então, às vezes, quando você precisa de sistemas que precisam de muita velocidade, muitas vezes, você não vai mapear o domínio e não vai fazer essa modelagem, ou você vai facilitar um pouco essa modelagem para, simplesmente, realizar uma transação do início ao fim, ganhando em performance, mas, também, assumindo o risco de o comportamento não estar tão coeso.

## Table module

Sistema organizado e orientado por tabelas do banco de dados.

Regras de negócio segregadas por tabela. Regras de negócio da aplicação de acordo com as regras que vai utilizar para persistir os dados que vai trabalhar.

- Vantagens
    - Alta coesão;
    - Simples; é muito simples trabalhar;
    - Fácil mapeamento;
    - Simples manutenção em diversas situações;
    - CRUD; Orientado a CRUD. Muitos ERP's são orientados a CRUD. Muitas ferramentas low-code/no-code são orientadas a CRUD.

- Desvantagens
    - Acoplamento forte com o banco de dados;
    - Duplicação de código. Diversos modelos fazendo exatamente a mesma coisa;
    - Baixa reutilização. Não consegue reaproveitar uma classe, porque ela está extremamente acoplada a uma tabela do banco de dados;
    - Baixa regra de domínio. Não tem regras de negócio. Se precisar, vai aumentar muito a complexidade.

## Service Layer

Camada de Serviços.

Camada intermediária entre a camada de apresentação e o acesso a dados.

Expõe funcionalidades de alto nível para os "clients".

Encapsula a lógica de negócios.

Quando se trabalha com Domain model, a lógica de negócios fica no modelo de domínio; quando está trabalhando com Service Layer, a lógica de negócios não fica no modelo de domínio, fica na própria Service Layer.

Se está trabalhando com Domain Model, não precisa, necessariamente, de alguém que fique trabalhando com as suas regras, porque as regras estão expressas dentro das próprias entidades; se está trabalhando na Service Layer, quando tiver as entidades, as entidades tendem a ser anêmicas, porque as regras de negócio não estão mais nas entidades, elas estão na Service Layer.

Orquestra a ordem das operações.

Tem acesso à camada de dados.

Gerencia transações, de forma atômica.

- Vantagens
    - Separação de responsabilidades;
    - Reutilização;
    - Melhor estabilidade do que Transaction Script. Se muda a Transaction Script, muda o comportamento de uma transação inteira. No caso da Service Layer, se você mudar um método, o restante das coisas continua funcionando.

- Desvantagens
    - Complexidade ao longo do tempo;
    - Serviços tendem a ficar maiores;
    - Maior acoplamento. Gera dependência entre os serviços. Dependência cíclica;
    - Complexidade no trabalho em equipes. Uma equipe mexe no comportamento de um serviço e quebra outro serviço em outra parte do sistema. 
    - Nesse caso, trabalhar com modelagem de domínio, ao longo do tempo, se prova ser mais eficiente, porque você vai estendendo o comportamento do sistema, mas mantendo a coesão e a forma original de como ele deveria trabalhar.
    - Mas modelar domínio, no início, é mais complexo: DDD Modelagem Estratégica. É uma mudança de paradigma que se tem que ter;
    - Tende a trabalhar com modelos de domínio anêmicos. Tirar responsabilidade das entidades e jogar tudo dentro das classes de serviço.


## Service Layer versus Transaction Script

Apesar de terem idéias semelhantes, possuem estruturas diferentes.

Transaction Script, normalmente, trabalham no formato de funções ou conjunto de funções específicas para cada operação, o que tende a gerar muita duplicação de código. Tende a ser muito procedural, fazer uma coisa atrás da outra, se quiser fazer algo semelhante, você duplica tudo.

Service Layer tende a ser mais flexível e reutilizável. Tem a flexibilidade de trabalhar com orientação a objetos, mesmo trabalhando de uma forma mais procedural, você consegue separar um pouco mais as responsabilidades, consegue ter um pouco mais de reuso.

Ambas as abordagens tendem a deixar o modelo de domínio de lado e são muito mais orientadas a casos de uso; resolver um problema do começo ao fim. Se tiver que fazer manutenção em algum ponto, vai ter implicações em vários pontos do sistema e acaba quebrando muito princípios do SOLID, como Open Closed Principle.

# Data Source Architectural Patterns

## Gateways

``
Objeto (Abstração) que encapsula e acessa sistemas ou recursos externos.
``

Temos duas formas de acesso principal à utilização das Gateways:

- Row Data Gateway:
    - Uma instância para cada linha retornada por uma consulta (uma linha por objeto).

- Table Data Gateway:
    - Estrutura genérica de tabelas que imitam a natureza tabular de um banco de dados - Record Set (uma classe por objeto de tabela). A idéia é simular como se tivesse a tabela inteira do banco de dados em memória dentro dos objetos.

## Table Module (Domain Logic)

Toda vez que a gente for utilizar o conceito de modelagem de domínio baseado em tabelas, provavelmente, vai fazer mais sentido utilizar Row Data Gateway ou Table Data Gateway.

Table Module: quando começa a especificação do sistema a partir da especificação do banco de dados; A regra de domínio não está baseada no caso de uso da empresa, mas na estrutura de dados.

## Domain Model (Domain Logic)

Conseguir modelar o domínio de forma mais rica, onde o comportamento da aplicação vai se dar baseado na relação entre os objetos. 

O ponto de partida que você vai ter na hora de programar, na hora de criar lógica do seu domínio, vai ser muito mais pensando na relação entre os objetos, comportamentos e estruturas desses objetos.

Normalmente, quando você trabalha com Domain Model, você vai perceber que, conforme o sistema cresce, mais o Domain Model se afasta da estrutura de banco de dados.

Em sistemas mais simples, onde o domínio pode ser 1:1 com o banco de dados, talvez seja mais recomendável trabalhar nos formatos de Row Data Gateway, Table Data Gateway. 

Mas, você vai perceber que, conforme o sistema crescer, e você estiver fazendo uma modelagem rica de domínio, você vai ter um descolamento muito claro do banco de dados e do modelo de domínio. 

O que muitas pessoas acabam confundindo é que elas começam a modelar o domínio, começam a trabalhar com Row Data Gateway ou Table Data Gateway e, conforme o sistema fica maior, você acaba entrando em uma complicação, porque o banco de dados não acompanha a modelagem. 

Então, pense bem no formato de abstração com o banco de dados que você vai utilizar.

## Active Record

Um objeto que encapsula uma linha, tabela ou view, e ainda adiciona lógica de domínio em seus dados.

Simples de utilizar.

Acoplamento muito grande com a forma de trabalhar com banco de dados. Entidades vão herdar uma classe base, que possui toda lógica para falar com banco de dados.

Carrega dados e comportamento.

Lógica de domínio tende a "vazar" para o comportamento do banco de dados.

Extremamente simples de usar. Dependendo do projeto, se você quiser ter uma velocidade muito grande para desenvolver, tende a ser uma boa opção.

Ruby on Rails, Laravel (PHP), Django (Python), utilizam Active Record.

Existe como utilizar Active Record, mas não ter vazamento da lógica de negócios dentro do banco de dados:

- Separar o Active Record e toda a sua lógica, pensando apenas na camada de dados, e você ter uma modelagem de domínio totalmente à parte do seu banco de dados e, quando você precisar falar com o Active Record, você tem uma camada de abstração.

Recomendação pessoal:

- Separe o modelo de domínio do modelo do Active Record;

- Enquanto Table Data Gateway ou Row Data Gateway apenas é acoplado no banco de dados, o Active Record tende também a ficar acoplado ao domínio.

Active Record - Quando não usar:

- Domínios complexos. Por exemplo, sistema financeiro; vai ter muito cálculo, muita regra de negócio, diversas métricas, diversas estratégias de persistência.

- Nesse caso, o domínio tende a ter um descolamento muito grande do banco de dados e, no meio desse descolamento, você vai ter que ficar fazendo muita interface para fazer as camadas se falarem e vai gerar um nível de complexidade muito grande;

- Para domínios simples, vai dar um boost de produtividade, para domínios complexos, vai dar um boost de trabalho para trabalhar com persistência de dados.

Active Record precisa de um match exato com as tabelas do banco de dados; é necessário espelhar o banco de dados na aplicação.

Conforme a complexidade do domínio aumenta, os objetos de domínio deixam de ser 1:1 com o banco de dados.

## Data Mapper

Uma camada de mapeamento move dados entre os objetos e o banco de dados enquanto os mantêm independentes um do outro; eu tenho o meu domínio complexo mapeado, aí, no meio do caminho, antes do banco de dados, eu crio uma interface, uma abstração que vai fazer com que eu fale com o banco de dados.

Quando deve utilizar Data Mapper?

- Ideal para domínios complexos;

- Domínio complexo cresce 100% independente do banco de dados; a abstração que vai fazer o seu domínio complexo conversar com o banco de dados é a abstração do Data Mapper;

Domínio não fica refém da estrutura do banco de dados.

Separe as entidades (objetos de mapeamento) do modelo de domínio. O ORM, por padrão, faz o mapeamento de entidade com banco de dados - que é o Data Mapper.

Colocar lógica de domínio na entidade que faz o mapeamento objeto relacional é o pior dos mundos: porque você tem a complexidade do data mapper e não tem os recursos do Active Record. 

Então, sempre a dica é: quando for utilizar Data Mapper, crie a modelagem de domínio e crie a entidade do Data Mapper; nunca misture a sua entidade de domínio com a sua entidade do Data Mapper.

Utilizar as mesmas entidades fará com que seu domínio fique anêmico. Muita gente que faz essa mistura, de utilizar entidades de domínio e entidade como Data Mappers, para evitar que a lógica do domínio vaze para o banco de dados, acaba não colocando lógica nenhuma dentro da entidade e isso significa que começamos a ter entidades anêmicas.

Significa que você vai ter uma entidade somente com getters/setters e que não resolve problema de domínio e, aí, você cria as classes de serviço, que têm toda a lógica de domínio; acaba que você vai ter somente entidades anêmicas que vão estar conversando com o banco de dados. Então, nesse caso, é melhor utilizar Active Record.

Se você for trabalhar com domínio anêmico, muitas vezes, não tem problema, principalmente, quando se está trabalhando de forma 1:1 com o banco de dados em sistemas mais simples. Agora, para domínios complexos, não se deve trabalhar com entidades anêmicas, porque vai dar muito trabalho de manutenção ao longo do tempo.

Recomendações - em vias de regra:

- Domínios simples: Active Record;

- Domínios complexos: Data Mapper;

- Não há verdade absoluta.

# Object Relational Behavioral Patterns

## Atomicidade

Como trabalhar com diversos objetos?

- Rodar uma transação de forma atômica, onde ela persista todos os objetos, ou não persista nenhum.

Como garantir persistência?

- Normalmente, por meio de transações.

Para trabalhar com transações, se for trabalhar com Active Record, vai ser de uma forma, se for trabalhar com Data Mapper, vai ser de outra forma e, mesmo assim, não é simples.

Como realizar compensações?

- Como você sabe que determinado objeto foi modificado ou não?

- Como saber que um objeto deve ser criado?

- Como perceber que um objeto deve ser alterado e não criado?

É por isso que falamos em um padrão chamado de Unit of Work.

O padrão Unit of Work mantém uma lista de objetos afetados por uma transação de negócios e coordena as mudanças e os problemas de concorrência. 

Todas as listas que você for ter de objetos que você for fazer uma transação, você adiciona isso em uma unidade de trabalho e, então, essa unidade de trabalho vai ter um mapeamento de tudo o que está sendo alterado ou inserido. 

Quando você tiver tudo isso mapeado, você apenas executa um commit e ele vai fazer essa mágica de fazer todas essas alterações.

