# Saga

![Saga](/10microsservicoesearquiteturabaseadaaeventos/imagens/saga.png)
<p align="left">Fonte: Full Cycle, 2024.</p>

Quando estamos trabalhando com microsserviços ou qualquer tipo de sistema distribuído, normalmente, em algum momento, nós vamos ter um desafio: 

- Coordenar uma série de etapas pré-definidas que você sabe o começo, o meio e o fim. 

Isso, de forma geral, é muito complexo: 

- Em relação ao lado do negócio, porque você depende de regras de negócio, etapas do negócio e, como as organizações mudam a todo momento, essas etapas também podem mudar a todo momento;

É necessário coordenar que etapas sejam executadas exatamente em uma determinada ordem e, caso algo dê errado, essas compensações, ou seja, o rollback que é necessário fazer, também seja executado em uma determinada ordem.

Dependendo do modelo de etapas de negócio, pode fazer mais sentido a utilização de um orquestrador, o qual vai garantir que todas as etapas foram cumpridas.

Falando em sistemas distribuídos, normalmente, cada uma dessas etapas rodam em microsserviços separados, acontecendo de forma assíncrona.

Principalmente, de forma a garantir mais resiliência, ao invés de utilizar chamadas REST, as etapas são definidas em tópicos ou filas e os sistemas vão recebendo e retornando os resultados.

## Dinâmica de uma saga

Saga em momento de orquestração e não coreografia, ou seja, temos uma aplicação no meio, orquestrando e garantindo que todos os passos são finalizados.

Pontos de partida:

- Partindo do princípio de que cada uma dessas transações vão acontecer em microsserviços que têm os seus próprios bancos de dados totalmente independentes.

- Imaginando que, para iniciar uma saga, teremos uma aplicação mediadora recebendo dados de um tópico (Apache Kafka, RabbitMQ, etc.), passando um dado inicial.

- Esse dado inicial vai acoplar a um inicializador da nossa saga.

- Esse inicializador vai mandar essa mesma mensagem para um tópico de um microsserviço 1 (srv 1).

- Aqui temos o primeiro momento em que iniciou a primeira transação.

- No momento em que isso acontece, a aplicação mediadora começa a guardar estado, ou seja, a saga iniciou, o passo que ela iniciou foi o passo 1 e o status do primeiro passo é running, porque o passo começou a ser executado:

``
SagaStatus = Started
Step = 1
StepStatus = Running
``

- Lembrando que cada saga que inicia tem um id para identificação para separar uma saga da outra.

- Em seguida, o microsserviço 1 vai processar a transação.

- No momento em que ele processa a transação, ele dá um retorno, nesse caso, para o próprio tópico, mas poderia ser um tópico diferente: um para recebimento e outro para envio de dados.

- A seguir, essa informação sobre o step 1 é consumida pela aplicação mediadora. Então, nesse momento, os dados da saga começam a mudar o estado. A saga, agora, está como running, para o step 1, o status foi sucesso e, agora, é setado o próximo step como 2, que vai começar a rodar:

``
SagaStatus = Running
Step = 1
StepStatus = Succeeded

Step = 2
StepStatus = Pending
``

- Quando o passo 2 começa a inicializar, ou seja, quando é enviado informação para o passo 2, temos uma publicação em um tópico para o microsserviço 2, que vai executar esse microsserviço 2.

- Será processado com sucesso, vai avisar o step 2 que ele processou. Nesse momento, a nossa saga continua rodando, falando que o step 2 foi realizado com sucesso. E, agora, o próximo step que a gente vai ter é o step 3, que vai começar a rodar:

``
SagaStatus = Running
Step = 2
StepStatus = Succeeded

Step = 3
StepStatus = Pending
``

- Lembrando que uma operação só é executada após a outra.

- Lembrando que, dependendo da situação, você consegue executar sagas de forma local dentro do seu próprio sistema ou único sistema, como um sistema monolítico, por exemplo.

- Seguindo, o step 3 começa a ser executado. E, nesse momento, ele envia informação para o tópico e, na hora em que ele executou, falhou a execução: aconteceu algum erro.

- Quando um erro acontece, é necessário compensar esse erro, ou seja, realizar um processo de rollback.

- Nesse caso, o microsserviço 3 vai retornar para o tópico falando compensate (deu erro e será necessário compensar: voltar todos os steps).

- Assim, o status da saga muda, ao invés de running, vai mudar para compensated, ou seja, iniciou um processo de compensação - o step que foi executado, step 3, falhou.

- Então, o próximo passo é a execução do microsserviço 3 novamente: step 3.

``
SagaStatus = Compensating
Step = 3
StepStatus = Failed

Step = 3
StepStatus = Compensating
``

- A seguir, no step 3, vai executar uma operação de compensação, ou seja, tudo que foi executado no step 3 vai voltar para garantir a consistência.

- Seguindo para o próximo step, a saga tem ainda o status de compensando e o step 3 igual a compensated, ou seja, já foi compensado o step 3. E o próximo step é o 2 e o status está como compensando.

``
SagaStatus = Compensating
Step = 3
StepStatus = Compensated

Step = 1
StepStatus = Compensating
``

- A seguir, o step 2 recebe a informação, vai executar a transação de compensação. A informação vai voltar e o status da saga deve ser alterado novamente. O status da saga ainda está compensando e o step 2 já foi compensado. O último step é o 1 e deve estar compensando.

- Então, a informação é enviada para o step 1. O microsserviço 1 executa a informação de compensação, retorna a informação para o step 1 e, agora, o status da saga muda para compensatedandcompleted, o que quer dizer que a saga, realmente, terminou. Mas terminou de forma compensada. Ou seja, não aconteceu o que era esperado, porque, caso contrário, o status da saga estaria como completed apenas:

``
SagaStatus = CompensatedAndCompleted
Step = 1
StepStatus = Compensated

Step = 1
StepStatus = Compensated
``




