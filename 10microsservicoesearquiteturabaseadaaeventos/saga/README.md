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

```
SagaStatus = Started
Step = 1
StepStatus = Running
```

- Lembrando que cada saga que inicia tem um id para identificação para separar uma saga da outra.

- Em seguida, o microsserviço 1 vai processar a transação.

- No momento em que ele processa a transação, ele dá um retorno, nesse caso, para o próprio tópico, mas poderia ser um tópico diferente: um para recebimento e outro para envio de dados.

- A seguir, essa informação sobre o step 1 é consumida pela aplicação mediadora. Então, nesse momento, os dados da saga começam a mudar o estado. A saga, agora, está como running, para o step 1, o status foi sucesso e, agora, é setado o próximo step como 2, que vai começar a rodar:

```
SagaStatus = Running
Step = 1
StepStatus = Succeeded

Step = 2
StepStatus = Pending
```

- Quando o passo 2 começa a inicializar, ou seja, quando é enviado informação para o passo 2, temos uma publicação em um tópico para o microsserviço 2, que vai executar esse microsserviço 2.

- Será processado com sucesso, vai avisar o step 2 que ele processou. Nesse momento, a nossa saga continua rodando, falando que o step 2 foi realizado com sucesso. E, agora, o próximo step que a gente vai ter é o step 3, que vai começar a rodar:

```
SagaStatus = Running
Step = 2
StepStatus = Succeeded

Step = 3
StepStatus = Pending
```

- Lembrando que uma operação só é executada após a outra.

- Lembrando que, dependendo da situação, você consegue executar sagas de forma local dentro do seu próprio sistema ou único sistema, como um sistema monolítico, por exemplo.

- Seguindo, o step 3 começa a ser executado. E, nesse momento, ele envia informação para o tópico e, na hora em que ele executou, falhou a execução: aconteceu algum erro.

- Quando um erro acontece, é necessário compensar esse erro, ou seja, realizar um processo de rollback.

- Nesse caso, o microsserviço 3 vai retornar para o tópico falando compensate (deu erro e será necessário compensar: voltar todos os steps).

- Assim, o status da saga muda, ao invés de running, vai mudar para compensated, ou seja, iniciou um processo de compensação - o step que foi executado, step 3, falhou.

- Então, o próximo passo é a execução do microsserviço 3 novamente: step 3.

```
SagaStatus = Compensating
Step = 3
StepStatus = Failed

Step = 3
StepStatus = Compensating
```

- A seguir, no step 3, vai executar uma operação de compensação, ou seja, tudo que foi executado no step 3 vai voltar para garantir a consistência.

- Seguindo para o próximo step, a saga tem ainda o status de compensando e o step 3 igual a compensated, ou seja, já foi compensado o step 3. E o próximo step é o 2 e o status está como compensando.

```
SagaStatus = Compensating
Step = 3
StepStatus = Compensated

Step = 2
StepStatus = Compensating
```

- A seguir, o step 2 recebe a informação, vai executar a transação de compensação. A informação vai voltar e o status da saga deve ser alterado novamente. O status da saga ainda está compensando e o step 2 já foi compensado. O último step é o 1 e deve estar compensando.

```
SagaStatus = Compensating
Step = 2
StepStatus = Compensated

Step = 1
StepStatus = Compensating
```

- Então, a informação é enviada para o step 1. O microsserviço 1 executa a informação de compensação, retorna a informação para o step 1 e, agora, o status da saga muda para compensatedandcompleted, o que quer dizer que a saga, realmente, terminou. Mas terminou de forma compensada. Ou seja, não aconteceu o que era esperado, pois, caso contrário, o status da saga estaria como completed apenas:

```
SagaStatus = CompensatedAndCompleted
Step = 1
StepStatus = Compensated

Step = 1
StepStatus = Compensated
```

## Compensação e gerenciamento de erros

- Saga utilizando um orquestrador:

    - O orquestrador vai ser um serviço separado, também chamado de mediador.

    - O orquestrador faz uso de eventos e esses eventos acabam enviando informações para tópicos.

    - Não necessariamente, para que uma saga funcione dessa forma, devem ser utilizadas filas. Poderia-se utilizar REST também. Qual é o tradeoff de trabalhar com REST, neste caso?

    - Desvantagem:

        - No momento de enviar a informação para um microsserviço x, o microsserviço x está fora do ar. Então a saga vai ter que trabalhar com retentativas (retries). Serão necessários mecanismos de retry, exponential backoff, ou seja, ele envia, aguarda 1 segundo, envia novamente, aguarda 2 segundos, envia de novo, aguarda 3 segundos..., etc.

        - Quer dizer que é mais uma complexidade a ser adicionada em relação a políticas de retry.

    - Quando se está trabalhando com filas, esse tipo de problema não vai existir, porque a resiliência é garantida através das filas.

    - Por outro lado, temos um cenário onde, ao tentar compensar, ocorre um erro na compensação.

    - O que fazer, nesse caso, quando acontece um erro na compensação?

        - Nesse caso, pode ser criado um novo tópico. Esse tópico é um tópico apenas de erros. E, caso um erro aconteça, a informação é enviada para esse tópico de erro e pode haver:
        
            - Um sistema que fica analisando os erros e reiniciando as operações;

            - Ou alguém manualmente corrigindo/reiniciando as operações.

        - Qualquer sistema pode ter erros. Nesse caso, ao ocorrer o erro, deve-se jogar para um local onde é possível armazenar todos os erros para analisar caso a caso e para que nenhuma transação seja deixada para trás.

## Draft de código

```
package main

import {
    "time"
    "github.com/devfullcycle/go-saga/internal/saga"
}

func main() {
    // criar pedido
    step1 := &saga.SagaStep{
        StepName:           "place-order",
        StepTopic:          "place-order",
        StepType:           saga.SagaStepTypeNormal,
        Status:             saga.SagaStepStatusPending,
        Data:               []byte("data"),
        CompensateRetries   3
    }

    // geração de uma nota fiscal
    step2 := &saga.SagaStep{
        StepName:           "generate-invoice",
        StepTopic:          "generate-invoice",
        StepType:           saga.SagaStepTypeNormal,
        Status:             saga.SagaStepStatusPending,
        Data:               []byte("data"),
        CompensateRetries   3
    }

    // remover produto do estoque
    step3 := &saga.SagaStep{
        StepName:           "remove-product-from-stock",
        StepTopic:          "remove-product-from-stock",
        StepType:           saga.SagaStepTypeNormal,
        Status:             saga.SagaStepStatusPending,
        Data:               []byte("data"),
        CompensateRetries   3
    }

    // realizar o envio
    step3 := &saga.SagaStep{
        StepName:           "ship-order",
        StepTopic:          "ship-order",
        StepType:           saga.SagaStepTypeNormal,
        Status:             saga.SagaStepStatusPending,
        Data:               []byte("data"),
        CompensateRetries   3
    }

    saga := &saga.Saga{
        SagaID:                 "saga-1",
        SagaName:               "saga-1",
        Status:                 saga.SagaStepStatusPending,
        CreateTime:             time.Now(),
        UpdateTime:             time.Now(),
        ExpireTime:             time.Now().Add(time.Hour * 24),
        Version:                1
    }

    saga.AddStep(step1)
    saga.AddStep(step2)
    saga.AddStep(step3)
    saga.AddStep(step4)

    // lendo uma fila de mensagens
    for {
        saga.Init([]byte("data"))
        // gerar um id para a saga/transação
        // guardar o estado no banco de dados
        // log de tudo que acontece
    }
}
```
- Primeira coisa a ser feita na configuração de uma saga é a configurar os passos (steps).

- Passo 1 criar o pedido. Passo 2 gera a nota fiscal. Passo 3 vai remover o produto do estoque. O passo 4 vai realizar o envio do produto.

- Os passos são encadeados: se não conseguir realizar o envio do produto, vai ser necessário recolocar o produto no estoque, cancelar a nota fiscal e mudar o status da ordem com cancelado.

- Um passo depende do outro e acontecem em momentos diferentes. A saga pode ficar em aberto até finalizar.

- Para inicializar uma saga com um id e um nome. Ela contém dados de auditoria: CreateTime, UpdateTime e também ExpireTime: se, em 24 horas, não concluir com sucesso, vai compensar e finalizar.

- Também é possível fazer o controle de versão da saga. A empresa muda a todo momento, então, é possível criar os passos (steps) e configurar qual a versão dessa saga, pois é possível ter várias versões de uma mesma saga.

- E, a seguir, a saga vai adicionar todos os passos que são necessários para rodar essa saga.

- Para cada transação que acontecer, vai ser executado um init, o qual vai injetar os dados iniciais da saga.

- Imaginemos que está sendo lido uma fila de mensagens, tendo um for infinito lendo essa fila e, para cada transação que é recebida, os dados são jogados dentro da saga, a qual inicia baseado nos passos que foram configurados.

- De forma geral, parece ser simples. O que não é simples são os microsserviços, as operações de compensação e, principalmente, o gerenciamento de estado da saga.

- Quando é executado um init, normalmente:

    - Será gerado um id para a saga/transação;

    - Vai guardar o estado no banco de dados;

    - Vai ficar fazendo o processamento;

    - E, para cada etapa (step), vamos ter o log de tudo o que acontece. Ou seja, para cada momento, podemos ver passo-a-passo, pois, se tivermos uma transação com problema, podemos ver o histórico da transação; aonde começaram os problemas, aonde começou a compensar.

    - É similar a event sourcing: cada coisa que vai acontecendo, você consegue ver. E, se em algum momento a saga tiver um problema, pode-se dar um replay nos eventos para ver o que que acontece;

    - É muito importante, por questões, inclusive, de auditoria, armazenar todos os estados de tudo o que acontece na saga.

## AWS step functions

Sistema de gerenciamento de fluxo para automatizar workflows.

Consegue utilizar esses processos para orquestrar microsserviços.

Consegue ser um orquestrador:

- Tem todo o rastreio dos steps; o que acontece caso algo dê certo ou não der certo.

Consegue desenhar todo o fluxo e, a cada fluxo que acontece, pode-se executar um evento.

Na AWS, os eventos podem resultar na chamada de diversos serviços.

Ao executar o step 1, se der certo: 

- Mudar algo no DynamoDB.

- Iniciar o AWS Elemental.

- Começar a converter um bucket.

Casos de uso:

- Automatizar ETL;

- Automatizar etapas de segurança;

- Orquestrar microsserviços;

    - Normalmente, step functions chama lambda functions, trabalhando serverless e, cada vez que uma coisa acontece, vai chamando uma função diferente.

- Orquestrar diversos workloads em paralelo. As step functions conseguem executar tarefas em paralelo e, depois, é possível unificar todas as tarefas, rodar novamente em paralelo, etc.

Desvantagem:

- Alto custo, porque ela cobra por cada mudança de estado e não por cada transação iniciada: Do step 1 para o step 2, é cobrado; do step 2 para o step 3, é cobrado, e assim por diante...

- No free tier, recebe-se 4000 mudanças de estado.

Quando usar:

Se você utiliza vários serviços da AWS, é possível integrar com diversos serviços, então, acaba ficando dentro de um ecossistema que facilita o desenvolvimento.

Desvantagem:

- Uma vez que você entra nesse fluxo e toda a sua operação está rodando dentro desse fluxo, acaba sendo difícil sair, ainda mais se são utilizados outros serviços.

- Então, o lockin é extremamente forte.

### Referência
MBA ARQUITETURA FULL CYCLE. Microsserviços e arquitetura baseada a eventos. 2024. Disponível em: https://plataforma.fullcycle.com.br/. Acesso em: 29 nov. 2024.